# Docker Monitoring Stack

## Overview

This document covers the Docker-based service and monitoring stack running on the HP homelab mini PC.

The mini PC runs Ubuntu Server and hosts infrastructure services using Docker. Portainer is used to manage the Docker environment, and Uptime Kuma is used to monitor network devices and services.

This part of the project shows how the homelab can run self-hosted infrastructure tools that support the rest of the network.

## Goals

The goals for this part of the project were:

* Use the HP mini PC as a dedicated homelab server.
* Run Ubuntu Server as the host operating system.
* Use Docker to host infrastructure services.
* Use Portainer to manage containers through a web interface.
* Use Uptime Kuma to monitor network and service availability.
* Monitor the gateway, Pi-hole, Pi-hole web dashboard, Portainer, Uptime Kuma, and external DNS targets.
* Keep all administrative services internal to the home network.

## Docker Host

| Setting          | Value                        |
| ---------------- | ---------------------------- |
| Device           | HP Mini PC                   |
| Hostname         | `hp-homelab`                 |
| Current IP       | `192.168.1.216`              |
| Operating System | Ubuntu Server                |
| Role             | Docker host / homelab server |
| CPU              | 6 cores                      |
| RAM              | 16 GB                        |

## Current Docker Services

| Service     | Container Name | URL                          | Purpose                        |
| ----------- | -------------- | ---------------------------- | ------------------------------ |
| Portainer   | `portainer`    | `https://192.168.1.216:9443` | Docker management              |
| Uptime Kuma | `uptime-kuma`  | `http://192.168.1.216:3001`  | Network and service monitoring |

## Portainer

Portainer provides a web interface for managing Docker containers, images, volumes, networks, and stacks.

### Portainer Dashboard

![Portainer Dashboard](../screenshots/portainer/portainer-dashboard.png)

The Portainer dashboard shows the local Docker environment running on the homelab mini PC.

## Uptime Kuma

Uptime Kuma is used to monitor key services and devices in the network.

### Uptime Kuma Dashboard

![Uptime Kuma Dashboard](../screenshots/uptime-kuma/uptime-kuma-dashboard.png)

The Uptime Kuma dashboard provides a simple status view for core infrastructure services.

## Current Monitors

| Monitor               | Type  | Target                       | Purpose                                         |
| --------------------- | ----- | ---------------------------- | ----------------------------------------------- |
| Gateway / Router      | Ping  | `192.168.1.1`                | Confirms the gateway is reachable               |
| Pi-hole               | Ping  | `192.168.1.120`              | Confirms the Raspberry Pi is reachable          |
| Pi-hole Web Dashboard | HTTP  | `http://192.168.1.120/admin` | Confirms the Pi-hole web interface is available |
| Portainer             | HTTPS | `https://192.168.1.216:9443` | Confirms Docker management UI is reachable      |
| Uptime Kuma           | HTTP  | `http://192.168.1.216:3001`  | Confirms the monitoring service is reachable    |
| Homelab Mini PC       | Ping  | `192.168.1.216`              | Confirms the Docker host is reachable           |
| Cloudflare DNS        | Ping  | `1.1.1.1`                    | Confirms external internet/DNS reachability     |
| Google DNS            | Ping  | `8.8.8.8`                    | Confirms external internet/DNS reachability     |

## Docker Validation

Docker containers were checked from the homelab mini PC using:

```bash
docker ps
```

Confirmed running containers:

```text
portainer
uptime-kuma
```

Example output:

```text
CONTAINER ID   IMAGE                        STATUS                 PORTS                       NAMES
3d4ef943fc19   louislam/uptime-kuma:2       Up                     0.0.0.0:3001->3001/tcp       uptime-kuma
bccb7f4da889   portainer/portainer-ce:lts   Up                     0.0.0.0:9443->9443/tcp       portainer
```

## SSH Access

The homelab mini PC can be managed from the local network using SSH.

```powershell
ssh rhanly@192.168.1.216
```

SSH is intended for local network administration only. SSH is not exposed directly to the public internet.

## Network Role

The Docker host supports the network by running management and monitoring services.

```text
UniFi Gateway
      |
      v
UniFi Switch
      |
      v
HP Mini PC
      |
      +--> Portainer
      +--> Uptime Kuma
```

## Best Practices Applied

* Docker services run on a dedicated Ubuntu mini PC.
* Portainer is used for Docker management and visibility.
* Uptime Kuma monitors core services and infrastructure.
* Administrative services are kept internal to the LAN.
* No Portainer, Uptime Kuma, or SSH ports are exposed directly to the public internet.
* The homelab mini PC should have a fixed/reserved IP address in UniFi.
* Future remote access should be done through VPN rather than port forwarding.

## Future Improvements

Planned improvements include:

* Move Docker services into documented Docker Compose files.
* Add Homepage or Homarr as a central homelab dashboard.
* Add Grafana and Prometheus for metrics-based monitoring.
* Add automated backups for Docker volumes.
* Add monitoring for disk usage, CPU usage, memory usage, and container health.
* Add VPN access for secure remote administration.
* Add local DNS names for services such as `portainer.home`, `uptime.home`, and `dashboard.home`.
