# Loki Production Deployment -- Enterprise Observability SOP

## Overview

This guide provides step-by-step instructions to deploy Grafana Loki in
a production environment.

Loki is a log aggregation system designed to:

• Collect system logs\
• Collect application logs\
• Store logs efficiently\
• Integrate with Grafana\
• Support centralized log monitoring

Loki works together with Promtail and Grafana.

------------------------------------------------------------------------

## Server Information

OS: Ubuntu 22.04 LTS\
Component: Loki\
Install Path: /opt/observability/loki\
Service User: observability\
Service Port: 3100

------------------------------------------------------------------------

## Step 1 --- Create directory structure

``` bash
sudo mkdir -p /opt/observability/loki/{bin,config,data/{chunks,index,wal,cache},logs}
sudo chown -R observability:observability /opt/observability/loki
```

------------------------------------------------------------------------

## Step 2 --- Download Loki

``` bash
cd /tmp

wget https://github.com/grafana/loki/releases/latest/download/loki-linux-amd64.zip

unzip loki-linux-amd64.zip
```

------------------------------------------------------------------------

## Step 3 --- Install binary

``` bash
sudo mv loki-linux-amd64 /opt/observability/loki/bin/loki

sudo chmod +x /opt/observability/loki/bin/loki
```

------------------------------------------------------------------------

## Step 4 --- Create Loki configuration

File:

    /opt/observability/loki/config/loki.yml

Example production configuration:

``` yaml
auth_enabled: false

server:
  http_listen_port: 3100

common:
  path_prefix: /opt/observability/loki/data

  storage:
    filesystem:
      chunks_directory: /opt/observability/loki/data/chunks
      rules_directory: /opt/observability/loki/data/rules

  replication_factor: 1

schema_config:
  configs:
    - from: 2024-01-01
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

storage_config:
  filesystem:
    directory: /opt/observability/loki/data/chunks

limits_config:
  retention_period: 30d

compactor:
  working_directory: /opt/observability/loki/data/compactor
```

------------------------------------------------------------------------

## Step 5 --- Create systemd service

File:

    /etc/systemd/system/loki.service

Service:

``` ini
[Unit]
Description=Grafana Loki Log Aggregation
After=network-online.target

[Service]

User=observability
Group=observability

ExecStart=/opt/observability/loki/bin/loki   -config.file=/opt/observability/loki/config/loki.yml

Restart=always

StandardOutput=append:/opt/observability/loki/logs/loki.log
StandardError=append:/opt/observability/loki/logs/loki.log

[Install]
WantedBy=multi-user.target
```

------------------------------------------------------------------------

## Step 6 --- Start Loki

``` bash
sudo systemctl daemon-reload

sudo systemctl enable loki

sudo systemctl start loki
```

------------------------------------------------------------------------

## Step 7 --- Verify service

Check status:

``` bash
sudo systemctl status loki
```

Check port:

``` bash
ss -lntp | grep 3100
```

Health check:

``` bash
curl http://localhost:3100/ready
```

Expected:

    ready

------------------------------------------------------------------------

## Step 8 --- Verify Loki API

``` bash
curl http://localhost:3100/metrics
```

Expected:

    loki_build_info

------------------------------------------------------------------------

## Step 9 --- Log location

    /opt/observability/loki/logs/loki.log

View logs:

``` bash
tail -f loki.log
```

------------------------------------------------------------------------

## Step 10 --- Grafana integration

In Grafana:

Settings → Data Sources → Add

Select:

    Loki

URL:

    http://localhost:3100

Click:

    Save & Test

Expected:

    Data source connected

------------------------------------------------------------------------

## Troubleshooting

Check logs:

``` bash
journalctl -u loki -n 100
```

Check config:

``` bash
/opt/observability/loki/bin/loki -config.file=/opt/observability/loki/config/loki.yml -verify-config=true
```

Check port conflict:

``` bash
ss -lntp | grep 3100
```

------------------------------------------------------------------------

## Result

Loki is now running and ready to receive logs from Promtail.

Logs can be queried and visualized in Grafana.

------------------------------------------------------------------------

Author: Avinash Chavan\
Role: DevOps / Cloud Engineer
[GitHub Profile](https://github.com/chavanavinassh)