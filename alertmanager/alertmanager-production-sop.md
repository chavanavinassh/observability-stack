# Alertmanager Production Deployment -- Enterprise Observability SOP

## Overview

This guide provides step-by-step instructions to deploy Prometheus
Alertmanager in a production environment.

Alertmanager is responsible for:

• Handling alerts from Prometheus\
• Sending notifications via Email, Slack, Teams, PagerDuty\
• Alert grouping and routing\
• Alert deduplication\
• Alert silencing

Alertmanager is a core component of an enterprise observability
platform.

------------------------------------------------------------------------

## Server Information

OS: Ubuntu 22.04 LTS\
Component: Alertmanager\
Install Path: /opt/observability/alertmanager\
Service User: observability\
Service Port: 9093

------------------------------------------------------------------------

## Step 1 --- Create directory structure

``` bash
sudo mkdir -p /opt/observability/alertmanager/{bin,config,data,logs,templates}
sudo chown -R observability:observability /opt/observability/alertmanager
```

------------------------------------------------------------------------

## Step 2 --- Download Alertmanager

``` bash
cd /tmp

wget https://github.com/prometheus/alertmanager/releases/latest/download/alertmanager-0.27.0.linux-amd64.tar.gz

tar -xvf alertmanager-*.tar.gz
```

------------------------------------------------------------------------

## Step 3 --- Install binaries

``` bash
sudo cp alertmanager-*/alertmanager /opt/observability/alertmanager/bin/

sudo cp alertmanager-*/amtool /opt/observability/alertmanager/bin/

sudo chmod +x /opt/observability/alertmanager/bin/*
```

------------------------------------------------------------------------

## Step 4 --- Create Alertmanager configuration

File:

    /opt/observability/alertmanager/config/alertmanager.yml

Example production configuration (Email):

``` yaml
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'your-email@gmail.com'
  smtp_auth_username: 'your-email@gmail.com'
  smtp_auth_password: 'your-app-password'

route:
  receiver: email-alert

receivers:

- name: email-alert
  email_configs:
  - to: your-email@gmail.com
    send_resolved: true
```

------------------------------------------------------------------------

## Step 5 --- Validate configuration

``` bash
/opt/observability/alertmanager/bin/amtool check-config /opt/observability/alertmanager/config/alertmanager.yml
```

Expected:

    SUCCESS

------------------------------------------------------------------------

## Step 6 --- Create systemd service

File:

    /etc/systemd/system/alertmanager.service

Service:

``` ini
[Unit]
Description=Prometheus Alertmanager
After=network-online.target

[Service]

User=observability
Group=observability

ExecStart=/opt/observability/alertmanager/bin/alertmanager   --config.file=/opt/observability/alertmanager/config/alertmanager.yml   --storage.path=/opt/observability/alertmanager/data

Restart=always

StandardOutput=append:/opt/observability/alertmanager/logs/alertmanager.log
StandardError=append:/opt/observability/alertmanager/logs/alertmanager.log

[Install]
WantedBy=multi-user.target
```

------------------------------------------------------------------------

## Step 7 --- Start Alertmanager

``` bash
sudo systemctl daemon-reload

sudo systemctl enable alertmanager

sudo systemctl start alertmanager
```

------------------------------------------------------------------------

## Step 8 --- Verify service

Check status:

``` bash
sudo systemctl status alertmanager
```

Check port:

``` bash
ss -lntp | grep 9093
```

Health check:

``` bash
curl http://localhost:9093/-/ready
```

Expected:

    OK

------------------------------------------------------------------------

## Step 9 --- Access Web UI

Open browser:

    http://SERVER-IP:9093

Alertmanager dashboard will load.

------------------------------------------------------------------------

## Step 10 --- Prometheus integration

Ensure Prometheus config contains:

``` yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - localhost:9093
```

Restart Prometheus:

``` bash
sudo systemctl restart prometheus
```

------------------------------------------------------------------------

## Step 11 --- Log location

    /opt/observability/alertmanager/logs/alertmanager.log

View logs:

``` bash
tail -f alertmanager.log
```

------------------------------------------------------------------------

## Troubleshooting

Check logs:

``` bash
journalctl -u alertmanager -n 100
```

Check config:

``` bash
amtool check-config alertmanager.yml
```

Check port conflict:

``` bash
ss -lntp | grep 9093
```

------------------------------------------------------------------------

## Result

Alertmanager is now running and ready to send alerts from Prometheus.

Supports:

• Email alerts\
• Slack alerts\
• Teams alerts\
• PagerDuty

------------------------------------------------------------------------

Author: Avinash Chavan\
Role: DevOps / Cloud Engineer
[GitHub Profile](https://github.com/chavanavinassh)
