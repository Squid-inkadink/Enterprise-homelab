# Build Log 002: Sophos SG 115 Rev. 3 to OPNsense 

Date: September 1, 2026 | Status: Installation Complete, Configuration Pending

## Objective

The primary goal for this session was relatively small:

1. Restore the OPNsense firewall from the previous session
2. Verify upstream Internet connectivity
3. Update OPNsense
4. Prepare the firewall for VLAN and managed switch configuration

The session ultimately became an exercise in physical interface mapping, subnet troubleshooting, and validating network connectivity layer by layer.

---

## Starting Configuration

The Sophos SG 115 Rev. 3 had already been converted to OPNsense 26.7 during the previous session.

After booting the firewall, the console initially showed:

```text
LAN: 192.168.1.1/24
WAN: No IPv4 address
```

Because OPNsense could not successfully perform a firmware update, the first assumption was that the firewall did not have functional WAN connectivity. To eliminate possible configuration drift from the previous session, the firewall was restored to factory defaults. Immediately after the reset, the default root password was changed before continuing configuration.

---

## Initial Problem

The firmware update failed because the firewall could not reach the Internet over IPv4. The initial symptom appeared to be a DHCP problem because the WAN interface did not have an IPv4 address. Rather than immediately modifying DHCP settings, interface status was checked from the OPNsense shell.

Example:

```shell
ifconfig igb1
```

The interface returned:

```text
status: no carrier
```

The default LAN interface also returned `no carrier`.

This indicated that the problem existed below DHCP and required verifying the physical Ethernet connections and interface assignments first.

---

## Physical Interface Mapping

OPNsense detected four Intel Ethernet interfaces:

```text
igb0
igb1
igb2
igb3
```

However, the factory-default OPNsense assignments did not correspond to the physical Sophos Ethernet ports in the expected order. A powered managed switch was used as a known-good Layer 1 connection. A single Ethernet cable was connected between the managed switch and each Sophos Ethernet port individually. `ifconfig` was then used to determine which logical interface changed to:

```text
status: active
```

This produced the following physical-to-logical mapping:

| Sophos Physical Port | OPNsense / FreeBSD Interface |
|---|---|
| Eth0 | `igb1` |
| Eth1 | `igb2` |
| Eth2 | `igb3` |
| Eth3 | `igb0` |

This confirmed that relying solely on the default `igb` numbering would have resulted in incorrect WAN and LAN assignments.

---

## Subnet Overlap

During troubleshooting, another issue became apparent.

The default OPNsense LAN network was:

```text
192.168.1.0/24
```

The upstream ISP router was also using the same private IPv4 network. This resulted in overlapping Layer 3 networks on opposite sides of the firewall.

Initial testing to:

```text
1.1.1.1
```

returned repeated:

```text
Time to live exceeded
```

responses involving `192.168.1.1`.

To eliminate the overlap, the OPNsense LAN was changed to:

```text
192.168.10.1/24
```

After changing the LAN subnet, the previous TTL loop disappeared. The firewall instead returned:

```text
No route to host
```

This was expected because the WAN interface still had not been correctly assigned.

---

## Correcting WAN and LAN Assignments

Using the manually verified interface mapping, the firewall interfaces were reassigned.

The intended physical layout was:

```text
ISP Router
    |
    |
Sophos Eth1
    |
   igb2
    |
   WAN
```

and:

```text
Sophos Eth0
    |
   igb1
    |
   LAN
    |
Managed Switch
```

The final assignments were therefore:

```text
WAN = igb2
LAN = igb1
```

LAGG, VLAN, and optional interface configuration were intentionally skipped during this phase.

The purpose was to establish basic network connectivity before introducing additional network abstractions.

---

## Connectivity Validation

After correcting the interface assignments and removing the subnet overlap, Internet connectivity was tested again.

A ping to Cloudflare DNS was successful:

```text
PING 1.1.1.1

3 packets transmitted
3 packets received
0.0% packet loss
```

This confirmed that the firewall now had:

- Physical Ethernet connectivity
- Correct WAN and LAN interface assignments
- Functional IPv4 routing
- A valid upstream path
- Internet connectivity

The firmware update was then retried using the OPNsense console:

```text
12) Update from console
```

The update completed successfully.

---

## Troubleshooting Flow

The troubleshooting process can be summarized as:

```text
Firmware update failed
        |
        v
Check Internet connectivity
        |
        v
WAN had no IPv4 connectivity
        |
        v
Check interface state
        |
        v
Interfaces reported "no carrier"
        |
        v
Validate physical Ethernet mappings
        |
        v
Discover Sophos port / igb mismatch
        |
        v
Identify overlapping IPv4 networks
        |
        v
Move LAN to 192.168.10.0/24
        |
        v
Correct WAN / LAN assignments
        |
        v
Validate Internet connectivity
        |
        v
Firmware update succeeds
```

---

## Key Takeaways

### Validate the Layer Below the Failure

The original symptom was an OPNsense firmware update failure.

The updater itself was not the problem.

The failure was caused by underlying network configuration issues. Troubleshooting moved downward from application functionality to routing, addressing, interface state, and finally physical Ethernet mapping.

This reinforced the importance of validating lower-layer dependencies before troubleshooting higher-layer services.

### Logical Interface Names Do Not Always Match Physical Labels

The Sophos chassis Ethernet numbering did not directly correspond to the FreeBSD `igb` interface numbering.

Physical ports were verified rather than assumed.

This is particularly important when repurposing proprietary network appliances with a different operating system.

### An IP Address Does Not Prove Physical Connectivity

An interface may have a configured IPv4 address while still reporting:

```text
status: no carrier
```

The configured IP exists at Layer 3, while carrier state reflects the physical Ethernet connection at Layer 1.

### Avoid Overlapping Routed Networks

Both sides of a router or firewall should not normally use the same IPv4 subnet.

The initial configuration effectively created:

```text
Upstream Network: 192.168.1.0/24
OPNsense LAN:     192.168.1.0/24
```

Moving the internal LAN to a different subnet eliminated the routing conflict.

### Troubleshooting Results Can Be Progress

The sequence of errors changed during troubleshooting:

```text
Firmware update failure
        ↓
TTL exceeded
        ↓
No route to host
        ↓
Successful ping
```

Each change provided additional information about the state of the network and helped isolate the next problem.

---

## Current State

At the end of Day 2:

- OPNsense 26.7 is installed on the Sophos SG 115 Rev. 3
- Root credentials have been changed from defaults
- Physical Ethernet ports have been mapped to FreeBSD interfaces
- WAN and LAN interfaces are correctly assigned
- LAN addressing has been moved away from the upstream ISP subnet
- IPv4 Internet connectivity has been verified
- OPNsense updates successfully
- Managed switch and VLAN configuration remain intentionally deferred

---

## Next Steps

The next phase will focus on network segmentation and management connectivity.

Planned tasks include:

1. Rebuild the current configuration from factory defaults to validate repeatability
2. Create the initial VLAN interfaces
3. Configure the OPNsense-to-switch 802.1Q trunk
4. Configure initial access ports on the managed switch
5. Connect the MicroPC to the Security VLAN
6. Reinstall and configure Proxmox
7. Place Proxmox management on the Management VLAN
8. Validate inter-VLAN routing
9. Configure limited firewall access between Security and Management
10. Verify access to both the OPNsense and Proxmox web interfaces from the MicroPC

This approach is intended to show not only what was built, but how technical understanding developed throughout the project.
