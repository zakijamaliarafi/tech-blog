---
heroImage: '/prometheus-grafana-monitoring-stack.svg'
title: 'Building a Monitoring Stack with Prometheus and Grafana'
description: 'Deploying a robust, scalable metrics and monitoring pipeline using Prometheus for data collection and Grafana for visualization.'
pubDate: 'May 14 2026'
---

In the modern infrastructure landscape, visibility is paramount. The combination of Prometheus (for scraping and storing metrics) and Grafana (for visualizing data) has become the de-facto standard for open-source monitoring.

## Why Prometheus?

Prometheus operates on a pull model. It actively scrapes HTTP endpoints exposing metrics in a specific text format. It excels in dynamic environments like Kubernetes because it handles service discovery natively.

## Setting Up Prometheus

A basic `prometheus.yml` configuration defines global settings and scrape jobs:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['192.168.1.10:9100', '192.168.1.11:9100']
```

In this example, we configure Prometheus to monitor itself and two other nodes running `node_exporter`, a daemon that exposes system-level metrics (CPU, memory, disk).

## Introducing Grafana

Grafana connects to various data sources, including Prometheus, and allows you to build rich, customizable dashboards.

### Configuring the Data Source
Once Grafana is running, navigate to Data Sources, select Prometheus, and provide the URL (e.g., `http://localhost:9090`).

### Creating Dashboards
You can build dashboards from scratch using PromQL (Prometheus Query Language). For instance, to calculate the CPU usage percentage over a 5-minute rate:
```promql
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

Alternatively, you can import highly polished, community-driven dashboards directly from Grafana.com (e.g., Dashboard ID `1860` for the Node Exporter Full dashboard).

## Alerting

Prometheus includes an Alertmanager component. You define alert rules in Prometheus:
```yaml
groups:
- name: NodeAlerts
  rules:
  - alert: HighMemoryUsage
    expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 90
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High memory usage on {{ $labels.instance }}"
```
Alertmanager handles deduplicating, grouping, and routing these alerts to integrations like Slack, PagerDuty, or Email.

