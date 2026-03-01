# Prometheus Production Deployment -- Enterprise Observability SOP

## Overview

This guide provides step‑by‑step instructions to deploy Prometheus in a
production environment using a secure, scalable, and enterprise‑grade
configuration.

## Server Information

OS: Ubuntu 22.04 LTS\
Component: Prometheus\
Version: 3.5.1\
Install Path: /opt/observability/prometheus\
Service User: observability\
Service Port: 9090

------------------------------------------------------------------------

## Step 1 --- Create observability user

``` bash
sudo useradd --system --home /opt/observability --shell /usr/sbin/nologin observability
```

Verify:

``` bash
id observability
```

------------------------------------------------------------------------

## Step 2 --- Create directory structure

``` bash
sudo mkdir -p /opt/observability/prometheus/{bin,config,data,rules,logs}
sudo chown -R observability:observability /opt/observability
```

------------------------------------------------------------------------

## Step 3 --- Download Prometheus

``` bash
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v3.5.1/prometheus-3.5.1.linux-amd64.tar.gz
tar -xvf prometheus-3.5.1.linux-amd64.tar.gz
```

------------------------------------------------------------------------

## Step 4 --- Install binaries

``` bash
sudo cp prometheus-3.5.1.linux-amd64/prometheus /opt/observability/prometheus/bin/
sudo cp prometheus-3.5.1.linux-amd64/promtool /opt/observability/prometheus/bin/
sudo chmod +x /opt/observability/prometheus/bin/*
```

------------------------------------------------------------------------

## Step 5 --- Create Prometheus configuration

File:

    /opt/observability/prometheus/config/prometheus.yml

Example:

``` yaml
global:
  scrape_interval: 15s
  scrape_timeout: 10s
  evaluation_interval: 15s

  external_labels:
    monitor: observability-stack
    environment: production

scrape_configs:

  - job_name: prometheus
    static_configs:
      - targets:
        - localhost:9090
```

------------------------------------------------------------------------

## Step 6 --- Validate configuration

``` bash
/opt/observability/prometheus/bin/promtool check config /opt/observability/prometheus/config/prometheus.yml
```

Expected:

    SUCCESS: configuration is valid

------------------------------------------------------------------------

## Step 7 --- Create systemd service

File:

    /etc/systemd/system/prometheus.service

Service:

``` ini
[Unit]
Description=Prometheus Monitoring Service
After=network-online.target

[Service]
User=observability
Group=observability

ExecStart=/opt/observability/prometheus/bin/prometheus   --config.file=/opt/observability/prometheus/config/prometheus.yml   --storage.tsdb.path=/opt/observability/prometheus/data   --storage.tsdb.retention.time=30d   --storage.tsdb.wal-compression   --web.listen-address=:9090

Restart=always

StandardOutput=append:/opt/observability/prometheus/logs/prometheus.log
StandardError=append:/opt/observability/prometheus/logs/prometheus.log

[Install]
WantedBy=multi-user.target
```

------------------------------------------------------------------------

## Step 8 --- Start Prometheus

``` bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
```

------------------------------------------------------------------------

## Step 9 --- Verify service

``` bash
sudo systemctl status prometheus
ss -lntp | grep 9090
curl localhost:9090/-/ready
```

Expected:

    Prometheus is Ready

------------------------------------------------------------------------

## Result

Prometheus is now running as a production‑grade monitoring service ready
for Grafana integration.

------------------------------------------------------------------------

Author: Avinash Chavan\
Role: DevOps / Cloud Engineer
[GitHub Profile](https://github.com/chavanavinassh)