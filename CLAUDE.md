# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Server monitoring stack for Docker environments using Grafana, Prometheus, cAdvisor, and Node Exporter. Originally forked from Raspberry Pi monitoring solution, compatible with any Linux server running Docker.

## Quick Start Commands

```bash
# Initial setup (run once after cloning)
mkdir -p prometheus/data grafana/data && \
sudo chown -R 472:472 grafana/ && \
sudo chown -R 65534:65534 prometheus/

# Start the monitoring stack
docker-compose up -d

# Check stack status
docker-compose ps

# View logs for specific container
docker logs -f monitoring-grafana
docker logs -f monitoring-prometheus
docker logs -f monitoring-loki
docker logs -f monitoring-promtail
docker logs -f monitoring-cadvisor
docker logs -f monitoring-node-exporter

# Stop the stack
docker-compose down

# Restart a specific service
docker-compose restart grafana
docker-compose restart loki
docker-compose restart promtail

# View Loki logs in Grafana
# Navigate to http://localhost:3001 > Explore > Select "Loki" datasource
# Query examples: {container="monitoring-grafana"} or {job="docker"}
```

## Architecture

The monitoring stack consists of six containerized services:

1. **Grafana** (port 3001): Visualization dashboard, auto-provisioned with datasources and dashboards
2. **Prometheus** (internal:9090): Time-series database, scrapes metrics every 5-15 seconds
3. **Loki** (internal:3100): Log aggregation system with 30-day retention
4. **Promtail** (internal:9080): Log collector that ships Docker container logs to Loki
5. **cAdvisor** (internal:8080): Container metrics collector
6. **Node Exporter** (internal:9100): Host system metrics collector

All services communicate via internal Docker network. Only Grafana is exposed externally on port 3001.

## Key Configuration Files

- `docker-compose.yml`: Service definitions and network configuration
- `grafana/.env`: Grafana admin credentials (default: admin/admin)
- `prometheus/prometheus.yml`: Scrape targets and intervals
- `loki/loki-config.yaml`: Loki storage and retention settings
- `promtail/promtail-config.yaml`: Docker log collection configuration
- `grafana/provisioning/dashboards/rpi-monitoring.json`: System metrics dashboard
- `grafana/provisioning/dashboards/container-logs.json`: Container logs dashboard
- `grafana/provisioning/datasources/datasource.yml`: Prometheus datasource config
- `grafana/provisioning/datasources/loki-datasource.yml`: Loki datasource config

## Development Workflow

### Modifying Prometheus Configuration
Edit `prometheus/prometheus.yml` to:
- Add new scrape targets
- Adjust scrape intervals
- Configure alerting rules

After changes: `docker-compose restart prometheus`

### Adding Grafana Dashboards
Place JSON dashboard files in `grafana/provisioning/dashboards/` directory. They auto-load on Grafana restart.

### Updating Service Versions
Modify Dockerfile in respective directories:
- `grafana/Dockerfile`
- `prometheus/Dockerfile`

Then rebuild: `docker-compose build && docker-compose up -d`

## Data Persistence

- Grafana data: Docker volume `grafana-data`
- Prometheus data: Docker volume `prometheus-data` (1 year retention)
- Loki data: Docker volume `loki-data` (30 days retention)
- Promtail positions: Docker volume `promtail-data`

## Monitoring External Targets

To monitor additional servers/containers, add them to `prometheus/prometheus.yml`:

```yaml
- job_name: 'external-server'
  static_configs:
    - targets: ['192.168.1.100:9100']  # Example external node-exporter
```

## Troubleshooting

If container metrics are missing, ensure cgroup memory accounting is enabled:
```bash
sudo sed -i 's/^GRUB_CMDLINE_LINUX=""/GRUB_CMDLINE_LINUX="cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1"/' /etc/default/grub
sudo update-grub
sudo reboot
```