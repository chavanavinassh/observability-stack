# Grafana Production Deployment -- Enterprise Observability SOP

## Overview

This guide provides step-by-step instructions to deploy Grafana in a
production environment.

Grafana provides:

• Visualization dashboards\
• Metrics monitoring (Prometheus)\
• Log monitoring (Loki)\
• Alerting\
• SSL, uptime, and infrastructure dashboards

Grafana acts as the central visualization layer of the observability
stack.

------------------------------------------------------------------------

## Server Information

OS: Ubuntu 22.04 LTS\
Component: Grafana\
Install Path: /opt/observability/grafana\
Service User: observability\
Service Port: 3000

------------------------------------------------------------------------

## Step 1 --- Create directory structure

``` bash
sudo mkdir -p /opt/observability/grafana/{bin,config,data,logs,provisioning/{datasources,dashboards}}
sudo chown -R observability:observability /opt/observability/grafana
```

------------------------------------------------------------------------

## Step 2 --- Download Grafana

``` bash
cd /tmp

wget https://dl.grafana.com/oss/release/grafana-11.1.0.linux-amd64.tar.gz

tar -xvf grafana-*.tar.gz
```

------------------------------------------------------------------------

## Step 3 --- Install Grafana

``` bash
sudo cp -r grafana-*/. /opt/observability/grafana/

sudo chown -R observability:observability /opt/observability/grafana
```

------------------------------------------------------------------------

## Step 4 --- Configure Grafana

Edit config file:

    /opt/observability/grafana/config/custom.ini

Example:

``` ini
[server]
http_port = 3000

[paths]
data = /opt/observability/grafana/data
logs = /opt/observability/grafana/logs
plugins = /opt/observability/grafana/data/plugins
provisioning = /opt/observability/grafana/provisioning

[security]
admin_user = admin
admin_password = StrongPassword123
```

------------------------------------------------------------------------

## Step 5 --- Create systemd service

File:

    /etc/systemd/system/grafana.service

Service:

``` ini
[Unit]
Description=Grafana Visualization Service
After=network-online.target

[Service]

User=observability
Group=observability

WorkingDirectory=/opt/observability/grafana

ExecStart=/opt/observability/grafana/bin/grafana-server   --config=/opt/observability/grafana/config/custom.ini   --homepath=/opt/observability/grafana

Restart=always

StandardOutput=append:/opt/observability/grafana/logs/grafana.log
StandardError=append:/opt/observability/grafana/logs/grafana.log

[Install]
WantedBy=multi-user.target
```

------------------------------------------------------------------------

## Step 6 --- Start Grafana

``` bash
sudo systemctl daemon-reload

sudo systemctl enable grafana

sudo systemctl start grafana
```

------------------------------------------------------------------------

## Step 7 --- Verify service

Check status:

``` bash
sudo systemctl status grafana
```

Check port:

``` bash
ss -lntp | grep 3000
```

------------------------------------------------------------------------

## Step 8 --- Access Grafana

Open browser:

    http://SERVER-IP:3000

Login:

    username: admin
    password: StrongPassword123

Change password after first login.

------------------------------------------------------------------------

## Step 9 --- Add Prometheus datasource

Grafana UI:

Settings → Data Sources → Add

Select:

    Prometheus

URL:

    http://localhost:9090

Click:

    Save & Test

Expected:

    Data source is working

------------------------------------------------------------------------

## Step 10 --- Add Loki datasource

Settings → Data Sources → Add

Select:

    Loki

URL:

    http://localhost:3100

Save & Test

Expected:

    Connected

------------------------------------------------------------------------

## Step 11 --- Import dashboards

Grafana → Dashboards → Import

Recommended:

Node Exporter dashboard ID:

    1860

Blackbox dashboard ID:

    7587

------------------------------------------------------------------------

## Step 12 --- Log location

    /opt/observability/grafana/logs/grafana.log

View:

``` bash
tail -f grafana.log
```

------------------------------------------------------------------------

## Troubleshooting

Check logs:

``` bash
journalctl -u grafana -n 100
```

Check port conflict:

``` bash
ss -lntp | grep 3000
```

Restart:

``` bash
sudo systemctl restart grafana
```

------------------------------------------------------------------------

## Result

Grafana is now running and connected to:

• Prometheus\
• Loki

Dashboards can now visualize metrics and logs.

------------------------------------------------------------------------

Author: Avinash Chavan\
Role: DevOps / Cloud Engineer
[GitHub Profile](https://github.com/chavanavinassh)