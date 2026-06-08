# Homelab Network Infrastructure

## Overview

This project documents the design, build, and validation of a structured homelab network environment using UniFi networking equipment, Raspberry Pi DNS services, Docker-based monitoring, and a rack-mounted physical infrastructure.

The purpose of this lab is to create a semi-professional environment for hands-on practice with networking, Linux administration, Docker, DNS, monitoring, VLAN segmentation, firewall policy, VPN access, and software-defined networking concepts.

This project is intended to demonstrate practical infrastructure skills through a real working network rather than a simulated-only environment.

## Project Goals

The main goals of this project are:

* Build a structured 12U network rack for home lab infrastructure.
* Centralize routing, switching, wireless access, DNS, monitoring, and server services.
* Use a managed UniFi network stack for gateway, switch, access point, DHCP, and future VLAN segmentation.
* Deploy Pi-hole and Unbound for network-wide DNS filtering and recursive DNS resolution.
* Run Docker services on an Ubuntu-based mini PC.
* Monitor key network services using Uptime Kuma.
* Document the network using professional-style diagrams, IP plans, port maps, and validation tests.
* Prepare the environment for future projects involving VLANs, guest networking, IoT isolation, VPN access, and SDN labs.

## Current Network Topology

```text
Spectrum / ISP Coax
        |
        v
Netgear Cable Modem
        |
        v
UniFi Gateway
        |
        v
UniFi USW Lite 8 PoE Switch
        |
        +--> Patch Panel
        |       +--> Room 1 Ethernet Drop
        |       +--> Room 2 Ethernet Drop
        |       +--> UniFi Access Point
        |
        +--> Raspberry Pi
        |       +--> Pi-hole
        |       +--> Unbound
        |
        +--> HP Mini PC
        |       +--> Ubuntu Server
        |       +--> Docker
        |       +--> Portainer
        |       +--> Uptime Kuma
        |
        +--> WD My Cloud / Network Storage
```

## Hardware Used

| Device                       | Role                                           |
| ---------------------------- | ---------------------------------------------- |
| Netgear Cable Modem          | ISP modem                                      |
| UniFi Gateway                | Router, firewall, DHCP server, network gateway |
| UniFi USW Lite 8 PoE         | Main managed switch                            |
| UniFi U6+ Access Point       | Wireless access point                          |
| 24-port Keystone Patch Panel | Structured Ethernet termination                |
| Raspberry Pi                 | Pi-hole and Unbound DNS server                 |
| HP Mini PC                   | Ubuntu Docker host                             |
| WD My Cloud                  | Network storage                                |
| 12U Network Rack             | Central equipment rack                         |
| Rack PDU                     | Power distribution                             |
| AC Infinity Rack Fan         | Rack cooling / airflow                         |

## Rack Layout

```text
Top / Roof: AC Infinity Rack Fan

U12  Blank / Airflow
U11  24-port Keystone Patch Panel
U10  UniFi Switch
U9   UniFi Gateway
U8   Raspberry Pi Rack Mount
U7   HP Mini PC Shelf
U6   Blank / Airflow
U5   Future Expansion
U4   Future Expansion
U3   Future Expansion
U2   Rack PDU / Power Strip
U1   Blank / Future UPS
```

## Network Services

### UniFi Gateway

The UniFi Gateway is used as the main router, firewall, DHCP server, and default gateway.

Current default network:

| Setting           | Value                         |
| ----------------- | ----------------------------- |
| Gateway IP        | `192.168.1.1`                 |
| Subnet            | `192.168.1.0/24`              |
| DHCP Server       | UniFi Gateway                 |
| DHCP Range        | `192.168.1.6 - 192.168.1.254` |
| Domain            | `localdomain`                 |
| Client DNS Server | `192.168.1.120`               |

### Pi-hole

Pi-hole provides network-wide DNS filtering for client devices.

| Setting           | Value                        |
| ----------------- | ---------------------------- |
| Hostname          | `pihole`                     |
| IP Address        | `192.168.1.120`              |
| Role              | DNS filtering                |
| DHCP              | Disabled                     |
| Upstream Resolver | Unbound                      |
| Admin URL         | `http://192.168.1.120/admin` |

### Unbound

Unbound is used as a recursive DNS resolver for Pi-hole.

| Setting          | Value            |
| ---------------- | ---------------- |
| Host             | Raspberry Pi     |
| Address          | `127.0.0.1`      |
| Port             | `5335`           |
| Pi-hole Upstream | `127.0.0.1#5335` |

DNS flow:

```text
Client Device
      |
      v
Pi-hole: 192.168.1.120
      |
      v
Unbound: 127.0.0.1#5335
      |
      v
Authoritative DNS / Internet
```

### Docker Host

The HP Mini PC runs Ubuntu Server and hosts Docker-based services.

| Setting              | Value           |
| -------------------- | --------------- |
| Hostname             | `hp-homelab`    |
| Current IP           | `192.168.1.216` |
| OS                   | Ubuntu Server   |
| Container Management | Portainer       |
| Monitoring           | Uptime Kuma     |

Current Docker services:

| Service     | URL                          | Purpose            |
| ----------- | ---------------------------- | ------------------ |
| Portainer   | `https://192.168.1.216:9443` | Docker management  |
| Uptime Kuma | `http://192.168.1.216:3001`  | Service monitoring |

## Monitoring

Uptime Kuma is used to monitor core infrastructure and services.

Current / planned monitors:

| Monitor               | Type  | Target                       |
| --------------------- | ----- | ---------------------------- |
| Gateway / Router      | Ping  | `192.168.1.1`                |
| Pi-hole               | Ping  | `192.168.1.120`              |
| Pi-hole Web Dashboard | HTTP  | `http://192.168.1.120/admin` |
| Portainer             | HTTPS | `https://192.168.1.216:9443` |
| Uptime Kuma           | HTTP  | `http://192.168.1.216:3001`  |
| Homelab Mini PC       | Ping  | `192.168.1.216`              |
| Cloudflare DNS        | Ping  | `1.1.1.1`                    |
| Google DNS            | Ping  | `8.8.8.8`                    |

## Validation

The network was validated using client DHCP checks, DNS lookups, Pi-hole query logs, Unbound tests, SSH access, Docker container status, and Uptime Kuma monitoring.

### Windows Client DHCP / DNS Validation

Command:

```powershell
ipconfig /all
```

Confirmed result:

```text
IPv4 Address: 192.168.1.200
Default Gateway: 192.168.1.1
DHCP Server: 192.168.1.1
DNS Servers: 192.168.1.120
```

### Pi-hole DNS Validation

Command:

```powershell
nslookup google.com
```

Confirmed result:

```text
Server:  pi.hole
Address: 192.168.1.120
```

### Pi-hole Blocking Validation

Command:

```powershell
nslookup doubleclick.net
```

Confirmed result:

```text
Addresses:
::
0.0.0.0
```

### Unbound Validation

Command:

```bash
dig google.com @127.0.0.1 -p 5335
```

Confirmed result:

```text
status: NOERROR
SERVER: 127.0.0.1#5335
```

### Docker Validation

Command:

```bash
docker ps
```

Confirmed running containers:

```text
portainer
uptime-kuma
```

## Skills Demonstrated

This project demonstrates hands-on experience with:

* Network rack planning and hardware installation
* Structured Ethernet cabling and patch panel organization
* UniFi gateway, switch, and access point management
* DHCP and DNS configuration
* Pi-hole DNS filtering
* Unbound recursive DNS resolution
* Linux server administration
* Docker container hosting
* Portainer container management
* Uptime Kuma monitoring
* Network troubleshooting with `ping`, `nslookup`, `dig`, `ipconfig`, and SSH
* Technical documentation and infrastructure validation

## Security and Best Practices

Current best practices applied:

* DHCP handled centrally by the UniFi Gateway.
* DNS handled centrally by Pi-hole.
* Pi-hole DHCP disabled to avoid DHCP conflicts.
* Infrastructure devices assigned or planned for fixed IP reservations.
* Administrative services are kept internal only.
* No SSH, Portainer, Pi-hole, or Uptime Kuma ports are exposed directly to the public internet.
* Power and Ethernet cabling are routed separately inside the rack.
* Documentation is maintained for topology, IP addressing, services, and validation.

## Future Improvements

Planned future work includes:

* Create a dedicated Guest VLAN and guest Wi-Fi network.
* Add IoT VLAN for smart devices and consumer electronics.
* Add Lab VLAN for Docker, SDN, and security experiments.
* Configure firewall rules between VLANs.
* Add VPN access using WireGuard, UniFi Teleport, or Tailscale.
* Add Grafana or Homepage dashboard for central visibility.
* Add secondary Pi-hole for DNS redundancy.
* Add rack temperature monitoring.
* Add UPS backup and power monitoring.
* Build SDN lab using Open vSwitch, Mininet, and a controller such as Ryu.

## Planned VLAN Design

| Network         | Example Subnet    | Purpose                                  |
| --------------- | ----------------- | ---------------------------------------- |
| Main LAN        | `192.168.1.0/24`  | Trusted devices and core services        |
| Guest VLAN      | `192.168.30.0/24` | Guest Wi-Fi, internet only               |
| IoT VLAN        | `192.168.20.0/24` | Smart TVs, sensors, consumer devices     |
| Lab VLAN        | `192.168.40.0/24` | Docker, test systems, SDN labs           |
| Management VLAN | `192.168.99.0/24` | Future infrastructure management network |

## Portfolio Purpose

This project is part of a larger homelab portfolio focused on practical infrastructure, networking, and systems administration experience.

The lab is designed to support future portfolio projects including:

* Segmented network design with VLANs and firewall rules
* Secure VPN-based remote access
* Dockerized monitoring and dashboard services
* Network-wide DNS filtering and recursive DNS
* Open vSwitch / Mininet SDN lab environments
* Infrastructure documentation and change tracking

## Repository Structure

Planned repository layout:

```text
homelab-network-infrastructure/
├── README.md
├── docs/
│   ├── 01-rack-build.md
│   ├── 02-pihole-unbound.md
│   ├── 03-docker-monitoring.md
│   ├── 04-vlan-segmentation.md
│   ├── 05-vpn-remote-access.md
│   └── maintenance-log.md
├── diagrams/
│   ├── network-topology.png
│   ├── rack-layout.png
│   └── vlan-plan.png
├── screenshots/
│   ├── unifi/
│   ├── pihole/
│   ├── uptime-kuma/
│   └── portainer/
└── configs/
    ├── docker-compose/
    └── sanitized-examples/
```

## Notes

Sensitive information should not be committed to this repository.

Do not include:

* Passwords
* API keys
* SSH private keys
* Public IP addresses
* Device serial numbers
* QR codes
* Recovery codes
* Full account screenshots containing private information

Private LAN IP addresses such as `192.168.1.x` are acceptable for documentation.

