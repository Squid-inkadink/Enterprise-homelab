# Network Documentation

This directory contains documentation for the physical and logical network architecture of the Enterprise Homelab.

The goal is to document not only how the network is configured, but why individual design decisions were made and how traffic moves through the environment.

## Planned Documentation

As the network develops, this directory will include documentation covering:

- Physical network topology
- Logical network topology
- IP addressing and subnet allocation
- VLAN design and segmentation
- 802.1Q trunking
- Managed switch configuration
- OPNsense interface configuration
- Routing and inter-VLAN communication
- Firewall rules and traffic policy
- DHCP and DNS
- Wireless network architecture
- Network troubleshooting and validation

## Current Network Components

| Component | Role | Status |
| --- | --- | --- |
| Sophos SG 115 Rev. 3 | OPNsense firewall and Layer 3 gateway | OPNsense Installed |
| Managed Switch | Layer 2 switching and VLAN connectivity | Configuration Pending |
| Wireless Access Point | Wireless management connectivity | Configuration Pending |
| Proxmox Host | Virtualized infrastructure | Network Integration Pending |
| Linux Micro PC | NOC/SOC workstation | Network Integration Pending |

## Design Approach

Network segmentation will be based on the purpose, trust level, and communication requirements of systems rather than creating VLANs simply for the sake of segmentation.

For each network or VLAN, I intend to document:

1. **Purpose** - Why does this network exist?
2. **Assets** - What systems belong to it?
3. **Addressing** - What subnet and gateway does it use?
4. **Connectivity** - What other networks must it communicate with?
5. **Security** - What traffic should be permitted or denied?
6. **Validation** - How was the configuration tested?

The initial network design will be developed as the firewall, managed switch, Proxmox host, NOC/SOC workstation, and wireless management network are integrated.

## Related Documentation

- [Build Log 001: Sophos SG 115 Rev. 3 to OPNsense](../build-log/2026-09-01-opnsense-installation.md)

---

> **Status:** Initial network architecture pending.
