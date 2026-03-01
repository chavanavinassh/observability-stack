# Promtail Production Deployment -- Enterprise Observability SOP

## Overview

This guide provides step-by-step instructions to deploy Grafana Promtail
in a production environment.

Promtail is a log shipping agent responsible for:

• Collecting system logs\
• Collecting application logs\
• Collecting service logs\
• Sending logs to Loki\
• Adding labels for structured querying

Promtail integrates directly with Loki and Grafana.

------------------------------------------------------------------------

## Server Information

OS: Ubuntu 22.04 LTS\
Component: Promtail\
Install Path: /opt/observability/promtail\
Service User: observability\
Loki Endpoint: http://localhost:3100

------------------------------------------------------------------------

## Step 1 --- Create directory structure

``` bash
sudo mkdir -p /opt/observability/promtail/{bin,config,positions,logs}
sudo chown -R observability:observability /opt/observability/promtail
```

------------------------------------------------------------------------

## Step 2 --- Download Promtail

``` bash
cd /tmp

wget https://github.com/grafana/loki/releases/latest/download/promtail-linux-amd64.zip

unzip promtail-linux-amd64.zip
```

------------------------------------------------------------------------

## Step 3 --- Install binary

``` bash
sudo mv promtail-linux-amd64 /opt/observability/promtail/bin/promtail

sudo chmod +x /opt/observability/promtail/bin/promtail
```

------------------------------------------------------------------------

## Step 4 --- Create Promtail configuration

File:

    /opt/observability/promtail/config/promtail.yml

Example production configuration:

``` yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /opt/observability/promtail/positions/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push

scrape_configs:

  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: system
          host: server
          __path__: /var/log/*.log

  - job_name: auth_logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: auth
          __path__: /var/log/auth.log

  - job_name: syslog
    static_configs:
      - targets:
          - localhost
        labels:
          job: syslog
          __path__: /var/log/syslog
```

------------------------------------------------------------------------

## Step 5 --- Create systemd service

File:

    /etc/systemd/system/promtail.service

Service:

``` ini
[Unit]
Description=Grafana Promtail Log Shipper
After=network-online.target

[Service]

User=observability
Group=observability

ExecStart=/opt/observability/promtail/bin/promtail   -config.file=/opt/observability/promtail/config/promtail.yml

Restart=always

StandardOutput=append:/opt/observability/promtail/logs/promtail.log
StandardError=append:/opt/observability/promtail/logs/promtail.log

[Install]
WantedBy=multi-user.target
```

------------------------------------------------------------------------

## Step 6 --- Start Promtail

``` bash
sudo systemctl daemon-reload

sudo systemctl enable promtail

sudo systemctl start promtail
```

------------------------------------------------------------------------

## Step 7 --- Verify service

Check status:

``` bash
sudo systemctl status promtail
```

Check port:

``` bash
ss -lntp | grep 9080
```

Check logs:

``` bash
tail -f /opt/observability/promtail/logs/promtail.log
```

Expected:

    level=info msg="Promtail started"

------------------------------------------------------------------------

## Step 8 --- Verify in Loki

Run:

``` bash
curl http://localhost:3100/metrics
```

Confirm Loki is receiving logs.

------------------------------------------------------------------------

## Step 9 --- Verify in Grafana

Open Grafana:

Explore → Select Loki

Query:

    {job="system"}

Expected:

Logs appear.

------------------------------------------------------------------------

## Step 10 --- Positions file

Location:

    /opt/observability/promtail/positions/positions.yaml

Purpose:

Prevents duplicate log ingestion after restart.

------------------------------------------------------------------------

## Log location

    /opt/observability/promtail/logs/promtail.log

------------------------------------------------------------------------

## Troubleshooting

Check service logs:

``` bash
journalctl -u promtail -n 100
```

Verify config:

``` bash
/opt/observability/promtail/bin/promtail -config.file=/opt/observability/promtail/config/promtail.yml -check-syntax
```

Check Loki connectivity:

``` bash
curl http://localhost:3100/ready
```

------------------------------------------------------------------------

## Result

Promtail is now running and sending logs to Loki.

Logs can be queried in Grafana.

------------------------------------------------------------------------

Author: Avinash Chavan\
Role: DevOps / Cloud Engineer
[GitHub Profile](https://github.com/chavanavinassh)