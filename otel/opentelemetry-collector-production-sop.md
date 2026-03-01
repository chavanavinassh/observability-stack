# OpenTelemetry Collector Production Deployment -- Enterprise Observability SOP

## Overview

This guide provides step-by-step instructions to deploy the
OpenTelemetry Collector in a production environment.

The OpenTelemetry Collector is used to:

• Collect traces from applications\
• Collect metrics and logs\
• Forward traces to Grafana Tempo\
• Forward metrics to Prometheus\
• Act as a centralized telemetry pipeline

It is vendor‑neutral and supports OTLP, Prometheus, Jaeger, and Zipkin.

------------------------------------------------------------------------

## Server Information

OS: Ubuntu 22.04 LTS\
Component: OpenTelemetry Collector\
Install Path: /opt/observability/otel\
Service User: observability

Ports:

OTLP gRPC: 4317\
OTLP HTTP: 4318

------------------------------------------------------------------------

## Step 1 --- Create directory structure

``` bash
sudo mkdir -p /opt/observability/otel/{bin,config,logs}
sudo chown -R observability:observability /opt/observability/otel
```

------------------------------------------------------------------------

## Step 2 --- Download OpenTelemetry Collector

``` bash
cd /tmp

wget https://github.com/open-telemetry/opentelemetry-collector-releases/releases/latest/download/otelcol-linux-amd64.tar.gz

tar -xvf otelcol-linux-amd64.tar.gz
```

------------------------------------------------------------------------

## Step 3 --- Install binary

``` bash
sudo mv otelcol /opt/observability/otel/bin/

sudo chmod +x /opt/observability/otel/bin/otelcol
```

------------------------------------------------------------------------

## Step 4 --- Create configuration

File:

    /opt/observability/otel/config/otelcol.yml

Example production configuration:

``` yaml
receivers:

  otlp:
    protocols:
      grpc:
      http:

processors:

  batch:

exporters:

  otlp:
    endpoint: localhost:4317
    tls:
      insecure: true

  logging:

service:

  pipelines:

    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlp, logging]
```

This configuration receives traces and forwards them to Tempo.

------------------------------------------------------------------------

## Step 5 --- Create systemd service

File:

    /etc/systemd/system/otelcol.service

Service:

``` ini
[Unit]
Description=OpenTelemetry Collector
After=network-online.target

[Service]

User=observability
Group=observability

ExecStart=/opt/observability/otel/bin/otelcol   --config=/opt/observability/otel/config/otelcol.yml

Restart=always

StandardOutput=append:/opt/observability/otel/logs/otelcol.log
StandardError=append:/opt/observability/otel/logs/otelcol.log

[Install]
WantedBy=multi-user.target
```

------------------------------------------------------------------------

## Step 6 --- Start service

``` bash
sudo systemctl daemon-reload

sudo systemctl enable otelcol

sudo systemctl start otelcol
```

------------------------------------------------------------------------

## Step 7 --- Verify service

Check status:

``` bash
sudo systemctl status otelcol
```

Check ports:

``` bash
ss -lntp | grep 4317
ss -lntp | grep 4318
```

------------------------------------------------------------------------

## Step 8 --- Verify logs

``` bash
tail -f /opt/observability/otel/logs/otelcol.log
```

Expected:

    Everything is ready

------------------------------------------------------------------------

## Step 9 --- Tempo integration

Ensure Tempo is running:

    http://localhost:3200

Collector exports traces to Tempo OTLP endpoint:

    localhost:4317

------------------------------------------------------------------------

## Step 10 --- Grafana verification

Open Grafana

Explore → Tempo

Search traces

------------------------------------------------------------------------

## Troubleshooting

Check logs:

``` bash
journalctl -u otelcol -n 100
```

Validate config:

``` bash
/opt/observability/otel/bin/otelcol --config=/opt/observability/otel/config/otelcol.yml --dry-run
```

Check ports:

``` bash
ss -lntp
```

------------------------------------------------------------------------

## Result

OpenTelemetry Collector is now running and forwarding telemetry data to
Tempo.

Supports:

• Distributed tracing\
• Metrics pipelines\
• Log pipelines

------------------------------------------------------------------------

Author: Avinash Chavan\
Role: DevOps / Cloud Engineer
[GitHub Profile](https://github.com/chavanavinassh)