# Build Log 001: Sophos SG 115 Rev. 3 to OPNsense

**Date:** September 1, 2026  
**Status:** Installation Complete, Configuration Pending

## Overview

The first major infrastructure component of my enterprise homelab is the firewall.

For this project, I am repurposing a **Sophos SG 115 Rev. 3** security appliance and replacing the existing Sophos UTM installation with **OPNsense 26.7**.

The goal is not simply to get a firewall running. I want to use the appliance to learn and document concepts that translate into enterprise networking and security, including:

- Firewall administration
- Layer 3 routing
- Network segmentation
- VLANs and 802.1Q trunking
- Inter-VLAN traffic control
- NAT
- DHCP and DNS
- Firewall policy development
- Logging and monitoring
- Network troubleshooting

This document records the initial hardware validation, installation of OPNsense, and some of the networking concepts I encountered along the way.

---

## Why a Sophos Appliance?

The Sophos SG 115 is an older purpose-built firewall appliance that can be repurposed as a general x86-64 firewall.

Rather than purchasing new hardware, this gives me a relatively inexpensive platform with multiple physical Ethernet interfaces that was originally designed to operate as a firewall.

The appliance provides four Ethernet interfaces labeled by Sophos as:

| Interface | Original Sophos Role |
| --- | --- |
| eth0 | LAN |
| eth1 | WAN |
| eth2 | DMZ |
| eth3 | HA |

These labels describe how Sophos originally intended the interfaces to be used. They do not permanently define what the ports must do.

Once OPNsense is installed, interfaces can be reassigned based on the needs of the network.

For my initial configuration, I plan to maintain the intuitive physical layout:

| Physical Port | Planned Role |
| --- | --- |
| eth0 | LAN / internal network |
| eth1 | WAN / upstream network |
| eth2 | Unassigned / future use |
| eth3 | Unassigned / future use |

As the lab develops, the LAN side will eventually connect to a managed switch using VLANs to segment different portions of the environment.

---

## Why OPNsense?

There are many firewall platforms I could use for this lab, including commercial products from Sophos, Palo Alto Networks, Cisco, Juniper, and others.

Those platforms are valuable to learn, particularly because many are widely deployed in enterprise environments. However, licensing, hardware requirements, feature restrictions, and access to updates can make commercial platforms less practical for a continuously evolving personal lab.

For this project, I wanted a firewall platform that:

- Can run on standard x86-64 hardware
- Does not require enterprise licensing for basic lab functionality
- Supports VLANs and inter-VLAN routing
- Provides stateful firewalling and NAT
- Supports VPN technologies
- Provides useful logging and monitoring
- Can grow alongside the rest of the homelab
- Allows me to experiment without worrying about consuming licensed features

OPNsense provides those capabilities while remaining open source and actively maintained.

### Why OPNsense Instead of pfSense?

Both OPNsense and pfSense are FreeBSD-based firewall platforms, and either would be capable of supporting this homelab.

I selected OPNsense primarily because I wanted an open-source platform with a modern management interface, frequent updates, and enough functionality to support increasingly complex network designs as the lab grows.

The purpose of the lab is not to argue that one platform is universally better than another. OPNsense simply fits the requirements of this project.

---

## Initial Hardware Validation

My first snag occurred almost immediately.

I initially connected the Sophos appliance using its first power input. The appliance powered on and the keyboard RGB illuminated, but I received no monitor output.

Observed indicators included:

- Solid red Status LED
- Intermittently blinking blue LED
- Green Ethernet activity on eth0
- Solid link indicator on eth0
- No video output

The Ethernet link indicators suggested that at least part of the appliance was receiving power and the NIC had established a physical link.

Rather than immediately assuming that the appliance itself was dead, I moved the power connection to the second power input/path.

The system subsequently produced video output and successfully booted to the existing **Sophos UTM 9.6** splash screen.

This isolated the original problem to the first power path, although additional testing would be required to determine whether the actual failure is the power adapter, connector, or another component in that path.

### Troubleshooting Takeaway

One of the simplest lessons from this was also one of the most important:

> **Change one variable at a time.**

The appliance showed some signs of life, so replacing or disassembling hardware would have been premature. Testing the alternate power path provided another known state and confirmed that the core appliance could POST and boot.

---

## Creating the OPNsense Installer

I did not have an OPNsense installer prepared beforehand, so the next step was creating a bootable USB flash drive.

The basic process is:

```text
OPNsense image
      |
      v
USB flash drive
      |
      v
Sophos SG 115
      |
      v
Boot from USB
      |
      v
Install OPNsense to internal storage
```

For this installation I used:

- **OPNsense version:** 26.7
- **Architecture:** amd64
- **Image:** VGA
- **Installation workstation:** macOS

The VGA installer was appropriate because the Sophos appliance provides local video output and I was performing the installation with a directly connected monitor and keyboard.

---

## Creating the USB Installer on macOS

### 1. Identify the USB Device

After connecting the USB flash drive, I ran:

```bash
diskutil list
```

macOS identified my removable drive as:

```text
/dev/disk4
```

> **Warning:** This identifier is specific to my system and should not be copied blindly.

Writing an image to the wrong disk with `dd` can destroy data on another drive.

As an additional safeguard, I verified the device:

```bash
diskutil info /dev/disk4
```

I checked the reported device information to ensure it matched the USB flash drive before continuing.

---

### 2. Decompress the OPNsense Image

The downloaded OPNsense installer may be distributed as a compressed `.bz2` file.

After navigating to the directory containing the download:

```bash
cd ~/Downloads
```

the image can be decompressed with:

```bash
bunzip2 OPNsense-*.img.bz2
```

This produces the `.img` file that will be written to the USB drive.

For this installation, my resulting image was:

```text
OPNsense-26.7-vga-amd64.img
```

---

### 3. Unmount the USB

Before writing the image, I unmounted the volumes on the USB drive:

```bash
diskutil unmountDisk /dev/disk4
```

macOS returned:

```text
Unmount of all volumes on disk4 was successful
```

The device remains connected, but its filesystems are no longer mounted by macOS.

---

### 4. Write the Image with `dd`

The basic `dd` command uses several important operands:

- `if` = **input file**, the OPNsense image
- `of` = **output file/device**, the USB device
- `bs` = **block size**, controlling the size of the blocks used for the operation

I initially attempted:

```bash
sudo dd if=OPNsense-*.img of=/dev/rdisk4 bs=4m
```

and received:

```text
no matches found: if=OPNsense-*.img
```

The zsh shell attempted to expand the `*` wildcard before executing `dd`. Because it could not find a matching file from my current working directory, zsh rejected the command before `dd` ran.

Instead of relying on the wildcard, I specified the complete filename:

```bash
sudo dd if=OPNsense-26.7-vga-amd64.img of=/dev/rdisk4 bs=4m
```

This successfully wrote the image.

### `/dev/disk4` Versus `/dev/rdisk4`

macOS exposes both a regular disk device and a corresponding raw disk device.

In this case:

```text
/dev/disk4
/dev/rdisk4
```

refer to the same physical USB storage device through different device interfaces.

The raw device was used for `dd` because it provides more direct access and is generally faster for sequential image-writing operations.

After the operation completed, I synchronized pending writes and ejected the device:

```bash
sync
diskutil eject /dev/disk4
```

The OPNsense installation USB was now ready.

---

## Booting the Sophos from USB

After booting from the newly created USB installer, OPNsense presented:

```text
Please login as 'root' to continue in live mode,
or as 'installer' to start the installation.
```

To start installation:

```text
login: installer
password: opnsense
```

I selected the default keyboard layout.

The installer then provided several options:

```text
Install ZFS
Install UFS
Other Modes
Import Config
Password Reset
Force Reboot
Force Halt
```

For this appliance I selected:

```text
Install UFS
```

---

## Why UFS?

ZFS provides valuable storage features including checksumming, snapshots, pools, and more advanced storage management.

Those features can be extremely useful for servers and larger storage systems.

However, this appliance is being used as a dedicated firewall with a single internal storage device. My priority is a simple and lightweight firewall installation rather than advanced storage management.

For that reason, I selected **UFS**.

This is also an important homelab design principle:

> **More features do not automatically mean a better architecture.**

Technology should be selected based on the requirements of the system.

---

## Selecting the Installation Disk

The installer detected two storage devices:

```text
da0  - ASolid USB
ada0 - AData ...
```

The devices represented:

```text
da0
└── OPNsense USB installer

ada0
└── Internal ADATA storage in the Sophos appliance
```

The target for the OPNsense installation was therefore:

```text
ada0
```

Selecting this disk overwrites the existing Sophos UTM installation.

The USB installer should **not** be selected as the installation target.

---

## WAN vs. LAN: A Basic Concept Worth Documenting

While waiting for OPNsense to install, I realized I had initially been thinking about WAN and LAN somewhat backwards.

It is easy to assume that an "enterprise" network somehow changes what these terms mean.

It does not.

A useful way to think about them from the firewall's perspective is:

```text
WAN = Outside / Upstream
LAN = Inside / Local
```

In this homelab, the WAN interface faces toward my upstream network/ISP, while the LAN interface faces toward networks managed by OPNsense.

At its simplest:

```text
                     Internet
                         |
                         v
                  Upstream / ISP
                         |
                         v
                 +---------------+
                 |    OPNsense   |
                 |               |
           WAN --|               |-- LAN
                 +---------------+
                         |
                         v
                  Managed Switch
                         |
                         v
              Internal Homelab Networks
```

The WAN interface represents the upstream side of the firewall.

The LAN interface represents the networks being managed behind the firewall.

An enterprise network does not reverse this relationship. Instead, the **LAN side becomes substantially more complex**.

Eventually, my environment may look more like:

```text
                         Internet
                            |
                            v
                         OPNsense
                            |
                       802.1Q Trunk
                            |
                            v
                      Managed Switch
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
        Management        Servers        Clients
           VLAN             VLAN           VLAN
             |              |              |
          Proxmox       Server VMs      Endpoints
          Switch        SIEM/etc.
          Management
```

OPNsense can then act as a Layer 3 boundary between these networks.

For example, traffic from a client VLAN attempting to reach a server VLAN may need to traverse OPNsense:

```text
Client
   |
   v
VLAN 30
   |
   v
OPNsense
   |
   |  Firewall policy:
   |  Is VLAN 30 allowed to reach
   |  this service on VLAN 20?
   |
   v
VLAN 20
   |
   v
Server
```

This is where VLAN segmentation and firewall policy begin turning a simple home network into a useful enterprise networking lab.

---

## Current Status

### Completed September 1, 2026

- [x] Sophos SG 115 hardware powered on
- [x] Alternate power path successfully tested
- [x] Existing Sophos UTM 9.6 installation verified
- [x] OPNsense 26.7 installer downloaded
- [x] Bootable USB created using macOS
- [x] Sophos successfully booted from OPNsense USB
- [x] UFS selected
- [x] Internal ADATA storage identified
- [x] OPNsense installation completed
- [x] Appliance safely powered down following installation

### Next Steps

Over the next several days:

- [ ] Boot into the installed OPNsense environment
- [ ] Identify OPNsense interface mappings
- [ ] Configure WAN
- [ ] Configure initial LAN
- [ ] Access OPNsense WebGUI
- [ ] Update OPNsense
- [ ] Verify basic network connectivity
- [ ] Document physical network topology
- [ ] Design VLAN architecture
- [ ] Determine VLAN IDs and subnet allocation
- [ ] Configure 802.1Q switch trunk
- [ ] Configure VLAN interfaces
- [ ] Develop inter-VLAN firewall policies
- [ ] Integrate Proxmox
- [ ] Connect the NOC/SOC Linux workstation
- [ ] Connect the wireless access point
- [ ] Establish wireless management access

---

## End of Session

The first session focused primarily on validating and repurposing the firewall hardware rather than configuring the network itself.

The Sophos SG 115 Rev. 3 was successfully validated, the existing Sophos UTM installation was replaced with OPNsense 26.7, and the appliance is ready for initial network configuration.

The next phase will move from installing individual infrastructure components toward designing how those components should communicate.

The primary focus will be:

```text
Physical Topology
       |
       v
Logical Network Design
       |
       v
VLAN / Subnet Architecture
       |
       v
802.1Q Trunking
       |
       v
Firewall Policy
       |
       v
Proxmox + Homelab Systems
```

Rather than adding complexity simply because the technology supports it, each network, VLAN, firewall rule, and system will be added based on a defined purpose and documented as the environment develops.
