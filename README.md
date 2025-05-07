# 📡 InfraWatchKit

A **complete infrastructure monitoring & logging toolkit**, designed for engineers who want full control over observability setups in both **Docker-based environments** and **bare-metal servers**.

> This repository is perfect for DevOps professionals, system administrators, and SREs who prefer clarity, control, and customization over black-box tools and pre-baked scripts.

---

## 🔍 What's Inside?

InfraWatchKit provides:

### ✅ Monitoring Stack (Docker & Server)

- **Prometheus** – Metric scraping and alerting
- **Grafana** – Visualization and dashboards
- Predefined targets and scrape configs
- Ready-to-use dashboard JSONs

### ✅ Logging Stack (Docker & Server)

- **Elasticsearch** – Scalable log storage
- **Logstash** – Parsing and pipeline config
- **Kibana** – Log visualization and search
- Filebeat or other log forwarder setup references

### ✅ Docker-Based Setup

- `docker-compose.yml` files for:
  - Prometheus + Grafana stack
  - ELK stack (Elasticsearch, Logstash, Kibana)
- Volume mounts, environment files, networking setup
- Config files structured and mounted properly

### ✅ Manual Server Configuration (No Auto Scripts)

- Docker & Docker Compose installation
- SSH hardening, UFW firewall, system tuning (`sysctl`)
- Folder structure and setup steps for servers without containers
