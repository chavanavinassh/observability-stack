# Blackbox Exporter Production Deployment -- Enterprise Observability SOP

## Overview

This guide provides step-by-step instructions to deploy the Prometheus
Blackbox Exporter in a production environment.\
Blackbox Exporter enables monitoring of:

• HTTP / HTTPS uptime\
• SSL certificate expiry\
• TCP connectivity\
• DNS resolution

It integrates directly with Prometheus using file-based service
discovery.

------------------------------------------------------------------------

## Server Information

OS: Ubuntu 22.04 LTS\
Component: Blackbox Exporter\
Install Path: /opt/observability/exporters/blackbox\
Service User: observability\
Service Port: 9115

------------------------------------------------------------------------

## Step 1 --- Create directory structure

``` bash
sudo mkdir -p /opt/observability/exporters/blackbox/{bin,config,logs}
sudo chown -R observability:observability /opt/observability/exporters/blackbox
```

------------------------------------------------------------------------

## Step 2 --- Download Blackbox Exporter

``` bash
cd /tmp

wget https://github.com/prometheus/blackbox_exporter/releases/latest/download/blackbox_exporter-0.25.0.linux-amd64.tar.gz

tar -xvf blackbox_exporter-*.tar.gz
```

------------------------------------------------------------------------

## Step 3 --- Install binary

``` bash
sudo cp blackbox_exporter-*/blackbox_exporter /opt/observability/exporters/blackbox/bin/

sudo chmod +x /opt/observability/exporters/blackbox/bin/blackbox_exporter
```

------------------------------------------------------------------------

## Step 4 --- Create configuration file

File:

    /opt/observability/exporters/blackbox/config/blackbox.yml

Example production configuration:

``` yaml
modules:

  http_2xx:
    prober: http
    timeout: 10s
    http:
      preferred_ip_protocol: ip4
      valid_status_codes: []

  https_2xx:
    prober: http
    timeout: 10s
    http:
      preferred_ip_protocol: ip4
      tls_config:
        insecure_skip_verify: false

  tcp_connect:
    prober: tcp
    timeout: 5s

  dns_lookup:
    prober: dns
    timeout: 5s
```

------------------------------------------------------------------------

## Step 5 --- Create systemd service

File:

    /etc/systemd/system/blackbox_exporter.service

Service:

``` ini
[Unit]
Description=Prometheus Blackbox Exporter
After=network-online.target

[Service]

User=observability
Group=observability

ExecStart=/opt/observability/exporters/blackbox/bin/blackbox_exporter   --config.file=/opt/observability/exporters/blackbox/config/blackbox.yml   --web.listen-address=:9115

Restart=always

StandardOutput=append:/opt/observability/exporters/blackbox/logs/blackbox.log
StandardError=append:/opt/observability/exporters/blackbox/logs/blackbox.log

[Install]
WantedBy=multi-user.target
```

------------------------------------------------------------------------

## Step 6 --- Start service

``` bash
sudo systemctl daemon-reload

sudo systemctl enable blackbox_exporter

sudo systemctl start blackbox_exporter
```

------------------------------------------------------------------------

## Step 7 --- Verify service

Check status:

``` bash
sudo systemctl status blackbox_exporter
```

Check port:

``` bash
ss -lntp | grep 9115
```

Test probe:

``` bash
curl "http://localhost:9115/probe?target=https://google.com&module=https_2xx"
```

Expected output contains:

    probe_success 1

------------------------------------------------------------------------

## Step 8 --- Prometheus integration

Create target file:

    /opt/observability/targets/blackbox/example.yml

Example:

``` yaml
- targets:
  - https://google.com
  labels:
    project: external
    service: google
    environment: prod
    probe: http
    bb_module: https_2xx
```

Reload Prometheus:

``` bash
sudo systemctl restart prometheus
```

------------------------------------------------------------------------

## Step 9 --- Verify in Prometheus

Open:

http://SERVER-IP:9090

Navigate:

Status → Targets

Expected:

    blackbox-http   UP
    blackbox-ssl    UP

------------------------------------------------------------------------

## Step 10 --- Log location

    /opt/observability/exporters/blackbox/logs/blackbox.log

View:

``` bash
tail -f blackbox.log
```

------------------------------------------------------------------------

## Troubleshooting

Check logs:

``` bash
journalctl -u blackbox_exporter -n 100
```

Check config:

``` bash
/opt/observability/exporters/blackbox/bin/blackbox_exporter --config.check --config.file=/opt/observability/exporters/blackbox/config/blackbox.yml
```

------------------------------------------------------------------------

## Result

Blackbox Exporter is now running and integrated with Prometheus for:

• Website uptime monitoring\
• SSL certificate monitoring\
• TCP connectivity monitoring\
• DNS monitoring

------------------------------------------------------------------------

Author: Avinash Chavan\
Role: DevOps / Cloud Engineer
[GitHub Profile](https://github.com/chavanavinassh)
