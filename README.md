# BIK-VDC



## Instalace lxd

`sudo snap install lxd`

## Sestavení klastru

### jahodvik-01

`sudo lxd init`


```

Would you like to use LXD clustering? (yes/no) [default=no]: yes

What IP address or DNS name should be used to reach this server? [default=10.119.76.68]: 

Are you joining an existing cluster? (yes/no) [default=no]: no

What member name should be used to identify this server in the cluster? [default=jahodvik-01]: 

Do you want to configure a new local storage pool? (yes/no) [default=yes]: 

Name of the storage backend to use (dir, lvm, zfs, btrfs) [default=zfs]: 

Create a new ZFS pool? (yes/no) [default=yes]: 

Would you like to use an existing empty block device (e.g. a disk or partition)? (yes/no) [default=no]: 

Size in GiB of the new loop device (1GiB minimum) [default=7GiB]: 

Do you want to configure a new remote storage pool? (yes/no) [default=no]: 

Would you like to connect to a MAAS server? (yes/no) [default=no]: 

Would you like to configure LXD to use an existing bridge or host interface? (yes/no) [default=no]: yes

Name of the existing bridge or host interface: ens192

Would you like stale cached images to be updated automatically? (yes/no) [default=yes]: 

Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: 


```

Povoleni web gui

`snap set lxd ui.enable=true`
`snap restart --reload lxd`
`lxc config set core.https_address :8443`

Postupujte podle pokynů na adrese https://<jahodvik-01 ip>:8443, které vás provedou procesem ověřování.


Generovani join tokenu pro pridani dalsich serveru do klastru


`lxc cluster add jahodvik-02`
`lxc cluster add jahodvik-03`


### jahodvik-02

`sudo lxd init`

Nezapomeneme vlozit dany token 

```
uld you like to use LXD clustering? (yes/no) [default=no]: yes

What IP address or DNS name should be used to reach this node?

[default=192.168.0.19]:

Are you joining an existing cluster? (yes/no) [default=no]: yes

Do you have a join token? (yes/no/[token]) [default=no]: eyJzZXJ2ZXJfbmFtZSI6Im1hcnZpbklJSSIsImZpbmdlcnByaW50IjoiNjhlMjljYzBlN2IxNTkzNWY1MGM5YjI3NjM0NmFhNDU1OTc2ZWQ1N2Y4ODAyZTYxMTc4MzUwOThlNjNkNmFmYSIsImFkZHJlc3NlcyI6WyIxOTIuMTY4LjAuMTg6ODQ0MyJdLCJzZWNyZXQiOiIxODg3ZWQxMmIxN2MwOTYwZDM1NTU0Zjc3M2IxNzU2NmZlNWExZjQ1M2VhZTc1NmVmYzk0OGExMDUyYjYwNTE5In0=

All existing data is lost when joining a cluster, continue? (yes/no)[default= no]: yes

Choose “size” property for storage pool “local”:

Choose "source" property for storage pool "local":

Choose "zfs.pool_name" property for storage pool "local":

Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]:

```


### jahodvik-03

`sudo lxd init`

Nezapomeneme vlozit dany token 

```
uld you like to use LXD clustering? (yes/no) [default=no]: yes

What IP address or DNS name should be used to reach this node?

[default=192.168.0.19]:

Are you joining an existing cluster? (yes/no) [default=no]: yes

Do you have a join token? (yes/no/[token]) [default=no]: eyJzZXJ2ZXJfbmFtZSI6Im1hcnZpbklJSSIsImZpbmdlcnByaW50IjoiNjhlMjljYzBlN2IxNTkzNWY1MGM5YjI3NjM0NmFhNDU1OTc2ZWQ1N2Y4ODAyZTYxMTc4MzUwOThlNjNkNmFmYSIsImFkZHJlc3NlcyI6WyIxOTIuMTY4LjAuMTg6ODQ0MyJdLCJzZWNyZXQiOiIxODg3ZWQxMmIxN2MwOTYwZDM1NTU0Zjc3M2IxNzU2NmZlNWExZjQ1M2VhZTc1NmVmYzk0OGExMDUyYjYwNTE5In0=

All existing data is lost when joining a cluster, continue? (yes/no)[default= no]: yes

Choose “size” property for storage pool “local”:

Choose "source" property for storage pool "local":

Choose "zfs.pool_name" property for storage pool "local":

Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]:

```

Po zadani prikau  ize bychom meli videt klastr ketry ma tri nody

`lxc cluster list`

## Pristup na klastr pres lokalni stroj

### jahodvik-01

`lxc config set core.trust_password <heslo>`

### lokalni pc pripojen na vpn s ssh pristupem na jahodvvik-01

`sudo snap install lxd`

Povolime fingerprint a zadame heslo ktere jseme nastavili v predchozim kroku

`lxc remote add rpi <ip jahodvik-01>`

## Monitoring metrik

Vytvorime novou instanci na nodu jahodvik-01

`lxc init ubuntu:20.04 metrics --target jahodvik-01`

Povoleni API

`lxc config set core.metrics_address ":8444"`

`mkdir /home/jahodvik/tls`
`cd /home/jahodvik/tls`





Prihlasime se do metrics instance bud prikazem nize nebo pres webovou konzoli v gui

`lxc exec metrics bash`

### metrics

`snap install prometheus`

`mkdir /var/snap/prometheus/current/tls`

`cd /var/snap/prometheus/current/tls`

Generovani certifikatu

`openssl req -x509 -newkey ec -pkeyopt ec_paramgen_curve:secp384r1 -sha384 -keyout metrics.key -nodes -out metrics.crt -days 3650 -subj "/CN=metrics.local"`

### jahodvik-01

`lxc file pull metrics/var/snap/prometheus/current/tls/metrics.crt`

Pridani certifikatu jako duveryhodny

`lxc config trust add metrics.crt --type=metrics`


`lxc file push /var/snap/lxd/common/lxd/server.crt metrics/var/snap/prometheus/current/tls/`


### jahodvik-02


`lxc config set core.metrics_address ":8444"`
### jahodvik-03



`lxc config set core.metrics_address ":8444"`

### metrics

do souboru vlozte tuto konfiguraci, nezapomente zmenit ip adresy v targets
`vi /var/snap/prometheus/current/prometheus.yml`

```


# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: lxd
    metrics_path: '/1.0/metrics'
    scheme: 'https'
    static_configs:
      - targets: 
        - '<ip jahodvik-01>:8444'
        - '<ip jahodvik-02>:8444'
        - '<ip jahodvik-03>:8444'  
    tls_config:
      insecure_skip_verify: true
      ca_file: 'tls/server.crt'
      cert_file: 'tls/metrics.crt'
      key_file: 'tls/metrics.key'
      # XXX: server_name is required if the target name
      #      is not covered by the certificate (not in the SAN list)
      server_name: 'jahodvik-cluster' 



```


`snap restart prometheus`


### jahodvik-01

Nasteveni port forwarding

`lxc network forward create lxdfan0 <ip jahodvik-01> `

`lxc network forward port add lxdfan0 <ip jahodvik-01> tcp 32456 <ip metrics inatance> 9090`

`lxc network forward port add lxdfan0 <ip jahodvik-01> tcp 32457 <ip metrics inatance> 3000`

Na adrese http://<jahodvik-01>:32456 bychom meli mit prometheus

### metrics

`sudo apt-get install -y apt-transport-https software-properties-common wget`

`sudo mkdir -p /etc/apt/keyrings/`
`wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null`

`echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list`
`sudo apt-get update`
`sudo apt-get install grafana loki`
`sudo /bin/systemctl daemon-reload`
`sudo /bin/systemctl enable grafana-server`
`sudo /bin/systemctl start grafana-server`


Na http://<jahodvik-01>:32457 mame grafanu

Nejprve si pripravime datasource z loki

Click Connections in the left-side menu.
Under Connections, click Add new connection.
Enter Loki in the search bar.
Select Loki data source.
Click Create a Loki data source in the upper right.

v poli url zadejte localhost:3100


To stejne pro prometheus

Click Connections in the left-side menu.
Under Connections, click Add new connection.
Enter Prometheus in the search bar.
Select Prometheus data source.
Click Create a Prometheus data source in the upper right.

v poli url zadejte localhost:9090


Navigate to the data source’s configuration page.
Select the Dashboards tab.
This displays dashboards for Grafana and Prometheus.

Select Import for the dashboard to import.

do grafana ID vlozte 19131 a stisknete load

A mame dashboard


## Nginx

`lxc init ubuntu:20.04 web --target jahodvik-02`

### web

`sudo apt install nginx`

### jahodvik-02

`lxc network forward create lxdfan0 <ip jahodvik-02> `

`lxc network forward port add lxdfan0 <ip jahodvik-02> tcp 32455 <ip web insatance> 80`


## Docker dashboard

`lxc init ubuntu:20.04 docker-homepage --target jahodvik-03`

### jahodvik-03


`lxc storage create docker btrfs --target jahodvik-03`
`lxc storage volume create docker docker-homepage`
`lxc config device add docker-homepage docker disk pool=docker source=docker-homepage path=/var/lib/docker`


`lxc config set docker-homepage security.nesting=true security.syscalls.intercept.setxattr=true security.syscalls.intercept.mknod=true`
 `lxc restart docker-homepage`


`lxc network forward create lxdfan0 <ip jahodvik-02> `

`lxc network forward port add lxdfan0 <ip jahodvik-02> tcp 32458 <ip docker-homepage insatance> 8080`


### docker-homepage

`snap install docker`

`docker run -d -p 8080:8080 --restart=always b4bz/homer:latest`

`docker exec -it <container id> sh`

Zkopirujte konfiguraci pozmente ip adresy

`vi /www/assets/config.yml`


```
---
# Homepage configuration
# See https://fontawesome.com/v5/search for icons options

title: "Project dashboard"
subtitle: "BIK-VDC"
logo: "logo.png"
# icon: "fas fa-skull-crossbones" # Optional icon

header: true
footer: '<p>Created with <span class="has-text-danger">❤️</span> with <a href="https://bulma.io/">bulma</a>, <a href="https://vuejs.org/">vuejs</a> & <a href="https://fontawesome.com/">font awesome</a> // Fork me on <a href="https://github.com/bastienwirtz/homer"><i class="fab fa-github-alt"></i></a></p>' # set false if you want to hide it.

# Optional theme customization
theme: default
colors:
  light:
    highlight-primary: "#3367d6"
    highlight-secondary: "#4285f4"
    highlight-hover: "#5a95f5"
    background: "#f5f5f5"
    card-background: "#ffffff"
    text: "#363636"
    text-header: "#ffffff"
    text-title: "#303030"
    text-subtitle: "#424242"
    card-shadow: rgba(0, 0, 0, 0.1)
    link: "#3273dc"
    link-hover: "#363636"
  dark:
    highlight-primary: "#3367d6"
    highlight-secondary: "#4285f4"
    highlight-hover: "#5a95f5"
    background: "#131313"
    card-background: "#2b2b2b"
    text: "#eaeaea"
    text-header: "#ffffff"
    text-title: "#fafafa"
    text-subtitle: "#f5f5f5"
    card-shadow: rgba(0, 0, 0, 0.4)
    link: "#3273dc"
    link-hover: "#ffdd57"


# Optional navbar
# links: [] # Allows for navbar (dark mode, layout, and search) without any links
links:
  - name: "Contribute"
    icon: "fab fa-github"
    url: "https://github.com/bastienwirtz/homer"
    target: "_blank" # optional html a tag target attribute
  - name: "Wiki"
    icon: "fas fa-book"
    url: "https://www.wikipedia.org/"
  # this will link to a second homer page that will load config from additional-page.yml and keep default config values as in config.yml file
  # see url field and assets/additional-page.yml.dist used in this example:
  #- name: "another page!"
  #  icon: "fas fa-file-alt"
  #  url: "#additional-page" 

# Services
# First level array represent a group.
# Leave only a "items" key if not using group (group name, icon & tagstyle are optional, section separation will not be displayed).
services:
  - name: "Applications"
    icon: "fas fa-cloud"
    items:
      - name: "LXD GUI"
        logo: "assets/tools/lxd.png"
        url: "https://<jahodvik-01>:8443"
      - name: "Grafana"
        logo: "assets/tools/grafana.png"
        url: "http://<jahodvik-01>:32457"
      - name: "Prometheus"
        logo: "assets/tools/prometheus.png"
        url: "http://<jahodvik-01>:32456"
      - name: "Nginx"
        logo: "assets/tools/nginx.png"
        url: "http://<jahodvik-02>:32455"  
        
```


