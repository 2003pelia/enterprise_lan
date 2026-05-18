# Enterprise LAN with Firewall Segmentation & HA

Virtualized enterprise LAN built on Proxmox with pfSense firewall segmentation across User, Server, Management, and DMZ zones.

## Overview

This is my capstone project: a fully virtualized simulation of an enterprise Local Area Network that mirrors the architecture, security controls, and redundancy you'd find in a real corporate environment — built entirely in VMs, no physical networking gear required.

The driving question:

> **How can network segmentation and firewall policy enforcement reduce the attack surface of an enterprise LAN without breaking legitimate user access?**

To answer it, the network is split into four isolated zones with a pfSense firewall enforcing strict inter-zone traffic policies. The design is then validated through connectivity testing, security scanning, and failover testing to show that segmentation meaningfully limits lateral movement after a breach.

## Architecture

Four internal zones sit behind a single pfSense firewall, each on its own subnet and VLAN-equivalent virtual network:

```
                     ┌───────────────┐
                     │   Internet    │
                     └───────┬───────┘
                             │
                       ┌─────▼─────┐
                       │  pfSense  │
                       │ (Firewall)│
                       └─┬─┬─┬─┬───┘
              ┌──────────┘ │ │ └──────────┐
              │            │ │            │
         ┌────▼───┐  ┌─────▼─┐ ┌─▼─────┐ ┌▼──────┐
         │  USER  │  │SERVER │ │ MGMT  │ │  DMZ  │
         │ VLAN10 │  │VLAN20 │ │VLAN30 │ │VLAN40 │
         └────────┘  └───────┘ └───────┘ └───────┘
```

| Zone        | Subnet            | Purpose                                  |
| ----------- | ----------------- | ---------------------------------------- |
| User        | 192.168.10.0/24   | End-user workstations                    |
| Server      | 192.168.20.0/24   | Internal services (DNS, DHCP, etc.)      |
| Management  | 192.168.30.0/24   | Admin access to all infrastructure       |
| DMZ         | 192.168.40.0/24   | Public-facing services, isolated         |

## Tech Stack

- **Hypervisor:** Proxmox VE 8.4 (ZFS RAID storage, no-subscription repo)
- **Firewall:** pfSense 2.8.1
- **DNS:** BIND9 on Ubuntu Server 24.04
- **DHCP:** ISC DHCP Server on Ubuntu Server 24.04
- **Test Workstation:** Ubuntu Desktop
- **High Availability:** CARP failover with a secondary pfSense instance

## Firewall Policy

Rules follow a least-privilege model — each zone only gets the access it actually needs:

- **User → Server:** allowed on TCP 22, 80, 443 only
- **User → Management / DMZ:** blocked
- **Server → Internet:** outbound DNS and HTTP/HTTPS for updates only
- **Server → Management:** blocked
- **Management → All zones:** full access (admin)
- **DMZ → Internal zones:** blocked
- **Internet → DMZ:** inbound web traffic allowed

The idea: if any one zone is compromised, the blast radius is contained.

## Skills Demonstrated

- Network architecture and subnet design
- Firewall rule writing and policy enforcement
- VLAN-style segmentation
- DNS and DHCP infrastructure
- High availability design with CARP
- Hypervisor administration (Proxmox, KVM/QEMU)
- Security testing with `ping`, `nc`, and `nmap`

## Status

Project rebuilt on Proxmox after stability issues with virt-manager. Core infrastructure (hypervisor, virtual networks, pfSense, firewall rules, test workstation) is in place. Remaining work: finish DNS/DHCP on Ubuntu Server, deploy DMZ web server, complete CARP HA failover, and run full validation tests.

## Author

**Preston Elia** — Built as part of my Networking capstone.
