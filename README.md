🚀 Observability Stack – Prometheus, Grafana & Nginx Monitoring
📌 Overview

This project implements a monitoring stack in a homelab environment to observe:

System resources (CPU, RAM, Disk, Network)
Docker containers
Nginx reverse proxy activity
🧱 Architecture
Node Exporter  ─┐
cAdvisor       ─┼──→ Prometheus ───→ Grafana
nginx-exporter ─┘
⚙️ Stack Components
Component	Role
Prometheus	Metrics collection
Grafana	Visualization
Node Exporter	Host metrics
cAdvisor	Container metrics
Nginx Exporter	Nginx metrics
🌐 Nginx Monitoring

Nginx runs on the host and exposes metrics using:

server {
    listen 0.0.0.0:8082;

    location /nginx_status {
        stub_status;
        allow 127.0.0.1;
        allow 172.17.0.0/16;
        deny all;
    }
}

Metrics are collected through:

nginx-prometheus-exporter
🚀 Implementation
🔹 Docker Compose
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
🔹 Prometheus Configuration
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
📊 Dashboards
Node Exporter → 1860
cAdvisor → 193
Nginx → 12708
⚠️ Challenges & Troubleshooting
❌ Prometheus scraping issues
Cause: container networking
Fix: use host.docker.internal
❌ SELinux blocking nginx port
semanage port -a -t http_port_t -p tcp 8082
❌ Exporter returning only internal metrics
Cause: nginx not reachable
Fix: expose stub_status correctly
📈 Results
Real-time monitoring (5s interval)
Visibility into system and container performance
Nginx metrics (connections, requests)
🧠 Key Learnings
Prometheus pull model
Docker networking
Exporter-based monitoring
Basic observability principles
🔐 Next Steps
Add alerting (Grafana / Alertmanager)
Add logging (Loki / ELK)
Extend to Kubernetes
✍️ 2. Medium Article (REALISTIC VERSION)

Title:

🚀 Building a Monitoring Stack with Prometheus, Grafana & Nginx
Intro

In this project, I built a monitoring stack in my homelab to better understand observability in real-world systems.

What I implemented
System monitoring using Node Exporter
Container monitoring using cAdvisor
Nginx monitoring using an exporter
Visualization using Grafana
Key challenge

One major issue I faced was:

👉 Prometheus couldn’t scrape services due to Docker networking

Fix:

Use host.docker.internal
Adjust exporter endpoints
Another issue

👉 SELinux blocked nginx on a custom port

Fix:

semanage port -a -t http_port_t -p tcp 8082
Results
Live dashboards with system metrics
Visibility into nginx traffic
Understanding of monitoring pipelines
