<p align="center"><img src="https://www.clipartmax.com/png/middle/118-1186067_prometheus-software-logo-prometheus-monitoring.png" width="200"></p>

## Hello {$username}, here is my solution for the Scoro Tech Task ##

I decided to deploy Prometheus and other stuff using [Docker Compose](https://docs.docker.com/compose/install/). I'm just doing homework so Docker is affordable in my case. For Production environment i would rather stick to Podman or CRI-O or something like that. (Security issues and non-root privileges escalations and so on, you know)

### Prerequisites ###

Let's pretend you can ssh to a Linux machine with Docker Engine and Docker Compose installed and docker systemd service is configured and enabled and you don't have to use sudo to launch containters and the following command gives you the proper output:

```console
docker-compose --version
```

### First of all, let's clone the repo and navigate to the project folder ###

```console
git clone https://github.com/ashliakhov/prom_homework.git

cd prom_homework
```
### Let's run our Prometheus and Node Exporter containers with a single command ###

```console
docker-compose up -d
```
The content of the Docker Compose file is as follows:

```yaml
# docker-compose.yml

version: '3'

services:
  prometheus:
    image: prom/prometheus:latest # We will NOT pull tag:latest in PROD before we check the changelog and TEST it before for sure
    container_name: prometheus
    volumes:
      - ./config/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./config/prometheus/alertrules.yml:/etc/prometheus/alertrules.yml
      - ./config/prometheus/targets.json:/etc/prometheus/targets.json
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    ports:
      - '9090:9090'
    restart: always # To restart a container after reboot or on failure
  
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - '9100:9100'
    restart: always
```

We can check if everything is up and running:

```console
docker ps

curl localhost:9090
```

### Prometheus scrapes Node exporter metrics which is defined in ./config/prometheus/prometheus.yml (File-based service discovery is used) ###

```yaml
# ./config/prometheus/prometheus.yml

global:
  scrape_interval:     15s 
  evaluation_interval: 15s

rule_files:
  - '/etc/prometheus/alertrules.yml'

scrape_configs:

  - job_name: 'node_exporter'
    file_sd_configs: # File-based service discovery config
    - files:
      - '/etc/prometheus/targets.json'
```
Scraped targets are described in a JSON file:

```json
# ./config/prometheus/targets.json

[
    {
        "labels": {
            "job": "node_exporter"
        },
        "targets": [
            "localhost:9100"
        ]
    }
]
```
and available @ http://localhost:9090/targets

### Alertmanager will not be deployed in scope of this homework ###
Alert rule is defined in ./config/prometheus/alertrules.yml

```yaml
# ./config/prometheus/alertrules.yml
groups:
- name: Node Exporter metrics
  rules:   
  - alert: Node Exporter is down
    expr: up{job="node_exporter"} == 0 # Job called "node_exporter" is not running
    for: 1m
    labels:
      severity: CRITICAL
    annotations:
      title: Node {{ $labels.instance }} is down
      description: Failed to scrape {{ $labels.job }} on {{ $labels.instance }} for more than 1 minute.
```

Alerts info is available @ http://localhost:9090/alerts

### Seems like we've successfully deployed and configured Prometheus, Node Exporter, File-based SD and will see alerts against failed scraped targets ###

### # ###

### In order to monitor SSL certificate expiration date we can basically use [Blackbox exporter](https://github.com/prometheus/blackbox_exporter) (metric called "probe_ssl_earliest_cert_expiry") for example ###

### # ###

### In order to automatically generate JSON file containing targets to scrape using data stored in etcd [Custom Service Discovery](https://prometheus.io/blog/2018/07/05/implementing-custom-sd/) could be implemented (if there is no native built-in SD integration for the particular product) ###

### ====== ###

<p></p><img src=https://sysdig.com/wp-content/uploads/2019/03/Sysdig-and-prometheus-3.png width="300"></p>

# STAY HEALTHY! #