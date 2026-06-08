# Rack Build and Physical Network Infrastructure

## Overview

This document covers the physical network rack build for the homelab environment. The goal was to centralize routing, switching, patching, DNS services, Docker services, and power distribution into a structured 12U rack.

## Goals

- Build a clean and expandable 12U rack layout.
- Mount core network equipment in a logical order.
- Place the patch panel directly above the switch for short patch cable runs.
- Keep power and Ethernet cabling separated where possible.
- Leave open rack space for future VLAN, VPN, monitoring, and SDN lab expansion.
- Document the physical layout for troubleshooting and portfolio use.

## Rack Equipment

| Rack Item | Purpose |
|---|---|
| 12U rack | Central rack enclosure |
| AC Infinity rack fan | Airflow and cooling |
| 24-port keystone patch panel | Ethernet termination point |
| UniFi USW Lite 8 PoE | Main managed switch |
| UniFi Cloud Gateway Ultra | Router, firewall, DHCP, gateway |
| Raspberry Pi rack mount | Mount for Pi-hole / Unbound Raspberry Pi |
| HP EliteDesk Mini PC | Ubuntu Docker host |
| Rack PDU | Power distribution |
| Blank rack spaces | Airflow and future expansion |

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
