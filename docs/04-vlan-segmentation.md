# VLAN Segmentation Plan

## Overview

This document outlines the planned VLAN segmentation design for the homelab network.

The current network uses a single default LAN. The goal of the VLAN project is to separate trusted devices, guest devices, IoT devices, and lab systems into different networks with firewall rules controlling traffic between them.

This document is currently a planning document. It will be updated after VLANs are created and tested in UniFi.

## Current Network

The current network uses one default LAN.

| Network     | Subnet           | Purpose                              |
| ----------- | ---------------- | ------------------------------------ |
| Default LAN | `192.168.1.0/24` | Current main network for all devices |

Current default gateway:

```text
192.168.1.1
```

Current DNS server:

```text
192.168.1.120
```

The DNS server is the Raspberry Pi running Pi-hole and Unbound.

## Goals

The goals for VLAN segmentation are:

* Separate trusted personal devices from guest devices.
* Separate IoT and consumer electronics from the main LAN.
* Create a lab network for Docker, SDN, virtual machines, and experiments.
* Keep infrastructure services protected.
* Allow necessary services such as DNS while blocking unnecessary cross-network access.
* Document firewall rules and validation tests.
* Build a more realistic network design similar to small business or enterprise segmentation.

## Planned VLANs

| Network         | VLAN ID | Subnet            | Purpose                              |
| --------------- | ------: | ----------------- | ------------------------------------ |
| Main LAN        |       1 | `192.168.1.0/24`  | Trusted devices and core services    |
| IoT VLAN        |      20 | `192.168.20.0/24` | Smart TVs, consumer devices, sensors |
| Guest VLAN      |      30 | `192.168.30.0/24` | Guest Wi-Fi, internet-only access    |
| Lab VLAN        |      40 | `192.168.40.0/24` | Docker, VMs, SDN labs, testing       |
| Management VLAN |      99 | `192.168.99.0/24` | Future infrastructure management     |

## Planned Network Roles

### Main LAN

The Main LAN is for trusted devices and core services.

Example devices:

* Desktop computer
* Laptop
* Phone
* Raspberry Pi / Pi-hole
* Homelab mini PC
* WD My Cloud
* Administrative access to UniFi, Portainer, Uptime Kuma, and Pi-hole

Planned subnet:

```text
192.168.1.0/24
```

### IoT VLAN

The IoT VLAN is for smart devices and consumer electronics.

Example devices:

* Smart TV
* Streaming devices
* Smart plugs
* Sensors
* Other consumer devices

Planned subnet:

```text
192.168.20.0/24
```

Primary rule goal:

```text
IoT devices can access the internet, but cannot freely access the Main LAN.
```

### Guest VLAN

The Guest VLAN is for temporary guest devices.

Example devices:

* Visitor phones
* Visitor laptops
* Guest Wi-Fi devices

Planned subnet:

```text
192.168.30.0/24
```

Primary rule goal:

```text
Guest devices can access the internet, but cannot access internal LAN devices.
```

### Lab VLAN

The Lab VLAN is for testing and learning.

Example uses:

* Docker experiments
* Virtual machines
* SDN labs
* Open vSwitch
* Mininet
* Test servers
* Firewall experiments

Planned subnet:

```text
192.168.40.0/24
```

Primary rule goal:

```text
Lab devices can be isolated from the Main LAN unless specific access is allowed.
```

### Management VLAN

The Management VLAN is a future improvement.

Potential devices:

* UniFi Gateway
* UniFi Switch
* UniFi Access Point
* Pi-hole
* Monitoring services

Planned subnet:

```text
192.168.99.0/24
```

This VLAN should be created later, after the simpler VLANs are working. Moving infrastructure management too early can cause lockout or access problems if firewall rules are incorrect.

## Planned DNS Design

Pi-hole will remain the main DNS server.

Current Pi-hole IP:

```text
192.168.1.120
```

Each VLAN should be allowed to reach Pi-hole for DNS.

Required DNS access:

| Source Network | Destination     | Port | Purpose |
| -------------- | --------------- | ---: | ------- |
| Main LAN       | `192.168.1.120` |   53 | DNS     |
| IoT VLAN       | `192.168.1.120` |   53 | DNS     |
| Guest VLAN     | `192.168.1.120` |   53 | DNS     |
| Lab VLAN       | `192.168.1.120` |   53 | DNS     |

DNS should be allowed over both TCP and UDP port `53`.

## Planned Firewall Policy

The general firewall approach will be:

```text
Allow required services first.
Block unnecessary inter-VLAN traffic second.
Allow internet access where appropriate.
```

## Planned Firewall Rules

### Guest VLAN Rules

| Rule                    | Action | Source     | Destination             | Purpose                           |
| ----------------------- | ------ | ---------- | ----------------------- | --------------------------------- |
| Allow Guest DNS         | Allow  | Guest VLAN | Pi-hole `192.168.1.120` | Allow DNS resolution              |
| Block Guest to Main LAN | Block  | Guest VLAN | `192.168.1.0/24`        | Prevent access to trusted devices |
| Allow Guest Internet    | Allow  | Guest VLAN | Internet                | Allow normal internet access      |

### IoT VLAN Rules

| Rule                  | Action | Source   | Destination             | Purpose                                            |
| --------------------- | ------ | -------- | ----------------------- | -------------------------------------------------- |
| Allow IoT DNS         | Allow  | IoT VLAN | Pi-hole `192.168.1.120` | Allow DNS resolution                               |
| Block IoT to Main LAN | Block  | IoT VLAN | `192.168.1.0/24`        | Prevent IoT devices from accessing trusted devices |
| Allow IoT Internet    | Allow  | IoT VLAN | Internet                | Allow normal device internet access                |

### Lab VLAN Rules

| Rule                  | Action | Source   | Destination             | Purpose                                   |
| --------------------- | ------ | -------- | ----------------------- | ----------------------------------------- |
| Allow Lab DNS         | Allow  | Lab VLAN | Pi-hole `192.168.1.120` | Allow DNS resolution                      |
| Block Lab to Main LAN | Block  | Lab VLAN | `192.168.1.0/24`        | Keep experiments isolated                 |
| Allow Main LAN to Lab | Allow  | Main LAN | Lab VLAN                | Allow administration from trusted devices |
| Allow Lab Internet    | Allow  | Lab VLAN | Internet                | Allow package updates and testing         |

## Planned SSIDs

The wireless networks may be mapped to VLANs.

| SSID        |     VLAN | Purpose                                |
| ----------- | -------: | -------------------------------------- |
| Main Wi-Fi  | Main LAN | Trusted devices                        |
| Guest Wi-Fi |       30 | Guest access                           |
| IoT Wi-Fi   |       20 | Smart devices and consumer electronics |

The Guest Wi-Fi network should use guest isolation where appropriate.

## Planned Validation Tests

After VLANs are created, each network should be tested.

### Guest VLAN Tests

Expected results:

| Test                                                     | Expected Result |
| -------------------------------------------------------- | --------------- |
| Guest device receives `192.168.30.x` IP                  | Pass            |
| Guest device can browse internet                         | Pass            |
| Guest device can resolve DNS through Pi-hole             | Pass            |
| Guest device cannot access `192.168.1.1` admin interface | Pass            |
| Guest device cannot access Pi-hole web dashboard         | Pass            |
| Guest device cannot access Portainer                     | Pass            |
| Guest device cannot access Uptime Kuma                   | Pass            |

### IoT VLAN Tests

Expected results:

| Test                                       | Expected Result |
| ------------------------------------------ | --------------- |
| IoT device receives `192.168.20.x` IP      | Pass            |
| IoT device can access internet             | Pass            |
| IoT device can resolve DNS through Pi-hole | Pass            |
| IoT device cannot access trusted desktop   | Pass            |
| IoT device cannot access Portainer         | Pass            |
| IoT device cannot access Uptime Kuma       | Pass            |

### Lab VLAN Tests

Expected results:

| Test                                             | Expected Result |
| ------------------------------------------------ | --------------- |
| Lab device receives `192.168.40.x` IP            | Pass            |
| Lab device can access internet                   | Pass            |
| Lab device can resolve DNS through Pi-hole       | Pass            |
| Main LAN admin device can access lab services    | Pass            |
| Lab device cannot freely access Main LAN devices | Pass            |

## Example Validation Commands

Check IP address:

```powershell
ipconfig
```

Check DNS server:

```powershell
ipconfig /all
```

Check DNS resolution:

```powershell
nslookup google.com
```

Check Pi-hole blocking:

```powershell
nslookup doubleclick.net
```

Check access to internal services:

```powershell
ping 192.168.1.120
ping 192.168.1.216
```

Test web access:

```text
http://192.168.1.120/admin
http://192.168.1.216:3001
https://192.168.1.216:9443
```

## Risks and Notes

VLAN and firewall changes can break access if rules are applied incorrectly.

Important precautions:

* Do not move the UniFi gateway, switch, or access point to a management VLAN until the simpler VLANs are working.
* Start with the Guest VLAN first.
* Test each rule from a real client device.
* Keep a trusted wired device on the Main LAN during configuration.
* Avoid exposing internal services directly to the internet.
* Use VPN for remote administration instead of port forwarding.

## Implementation Plan

Planned order:

1. Create Guest VLAN.
2. Create Guest Wi-Fi network.
3. Test internet access from Guest VLAN.
4. Test that Guest VLAN cannot access Main LAN services.
5. Create IoT VLAN.
6. Move one test IoT device to the IoT VLAN.
7. Test IoT restrictions.
8. Create Lab VLAN.
9. Move test systems or VMs into Lab VLAN.
10. Add firewall rules and document results.
11. Update this document with screenshots and validation results.

## Future Improvements

Future improvements may include:

* Dedicated Management VLAN.
* More detailed firewall rule documentation.
* VLAN diagrams.
* Screenshots of UniFi network and firewall rule configuration.
* VPN access into trusted network only.
* Separate DNS policies per VLAN.
* Monitoring VLAN reachability in Uptime Kuma.
* SDN lab integration with the Lab VLAN.
