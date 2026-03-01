# 🚀 Production‑Grade Observability Stack (DevOps Portfolio)

![Linux](https://img.shields.io/badge/Linux-Production-blue?logo=linux)
![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-orange?logo=prometheus)
![Grafana](https://img.shields.io/badge/Grafana-Dashboards-F46800?logo=grafana)
![Loki](https://img.shields.io/badge/Loki-Log%20Aggregation-purple)
![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?logo=docker)
![Status](https://img.shields.io/badge/Status-Production--Ready-success)

------------------------------------------------------------------------

# 📌 Overview

This project demonstrates a **production‑ready observability platform**
built using industry‑standard DevOps tools:

**Prometheus • Grafana • Loki • Promtail • Alertmanager • Blackbox
Exporter • Node Exporter**

It provides:

• Infrastructure monitoring\
• Centralized logging\
• Website and API uptime monitoring\
• Alerting and incident response\
• Production‑ready Linux deployment

This architecture reflects **real‑world DevOps and SRE monitoring
systems** used in cloud and enterprise environments.

------------------------------------------------------------------------

# 🏗 Architecture Diagram

                            ┌────────────────────┐
                            │    Linux Server    │
                            │ (Logs & Metrics)   │
                            └─────────┬──────────┘
                                      │
                     ┌────────────────┴───────────────┐
                     │                                │
            ┌────────▼────────┐              ┌────────▼────────┐
            │  Node Exporter  │              │     Promtail    │
            │ (System Metrics)│              │   (Log Agent)   │
            └────────┬────────┘              └────────┬────────┘
                     │                                │
                     │                                │
               ┌─────▼──────────┐             ┌──────▼─────────┐
               │   Prometheus   │             │      Loki      │
               │ Metrics Storage│             │  Log Storage   │
               └─────┬──────────┘             └──────┬─────────┘
                     │                                │
                     └──────────────┬─────────────────┘
                                    │
                             ┌──────▼──────┐
                             │   Grafana   │
                             │ Dashboards  │
                             └──────┬──────┘
                                    │
                             ┌──────▼──────┐
                             │ Alertmanager│
                             │Notifications│
                             └─────────────┘


    External Monitoring:
    Blackbox Exporter → Prometheus → Grafana

------------------------------------------------------------------------

# 🎯 Key Features

## Infrastructure Monitoring

Monitor:

• CPU usage\
• Memory usage\
• Disk usage\
• Network traffic\
• Server uptime

------------------------------------------------------------------------

## Log Aggregation

Centralized logging from:

• Linux system logs\
• Application logs\
• Docker logs

Enables fast troubleshooting and root cause analysis.

------------------------------------------------------------------------

## Website & API Monitoring

Monitor:

• Website uptime\
• API availability\
• HTTPS response\
• SSL certificate expiry

------------------------------------------------------------------------

## Alerting System

Alert examples:

• High CPU usage\
• Server down\
• Website down\
• Disk full

Alert flow:

Prometheus → Alertmanager → Email / Slack / Webhook

------------------------------------------------------------------------

# 📁 Production Directory Structure

    /opt/observability/

    ├── prometheus/
    ├── grafana/
    ├── loki/
    ├── promtail/
    ├── alertmanager/
    ├── blackbox/
    └── exporters/

Follows **Linux production best practices**.

------------------------------------------------------------------------

# 🖥 Dashboards

Grafana dashboards include:

• Server monitoring dashboard\
• Website uptime dashboard\
• SSL monitoring dashboard\
• Log dashboard

------------------------------------------------------------------------

# 📸 Screenshots

- **Grafana**: 
![Grafana ](https://avi-chavan-96.sirv.com/Projects/observability/grafana.png)

- **Grafana Dashboard**: 
![Dashboard ](https://avi-chavan-96.sirv.com/Projects/observability/grafana-dash.png)

- **Grafana-Metrics**: 
![Metrics ](https://avi-chavan-96.sirv.com/Projects/observability/grafana-metrics.png)

- **Grafana-Logs**: 
![Logs ](https://avi-chavan-96.sirv.com/Projects/observability/grafana-logs.png)

- **Grafana-Datasource**: 
![Datasource ](https://avi-chavan-96.sirv.com/Projects/observability/grafana-datasource.pngg)

- **Grafana-Node**: 
![Node ](https://avi-chavan-96.sirv.com/Projects/observability/grafana-node.png)

- **Prometheus**: 
![Prometheus ](https://avi-chavan-96.sirv.com/Projects/observability/prometheus.png)

![Prometheus ](https://avi-chavan-96.sirv.com/Projects/observability/prometheus1.png)

- **Alertmanager**: 
![Alertmanager ](https://avi-chavan-96.sirv.com/Projects/observability/alertmanager.png)

![Alertmanager-Test ](https://avi-chavan-96.sirv.com/Projects/observability/alertmanager-test.png)

------------------------------------------------------------------------

# 🚀 Deployment

## Linux Deployment

Start services:

systemctl start prometheus
systemctl start loki
systemctl start promtail
systemctl start tempo
systemctl start otelcol
systemctl start grafana
systemctl start alertmanager

Enable auto start:

    sudo systemctl enable prometheus
    sudo systemctl enable grafana


------------------------------------------------------------------------

# 🌐 Access

Grafana\
http://SERVER-IP:3000

Prometheus\
http://SERVER-IP:9090

Alertmanager\
http://SERVER-IP:9093

------------------------------------------------------------------------

# 🔐 Production Security

Supports:

• Reverse proxy (NGINX)\
• HTTPS (Let's Encrypt)\
• Firewall isolation\
• Role‑based dashboards

Example:

grafana.yourdomain.com

------------------------------------------------------------------------

# 📊 Example Prometheus Queries

CPU usage:

    100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

Website uptime:

    probe_success

Logs:

    {job="syslog"}

------------------------------------------------------------------------

# 💼 DevOps Skills Demonstrated

Linux Administration\
Monitoring and Observability\
Prometheus configuration\
Grafana dashboards\
Log aggregation\
Alert configuration\
Production deployment\
Troubleshooting

------------------------------------------------------------------------

# 🧠 Real‑World Use Cases

Cloud infrastructure monitoring\
Production server monitoring\
Application monitoring\
DevOps platform monitoring

------------------------------------------------------------------------

# ⭐ Why This Project Matters

This project demonstrates the ability to design and deploy a **complete
monitoring and logging system**, a critical responsibility of DevOps and
Site Reliability Engineers.

------------------------------------------------------------------------

# 👨‍💻 Author

Avinash Chavan\
DevOps Engineer

[GitHub Profile](https://github.com/chavanavinassh)


## 💬 Feedback & Support

- Raise issues via GitHub Issues tab
- Contact developer via [GitHub Profile](https://github.com/chavanavinassh)


## 📄 License
This project is licensed under the [MIT License](https://mit-license.org/)