# Data Manager

:triangular_flag_on_post: **Data Manager** application package.

## Author

**Kevin Doolaeghe**

## Setup

```
docker compose -p datamanager up -d
```

:warning: This program require a docker instance to be executed.

## Configuration

* Sample `configuration.yaml` configuration file for `homeassistant` :
 
```
panel_iframe:
  portainer:
    title: "Portainer"
    url: "https://portainer.kdoolaeghe.duckdns.org/#!/2/docker/dashboard"
    icon: mdi:docker
    require_admin: true
  proxy:
    title: "Proxy"
    url: "https://proxy.kdoolaeghe.duckdns.org"
    icon: mdi:arrow-decision
    require_admin: true
  dashy:
    title: "Dashy"
    url: "https://dashy.kdoolaeghe.duckdns.org"
    icon: mdi:monitor-dashboard
  grafana:
    title: "Grafana"
    url: "https://grafana.kdoolaeghe.duckdns.org"
    icon: mdi:chart-line
  plex:
    title: "Plex"
    url: "https://plex.kdoolaeghe.duckdns.org"
    icon: mdi:plex

# Loads default set of integrations. Do not remove.
default_config:

# Load frontend themes from the themes folder
frontend:
  themes: !include_dir_merge_named themes

automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml

http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 172.27.0.0/24

influxdb:
  api_version: 1
  host: influxdb
  port: 8086
  database: influxdb
  username: admin
  password: admin
  max_retries: 3
  default_measurement: state
  ssl: false
  verify_ssl: false
```

* Sample `telegraf.conf` configuration file for `telegraf` :

```
[global_tags]

[agent]
  interval = "30s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "10s"
  flush_jitter = "0s"
  precision = ""
  hostname = ""
  omit_hostname = false

[[outputs.influxdb]]
  urls = ["http://influxdb:8086"]
  database = "influxdb"
  timeout = "5s"
  username = "admin"
  password = "admin"

[[inputs.docker]]
  endpoint = "unix:///var/run/docker.sock"
  gather_services = false
  container_names = []
  source_tag = false
  container_name_include = []
  container_name_exclude = []
  timeout = "5s"
  perdevice = true
  total = false
  docker_label_include = []
  docker_label_exclude = []
  tag_env = ["JAVA_HOME", "HEAP_SIZE"]

[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false
  report_active = false

[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]

[[inputs.mem]]

[[inputs.processes]]
```
