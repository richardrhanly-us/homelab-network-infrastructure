# Maintenance Log

This document tracks major build steps, configuration changes, troubleshooting notes, and validation results for the homelab network infrastructure project.

## 2026-06-03

### Rack and Physical Network

* Assembled the 12U network rack.
* Mounted the patch panel, UniFi switch, UniFi gateway, Raspberry Pi rack mount, HP mini PC, and rack PDU.
* Organized the rack layout with the patch panel above the switch for shorter patch cable runs.
* Routed Ethernet and power cabling through the rack.
* Confirmed the rack layout leaves space for airflow and future expansion.

### UniFi Network

* Routed the home network through the UniFi Gateway.
* Connected the UniFi USW Lite 8 PoE switch.
* Connected the UniFi U6+ access point through the rack and patch panel.
* Confirmed client devices are receiving DHCP from the UniFi Gateway.
* Confirmed the default gateway is `192.168.1.1`.

### Homelab Mini PC

* Connected the HP mini PC to the UniFi network.
* Confirmed the mini PC received a DHCP address from the UniFi Gateway.
* Identified the mini PC at `192.168.1.216`.
* Confirmed SSH access to the mini PC from the local network.
* Confirmed Docker is running on the mini PC.
* Confirmed Portainer and Uptime Kuma containers are running.

### Pi-hole and Unbound

* Configured Raspberry Pi as the Pi-hole DNS server.
* Set Pi-hole IP address to `192.168.1.120`.
* Confirmed Pi-hole DHCP is disabled.
* Confirmed UniFi Gateway remains the only DHCP server.
* Configured UniFi DHCP to hand out Pi-hole as the DNS server.
* Confirmed Unbound is working at `127.0.0.1#5335`.
* Confirmed Pi-hole is using Unbound as its upstream resolver.

### DNS Validation

* Confirmed Windows desktop received DNS server `192.168.1.120`.
* Confirmed `nslookup google.com` used `pi.hole`.
* Confirmed `doubleclick.net` returned `0.0.0.0` and `::`.
* Confirmed Pi-hole query log shows client DNS activity.

### Monitoring

* Added Pi-hole ping monitor in Uptime Kuma.
* Added Pi-hole web dashboard monitor in Uptime Kuma.
* Added Portainer monitor in Uptime Kuma.
* Confirmed core services appear online in Uptime Kuma.

### Documentation

* Created GitHub repository for the homelab network infrastructure project.
* Added main `README.md`.
* Added rack build documentation.
* Added Pi-hole and Unbound documentation.
* Added Docker monitoring documentation.
* Added VLAN segmentation planning documentation.
* Added sanitized screenshots for public portfolio use.

## Future Log Items

Use this section to track future changes.

### Planned

* Add UniFi switch port screenshot.
* Add Pi-hole dashboard screenshot.
* Add Pi-hole query log screenshot.
* Add Uptime Kuma dashboard screenshot.
* Add Portainer dashboard screenshot.
* Create Guest VLAN.
* Create IoT VLAN.
* Create Lab VLAN.
* Add VLAN firewall rules.
* Test VLAN isolation.
* Add VPN remote access.
* Add secondary Pi-hole for DNS redundancy.
* Add rack temperature monitoring.
* Add UPS backup.
