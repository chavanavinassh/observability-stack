# Grafana Tempo Production Deployment -- Enterprise Observability SOP

## Overview

This guide provides step-by-step instructions to deploy Grafana Tempo in
a production environment.

Tempo is a distributed tracing backend used to:

• Collect traces from applications\
• Integrate with OpenTelemetry\
• Store traces efficiently\
• Correlate metrics, logs, and traces\
• Visualize traces in Grafana

Tempo completes the observability stack alongside Prometheus and Loki.

------------------------------------------------------------------------

## Server Information

OS: Ubuntu 22.04 LTS\
Component: Grafana Tempo\
Install Path: /opt/observability/tempo\
Service User: observability\
Service Ports:

HTTP: 3200\
OTLP gRPC: 4317\
OTLP HTTP: 4318

------------------------------------------------------------------------

## Step 1 --- Create directory structure

``` bash
sudo mkdir -p /opt/observability/tempo/{bin,config,data/{traces,wal},logs}
sudo chown -R observability:observability /opt/observability/tempo
```

------------------------------------------------------------------------

## Step 2 --- Download Tempo

``` bash
cd /tmp

wget https://github.com/grafana/tempo/releases/latest/download/tempo-linux-amd64.tar.gz

tar -xvf tempo-linux-amd64.tar.gz
```

------------------------------------------------------------------------

## Step 3 --- Install binary

``` bash
sudo mv tempo-linux-amd64 /opt/observability/tempo/bin/tempo

sudo chmod +x /opt/observability/tempo/bin/tempo
```

------------------------------------------------------------------------

## Step 4 --- Create Tempo configuration

File:

    /opt/observability/tempo/config/tempo.yml

Example production configuration:

``` yaml
server:
  http_listen_port: 3200

distributor:
  receivers:

    otlp:
      protocols:
        grpc:
        http:

storage:
  trace:
    backend: local
    local:
      path: /opt/observability/tempo/data/traces

compactor:
  compaction:
    block_retention: 168h

ingester:
  trace_idle_period: 10s
  max_block_duration: 5m
```

------------------------------------------------------------------------

## Step 5 --- Create systemd service

File:

    /etc/systemd/system/tempo.service

Service:

``` ini
[Unit]
Description=Grafana Tempo Tracing Backend
After=network-online.target

[Service]

User=observability
Group=observability

ExecStart=/opt/observability/tempo/bin/tempo   -config.file=/opt/observability/tempo/config/tempo.yml

Restart=always

StandardOutput=append:/opt/observability/tempo/logs/tempo.log
StandardError=append:/opt/observability/tempo/logs/tempo.log

[Install]
WantedBy=multi-user.target
```

------------------------------------------------------------------------

## Step 6 --- Start Tempo

``` bash
sudo systemctl daemon-reload

sudo systemctl enable tempo

sudo systemctl start tempo
```

------------------------------------------------------------------------

## Step 7 --- Verify service

Check status:

``` bash
sudo systemctl status tempo
```

Check ports:

``` bash
ss -lntp | grep 3200
ss -lntp | grep 4317
ss -lntp | grep 4318
```

Health check:

``` bash
curl http://localhost:3200/ready
```

Expected:

    ready

------------------------------------------------------------------------

## Step 8 --- Verify metrics

``` bash
curl http://localhost:3200/metrics
```

Expected:

    tempo_build_info

------------------------------------------------------------------------

## Step 9 --- Grafana integration

Open Grafana

Settings → Data Sources → Add

Select:

    Tempo

URL:

    http://localhost:3200

Click:

    Save & Test

Expected:

    Data source connected

------------------------------------------------------------------------

## Step 10 --- Log location

    /opt/observability/tempo/logs/tempo.log

View:

``` bash
tail -f tempo.log
```

------------------------------------------------------------------------

## Troubleshooting

Check logs:

``` bash
journalctl -u tempo -n 100
```

Check config:

``` bash
/opt/observability/tempo/bin/tempo -config.file=/opt/observability/tempo/config/tempo.yml -verify-config=true
```

Check port conflicts:

``` bash
ss -lntp
```

------------------------------------------------------------------------

## Result

Tempo is now running and ready to receive traces from OpenTelemetry and
applications.

Traces can be visualized and correlated with logs and metrics in
Grafana.

------------------------------------------------------------------------

Author: Avinash Chavan\
Role: DevOps / Cloud Engineer
[GitHub Profile](https://github.com/chavanavinassh)