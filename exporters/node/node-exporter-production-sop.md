# Node Exporter Production Deployment -- Enterprise Observability SOP

## Overview

This guide provides step-by-step instructions to deploy Prometheus Node
Exporter in a production environment.

Node Exporter collects system-level metrics including:

• CPU usage\
• Memory usage\
• Disk utilization\
• Network statistics\
• Filesystem usage\
• Load average

These metrics are scraped by Prometheus and visualized in Grafana.

------------------------------------------------------------------------

## Server Information

OS: Ubuntu 22.04 LTS\
Component: Node Exporter\
Install Path: /opt/observability/exporters/node\
Service User: observability\
Service Port: 9100

------------------------------------------------------------------------

## Step 1 --- Create directory structure

``` bash
sudo mkdir -p /opt/observability/exporters/node/{bin,config,logs}
sudo chown -R observability:observability /opt/observability/exporters/node
```

------------------------------------------------------------------------

## Step 2 --- Download Node Exporter

``` bash
cd /tmp

wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-1.8.2.linux-amd64.tar.gz

tar -xvf node_exporter-*.tar.gz
```

------------------------------------------------------------------------

## Step 3 --- Install binary

``` bash
sudo cp node_exporter-*/node_exporter /opt/observability/exporters/node/bin/

sudo chmod +x /opt/observability/exporters/node/bin/node_exporter
```

------------------------------------------------------------------------

## Step 4 --- Create systemd service

File:

    /etc/systemd/system/node_exporter.service

Service:

``` ini
[Unit]
Description=Prometheus Node Exporter
After=network-online.target

[Service]

User=observability
Group=observability

ExecStart=/opt/observability/exporters/node/bin/node_exporter   --web.listen-address=:9100   --log.level=info

Restart=always

StandardOutput=append:/opt/observability/exporters/node/logs/node_exporter.log
StandardError=append:/opt/observability/exporters/node/logs/node_exporter.log

[Install]
WantedBy=multi-user.target
```

------------------------------------------------------------------------

## Step 5 --- Start service

``` bash
sudo systemctl daemon-reload

sudo systemctl enable node_exporter

sudo systemctl start node_exporter
```

------------------------------------------------------------------------

## Step 6 --- Verify service

Check status:

``` bash
sudo systemctl status node_exporter
```

Check port:

``` bash
ss -lntp | grep 9100
```

Test metrics:

``` bash
curl localhost:9100/metrics
```

Expected output:

    node_cpu_seconds_total
    node_memory_MemAvailable_bytes
    node_filesystem_size_bytes

------------------------------------------------------------------------

## Step 7 --- Prometheus integration

Create target file:

    /opt/observability/targets/node/server.yml

Example:

``` yaml
- targets:
  - localhost:9100
  labels:
    project: infrastructure
    service: server
    environment: production
```

Restart Prometheus:

``` bash
sudo systemctl restart prometheus
```

------------------------------------------------------------------------

## Step 8 --- Verify in Prometheus

Open:

http://SERVER-IP:9090

Navigate:

Status → Targets

Expected:

    node   UP

------------------------------------------------------------------------

## Step 9 --- Log location

    /opt/observability/exporters/node/logs/node_exporter.log

View logs:

``` bash
tail -f node_exporter.log
```

------------------------------------------------------------------------

## Troubleshooting

Check service logs:

``` bash
journalctl -u node_exporter -n 100
```

Check port conflict:

``` bash
ss -lntp | grep 9100
```

------------------------------------------------------------------------

## Result

Node Exporter is now running and exporting system metrics to Prometheus.

These metrics can be visualized in Grafana dashboards and used for
alerting.

------------------------------------------------------------------------

Author: Avinash Chavan\
Role: DevOps / Cloud Engineer
[GitHub Profile](https://github.com/chavanavinassh)