# Enterprise Homelab

An enterprise-style homelab built to develop hands-on experience with
networking, systems administration, security, monitoring, and automation.

The goal of this project is not simply to deploy services, but to understand
and document the reasoning behind architecture, configuration, troubleshooting,
and security decisions.

> **Status:** Active Build  
> **Started:** September 2026

## Project Goals

- Design and operate a segmented enterprise-style network
- Develop practical experience with VLANs, routing, switching, and firewalling
- Build and administer virtualized infrastructure with Proxmox
- Deploy Windows and Linux systems and services
- Centralize monitoring and security visibility
- Introduce infrastructure automation as the environment matures
- Document troubleshooting, failures, and design decisions

## Current Environment

| Component | Platform | Purpose | Status |
| --- | --- | --- | --- |
| Firewall | Sophos SG 115 Rev. 3 / OPNsense | Routing, firewalling, VLAN gateway | In Progress |
| Switch | Managed Ethernet Switch | Layer 2 switching and VLAN trunking | Planned |
| Hypervisor | Proxmox VE | Virtualization / compute | Planned |
| NOC/SOC Workstation | Linux Micro PC | Monitoring and security dashboards | Planned |
| Management Client | Asahi Linux | Wireless/SSH administration | Planned |
| Wireless AP | TBD | Management wireless access | Planned |

## Architecture

Network architecture is currently being designed.

Initial design work will include:

- Physical topology
- Logical topology
- VLAN and subnet allocation
- 802.1Q trunking
- Management network
- Server/virtualization networks
- Firewall policy
- Monitoring and security infrastructure

## Documentation

### Build Log

- [2026-09-01 - Sophos SG 115 and OPNsense Installation](docs/build-log/2026-09-01-opnsense-install.md)

### Network

Network documentation will be added as the architecture is developed.

### Systems

System documentation will be added as services are deployed.

## Documentation Philosophy

Failures and troubleshooting are intentionally documented.

This repository is intended to show not only the final configuration, but the
process used to design, troubleshoot, and understand the environment.
