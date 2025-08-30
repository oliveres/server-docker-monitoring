# Server & Docker Monitoring with Logs

## Introduction

Introducing the server monitoring solution using Grafana, Prometheus, Loki, Promtail, Cadvisor, and Node-Exporter Stack! This project aims to provide a comprehensive and user-friendly way to monitor the performance of your server and view container logs in real-time. With Grafana's intuitive dashboards, you can easily visualize system metrics collected by Prometheus and Cadvisor, while Node-Exporter provides valuable information about the server hardware and operating system. Loki and Promtail add powerful log aggregation capabilities, allowing you to view and search through container logs directly in Grafana. The combination of these tools results in a powerful and efficient monitoring solution that will give you complete visibility into your system's health and application logs. Check out the project and take your server monitoring to the next level!

This repository contains a `docker-compose` file to run a monitoring stack. It is based on the following projects:
- [Prometheus](https://prometheus.io/) - Metrics collection and storage
- [Grafana](http://grafana.org/) - Visualization and dashboards
- [Loki](https://grafana.com/oss/loki/) - Log aggregation system
- [Promtail](https://grafana.com/docs/loki/latest/clients/promtail/) - Log collector for Docker containers
- [cAdvisor](https://github.com/google/cadvisor) - Container metrics
- [NodeExporter](https://github.com/prometheus/node_exporter) - Host metrics

This repository is compatible with Portainer Stack deployment.

## Prerequisites

Before we get started installing the stack, we need to make sure that the following prerequisites are met:
- Docker is installed on the host machine
- Docker Compose is installed on the host machine
- The host machine is running a compatible Linux distribution

## Repository structure

```
.
├── docker-compose.yml
├── grafana/
│   ├── Dockerfile
│   ├── .env
│   └── provisioning/
│       ├── dashboards/
│       │   ├── dashboard.yml
│       │   ├── monitoring.json
│       │   └── container-logs.json
│       └── datasources/
│           ├── datasource.yml
│           └── loki-datasource.yml
├── prometheus/
│   ├── Dockerfile
│   └── prometheus.yml
├── loki/
│   ├── Dockerfile
│   └── loki-config.yaml
└── promtail/
    ├── Dockerfile
    └── promtail-config.yaml
```

## Installation and Configuration

To install the stack, follow the steps below:

- Clone this repository to your host machine.
```bash
git clone https://github.com/oliveres/server-docker-monitoring.git
```

- Enter to the cloned directory.
```bash
cd server-docker-monitoring
```

 - Create `data` directory and change the ownership of the `prometheus` and `grafana` folders for a nice and clean installation.
```bash
mkdir -p prometheus/data grafana/data && \
sudo chown -R 472:472 grafana/ && \
sudo chown -R 65534:65534 prometheus/
```

 - Start the stack with `docker-compose`.
```bash
docker-compose up -d
```

This will start all the containers and make them available on the host machine.
<br/>The following ports are used (only Grafana is exposed on the host machine):
- 3000: Grafana (mapped to 3001 on host)
- 9090: Prometheus (internal)
- 3100: Loki (internal)
- 9080: Promtail (internal)
- 8080: cAdvisor (internal)
- 9100: NodeExporter (internal)

The Grafana dashboard can be accessed by navigating to `http://<host-ip>:3001` in your browser for example `http://192.168.1.100:3001`.
<br/>The default username and password are both `admin`. You will be prompted to change the password on the first login.
<br/>Credentials can be changed by editing the [.env](grafana/.env) file.

If you would like to change which targets should be monitored, you can edit the [prometheus.yml](prometheus/prometheus.yml) file.
<br/>The targets section contains a list of all the targets that should be monitored by Prometheus.
<br/>The names defined in the `job_name` section are used to identify the targets in Grafana.
<br/>The `static_configs` section contains the IP addresses of the targets that should be monitored. Actually, they are sourced from the service names defined in the [docker-compose.yml](docker-compose.yml) file.
<br/>If you think that the `scrape_interval` value is too aggressive, you can change it to a more suitable value.

In order to check if the stack is running correctly, you can run the following command:
```bash
docker-compose ps
```

View the logs of a specific container by running the following command:
```bash
docker logs -f <container-name>
```

## Add Data Sources and Dashboards

Since Grafana v5 has introduced the concept of provisioning, it is possible to automatically add data sources and dashboards to Grafana.
<br/>This is done by placing the `datasources` and `dashboards` directories in the [provisioning](grafana/provisioning) folder. The files in these directories are automatically loaded by Grafana on startup.

The stack comes pre-configured with:
- **Prometheus datasource** - for metrics visualization
- **Loki datasource** - for log aggregation and search
- **System monitoring dashboard** - displays host and container metrics
- **Container logs dashboard** - displays real-time logs from all containers

If you like to add a new dashboard, simply place the JSON file in the [dashboards](grafana/provisioning/dashboards) directory, and it will be automatically loaded next time Grafana is started.

## Viewing Container Logs

The stack includes Loki and Promtail for log aggregation:

1. Navigate to Grafana at `http://<host-ip>:3001`
2. Go to **Dashboards** → **Container Logs**
3. You can see real-time logs from all monitoring containers
4. Use the search bar to filter logs by container name or content
5. Alternatively, go to **Explore** and select **Loki** datasource for advanced log queries

Example Loki queries:
- `{container="monitoring-grafana"}` - Show logs from Grafana container
- `{job="docker"} |= "error"` - Show all logs containing "error"
- `{container=~"monitoring-.*"} |= "warning"` - Show warnings from all monitoring containers

# Install Dashboard from Grafana.com (Optional)

If you would like to install this dashboard from Grafana.com, simply follow the steps below:
- Navigate to the dashboard on [Grafana.com Dashboard](https://grafana.com/grafana/dashboards/15120-raspberry-pi-docker-monitoring/)
- Click on the `Copy ID to Clipboard` button
- Navigate to the `Import` page in Grafana
- Paste the ID into the `Import via grafana.com` field
- Click on the `Load` button
- Click on the `Import` button

Or you can follow the steps described in the [Grafana Documentation](https://grafana.com/docs/grafana/latest/dashboards/manage-dashboards/#import-a-dashboard).

This dashboard is intended to help you get started with monitoring your server. If you have any changes or suggestions, you would like to see, please feel free to open an issue or create a pull request.

Here is a screenshot of the dashboard:
![Grafana Dashboard](grafana/screenshots/dashboard.png)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

This project is a fork of the Raspberry Pi monitoring solution by oijnk (https://github.com/oijkn/Docker-Raspberry-PI-Monitoring)

## Troubleshooting

Enable `c-group` memory and swap accounting on the host machine by running the following command:
```bash
sudo sed -i 's/^GRUB_CMDLINE_LINUX=""/GRUB_CMDLINE_LINUX="cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1"/' /etc/default/grub
sudo update-grub
sudo reboot
```
