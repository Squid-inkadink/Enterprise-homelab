# Systems Documentation

This directory contains documentation for the physical systems, operating systems, virtualization infrastructure, and services deployed within the Enterprise Homelab.

The goal is to develop practical systems administration experience while documenting how each system is installed, configured, maintained, secured, monitored, and integrated into the larger environment.

## Planned Infrastructure

The initial systems environment will include:

| System | Platform | Planned Purpose | Status |
| --- | --- | --- | --- |
| Dell Precision 5820 | Proxmox VE | Primary virtualization and compute host | Integration Pending |
| Micro PC | Linux | NOC/SOC monitoring workstation | Distribution TBD |
| MacBook Air | Asahi Linux | Remote management and SSH client | Integration Pending |

Additional virtual machines and services will be documented as requirements for them develop.

## Planned Documentation

As systems are deployed, this directory may include documentation covering:

- Hardware configuration
- Operating system installation
- Proxmox configuration
- Virtual machine deployment
- Linux administration
- Windows administration
- Storage configuration
- User and permission management
- SSH and remote administration
- Patch and update management
- System hardening
- Logging and monitoring
- Backup and recovery
- Service configuration
- Troubleshooting

## System Documentation Approach

Each major system should eventually answer several basic questions:

1. **Purpose** - Why does this system exist?
2. **Hardware/Resources** - What resources are assigned to it?
3. **Operating System** - What is installed and why?
4. **Network Placement** - Where does it live within the network?
5. **Services** - What applications or infrastructure does it provide?
6. **Access** - How is the system administered?
7. **Security** - How is access restricted and the system hardened?
8. **Monitoring** - How is system health and activity observed?
9. **Recovery** - How can the system or service be restored?
10. **Validation** - How do I know the system is functioning as intended?

Not every system will require every category. Documentation will develop alongside the environment rather than attempting to define every system before it exists.

## Proxmox

The Dell Precision 5820 will serve as the primary virtualization platform for the environment.

Proxmox will provide the compute resources for virtual machines and services while OPNsense and the managed switch provide the primary network and security infrastructure.

Detailed Proxmox architecture will be documented after the initial VLAN and network design is established.

## NOC/SOC Workstation

The Linux micro PC is planned as a dedicated interface for observing and administering the environment.

Its Linux distribution and final software stack have not yet been selected.

Potential responsibilities include:

- Infrastructure monitoring
- Network visibility
- Security dashboards
- Centralized logging
- Administrative tools
- Remote access to infrastructure

The system will initially be integrated into the network before monitoring and security applications are selected.

---

> **Status:** Initial systems architecture pending.
