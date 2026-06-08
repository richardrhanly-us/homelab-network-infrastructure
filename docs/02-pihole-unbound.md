# Pi-hole and Unbound DNS

## Overview

This document covers the DNS portion of the homelab network.

Pi-hole is used as the network-wide DNS filtering service, and Unbound is used as the recursive DNS resolver. Client devices receive the Pi-hole IP address from the UniFi Gateway through DHCP.

This setup provides centralized DNS filtering, better visibility into network DNS traffic, and a more self-contained DNS resolution path.

## Goals

The goals for this part of the project were:

* Use Pi-hole as the main DNS server for the home network.
* Keep DHCP handled by the UniFi Gateway.
* Disable DHCP inside Pi-hole to avoid DHCP conflicts.
* Use Unbound as the upstream recursive DNS resolver.
* Confirm that client devices are receiving Pi-hole as their DNS server.
* Confirm that Pi-hole is blocking known ad/tracking domains.
* Monitor Pi-hole availability using Uptime Kuma.

## DNS Architecture

```text
Client Device
      |
      v
UniFi Gateway gives DNS through DHCP
      |
      v
Pi-hole: 192.168.1.120
      |
      v
Unbound: 127.0.0.1#5335
      |
      v
Authoritative DNS Servers
```

## Device Roles

| Device         | Role                                |
| -------------- | ----------------------------------- |
| UniFi Gateway  | DHCP server and network gateway     |
| Raspberry Pi   | Pi-hole and Unbound DNS server      |
| Pi-hole        | DNS filtering                       |
| Unbound        | Recursive DNS resolver              |
| Client Devices | Receive Pi-hole as DNS through DHCP |

## Pi-hole Configuration

| Setting      | Value                        |
| ------------ | ---------------------------- |
| Hostname     | `pihole`                     |
| IP Address   | `192.168.1.120`              |
| Role         | DNS filtering                |
| DHCP         | Disabled                     |
| Upstream DNS | `127.0.0.1#5335`             |
| Admin URL    | `http://192.168.1.120/admin` |

Pi-hole is used only for DNS filtering. DHCP remains handled by the UniFi Gateway.

## Unbound Configuration

| Setting          | Value            |
| ---------------- | ---------------- |
| Host             | Raspberry Pi     |
| Address          | `127.0.0.1`      |
| Port             | `5335`           |
| Pi-hole Upstream | `127.0.0.1#5335` |

Unbound acts as the recursive DNS resolver for Pi-hole. Instead of forwarding DNS requests to a public DNS provider, Pi-hole forwards allowed queries to Unbound running locally on the Raspberry Pi.

## UniFi DHCP Configuration

| Setting                      | Value                         |
| ---------------------------- | ----------------------------- |
| DHCP Server                  | UniFi Gateway                 |
| Gateway IP                   | `192.168.1.1`                 |
| DHCP Range                   | `192.168.1.6 - 192.168.1.254` |
| Domain                       | `localdomain`                 |
| DNS Server handed to clients | `192.168.1.120`               |

## Pi-hole Dashboard

![Pi-hole Dashboard](../screenshots/pihole/pihole-dashboard.png)

The Pi-hole dashboard confirms that DNS filtering is active, queries are being processed, domains are being blocked, and blocklists are loaded.

## Pi-hole Query Log

![Pi-hole Query Log](../screenshots/pihole/pihole-query-log.png)

The query log confirms that client devices are sending DNS requests through Pi-hole.

## Validation

### Unbound Validation

Command run on the Raspberry Pi:

```bash
dig google.com @127.0.0.1 -p 5335
```

Confirmed result:

```text
status: NOERROR
SERVER: 127.0.0.1#5335
```

This confirms that Unbound is responding on port `5335`.

### Pi-hole Local DNS Validation

Command run on the Raspberry Pi:

```bash
dig google.com @127.0.0.1
```

Confirmed result:

```text
status: NOERROR
SERVER: 127.0.0.1#53
```

This confirms that Pi-hole is responding on local DNS port `53`.

### Windows Client DNS Validation

Command run from a Windows desktop:

```powershell
nslookup google.com
```

Confirmed result:

```text
Server:  pi.hole
Address: 192.168.1.120
```

This confirms that the Windows client is using Pi-hole as its DNS server.

### Pi-hole Blocking Validation

Command run from a Windows desktop:

```powershell
nslookup doubleclick.net
```

Confirmed result:

```text
Addresses:
::
0.0.0.0
```

This confirms that Pi-hole is blocking the test advertising/tracking domain.

## Windows Client DHCP Result

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

This confirms that the UniFi Gateway is still handling DHCP and that it is handing out Pi-hole as the DNS server.

## Monitoring

Pi-hole is monitored using Uptime Kuma.

Current monitors:

| Monitor               | Type | Target                       |
| --------------------- | ---- | ---------------------------- |
| Pi-hole               | Ping | `192.168.1.120`              |
| Pi-hole Web Dashboard | HTTP | `http://192.168.1.120/admin` |

The ping monitor verifies that the Raspberry Pi is reachable on the network.

The HTTP monitor verifies that the Pi-hole web dashboard is available.

## Best Practices Applied

* Pi-hole has a fixed/reserved IP address.
* UniFi Gateway remains the only DHCP server.
* Pi-hole DHCP is disabled.
* Client DNS is assigned through UniFi DHCP.
* Pi-hole forwards allowed queries to Unbound.
* Unbound runs locally on the Raspberry Pi.
* No public DNS provider is configured as Pi-hole upstream DNS.
* DNS behavior was validated from both the Raspberry Pi and a Windows client.
* Pi-hole is monitored in Uptime Kuma.

## Future Improvements

Planned improvements include:

* Add a secondary Raspberry Pi running a second Pi-hole instance.
* Configure the secondary Pi-hole with its own IP address.
* Use Gravity Sync to keep both Pi-hole instances synchronized.
* Add the secondary Pi-hole as DNS Server 2 in UniFi DHCP.
* Add VLAN firewall rules that allow each VLAN to reach Pi-hole for DNS.
* Add dashboard monitoring for DNS health and query volume.
