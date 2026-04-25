# 🚀 Observability Stack – Prometheus, Grafana & Nginx Monitoring

![Docker](https://img.shields.io/badge/Docker-Containerized-blue)
![Prometheus](https://img.shields.io/badge/Monitoring-Prometheus-orange)
![Grafana](https://img.shields.io/badge/Visualization-Grafana-yellow)

---

## 📌 Overview

This project implements a **monitoring stack in a homelab environment** to provide visibility into:

- System resources (CPU, RAM, Disk, Network)
- Docker containers
- Nginx reverse proxy performance

The stack is based on **Prometheus (metrics collection)** and **Grafana (visualization)**.

---

## 🎯 Objectives

- Build a production-like monitoring environment  
- Understand Prometheus scraping model  
- Monitor host-level and application-level metrics  
- Troubleshoot real-world issues (networking, SELinux, exporters)

---

## 🧱 Architecture

```
Node Exporter  ─┐
cAdvisor       ─┼──→ Prometheus ───→ Grafana
nginx-exporter ─┘
```

---

## ⚙️ Stack Components

| Component         | Role |
|------------------|------|
| Prometheus       | Metrics collection |
| Grafana          | Visualization |
| Node Exporter    | Host metrics |
| cAdvisor         | Docker metrics |
| Nginx Exporter   | Nginx metrics |

---

## 🌐 Nginx Monitoring

Nginx runs on the **host**, not in Docker.

Metrics are exposed using:

```nginx
server {
    listen 0.0.0.0:8082;

    location /nginx_status {
        stub_status;
        allow 127.0.0.1;
        allow 172.17.0.0/16;
        deny all;
    }
}
```

These metrics are scraped using:

```
nginx-prometheus-exporter
```

---

## 🚀 Implementation

### 🔹 Docker Compose

```yaml
version: "3.8"

services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9091:9090"
    extra_hosts:
      - "host.docker.internal:host-gateway"

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"

  node-exporter:
    image: prom/node-exporter
    network_mode: host

  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    ports:
      - "8081:8080"

  nginx-exporter:
    image: nginx/nginx-prometheus-exporter
    command:
      - -nginx.scrape-uri=http://host.docker.internal:8082/nginx_status
    ports:
      - "9113:9113"
```

---

### 🔹 Prometheus Configuration

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: "node"
    static_configs:
      - targets: ["host.docker.internal:9100"]

  - job_name: "cadvisor"
    static_configs:
      - targets: ["cadvisor:8080"]

  - job_name: "nginx"
    static_configs:
      - targets: ["nginx-exporter:9113"]
```

---

## 📊 Dashboards

Imported dashboards from Grafana:

- **1860** → Node Exporter Full  
- **193** → cAdvisor (Docker monitoring)  
- **12708** → Nginx Monitoring  

---

## ⚠️ Challenges & Troubleshooting

### ❌ Prometheus scraping `localhost`
- Cause: container isolation  
- Fix: use `host.docker.internal`

---

### ❌ Docker DNS / networking issues
- Cause: services not reachable  
- Fix: correct targets and networking

---

### ❌ SELinux blocking nginx port

```bash
semanage port -a -t http_port_t -p tcp 8082
```

---

### ❌ Exporter returning only internal metrics
- Cause: nginx not reachable  
- Fix: expose `stub_status` properly

---

## 📈 Results

- Real-time monitoring (5s interval)
- Full visibility into system resources
- Container-level monitoring
- Nginx metrics (connections, requests)

---

## 🧠 Key Learnings

- Prometheus pull-based monitoring model  
- Docker networking and service discovery  
- Exporter-based observability  
- Troubleshooting distributed systems  

---

## 🔐 Future Improvements

- Add alerting (Grafana / Alertmanager)  
- Add logging (Loki / ELK)  
- Monitor additional services  
- Extend to Kubernetes (k3s)  

---

## 💼 Resume Statement

> Built a monitoring stack using Prometheus and Grafana to collect and visualize system and application metrics in a Docker-based environment.
