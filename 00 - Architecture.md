# Chapter 01 - Architecture & Infrastructure Planning

> **EduTek Labs Enterprise Deployment Manual**
>
> Version: 1.0

## Objective

Design a reliable, repeatable virtualization environment using Proxmox VE that hosts Windows 10, Windows 11, Ubuntu Desktop, and Ubuntu Server virtual machines, all accessible through Apache Guacamole.

---

# High-Level Architecture

```text
                 Internet
                     |
            Cloudflare Tunnel
                     |
             Docker Host / Guacamole
                     |
              -------------------
              |                 |
           Proxmox VE 8.x (vmbr0)
      -----------+-----------------------
      |          |          |           |
 Windows10   Windows11   Ubuntu     Ubuntu
    RDP         RDP      XRDP         SSH
```

## Goals

- Create reusable golden VM templates.
- Validate native connectivity before configuring Guacamole.
- Standardize protocols:
  - Windows 10 → RDP
  - Windows 11 → RDP
  - Ubuntu Desktop → XRDP (RDP)
  - Ubuntu Server → SSH

## Recommended Hardware

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | 8 cores | 16+ cores |
| RAM | 32 GB | 64 GB+ |
| Storage | 1 TB SSD | 2 TB NVMe |
| Network | 1 Gbps | 2.5/10 Gbps |

## Network Plan

Example:

| Device | IP |
|---------|----|
| Proxmox | 192.168.1.10 |
| Guacamole | 192.168.1.20 |
| Windows Template | DHCP while building |
| Ubuntu Desktop | DHCP while building |
| Ubuntu Server | DHCP while building |

Reserve or assign static IPs after installation.

## Deployment Principles

1. Build one OS at a time.
2. Test using the native client:
   - Windows: mstsc
   - Ubuntu Desktop: mstsc/Remmina
   - Ubuntu Server: ssh
3. Configure Guacamole only after native access succeeds.
4. Convert to a template only after validation.

## Validation Gate

Before proceeding to Chapter 02:

- [ ] Proxmox installed
- [ ] ISO storage available
- [ ] Windows 10 ISO downloaded
- [ ] Windows 11 ISO downloaded
- [ ] Ubuntu Desktop 24.04 ISO downloaded
- [ ] Ubuntu Server 24.04 ISO downloaded
- [ ] VirtIO ISO downloaded
- [ ] Cloudflare account available
- [ ] Docker/Guacamole host planned

## Common Mistakes

- Skipping native RDP/SSH testing.
- Converting a VM to a template before updates complete.
- Mixing DHCP and static addressing without documentation.
- Forgetting the QEMU Guest Agent.

## Next Chapter

Continue with **02-Proxmox-Preparation.md**, where you'll configure storage, networking, ISOs, VirtIO drivers, and the base environment before creating any virtual machines.
