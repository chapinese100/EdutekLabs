# EduTek Labs

> Enterprise Remote Lab Platform using Proxmox VE, Apache Guacamole, Cloudflare Tunnel, Docker, and Golden VM Templates

---

# Table of Contents

1. Project Overview
2. Goals
3. Architecture
4. Technology Stack
5. Features
6. Prerequisites
7. Quick Start
8. Deployment Workflow
9. Documentation Structure
10. Security Model
11. Networking Model
12. Golden Template Strategy
13. Student Provisioning Lifecycle
14. Backup & Recovery
15. Troubleshooting Philosophy
16. Roadmap
17. Contributing
18. License

---

# Project Overview

EduTek Labs is an educational remote lab platform designed to provide browser-based access to Windows and Linux virtual machines without requiring VPNs, public IP addresses, or direct firewall exposure.

The platform combines:

- Proxmox VE
- Apache Guacamole
- Docker
- Cloudflare Tunnel
- Windows 10/11
- Ubuntu Desktop
- Ubuntu Server

The objective is to create a repeatable, secure, scalable environment for universities, bootcamps, technical training programs, and cybersecurity laboratories.

---

# Goals

- Standardized lab deployments
- Simplified student onboarding
- Secure remote access
- Template-driven infrastructure
- Reduced administration overhead
- Semester-to-semester repeatability
- Production-ready documentation

---

# High-Level Architecture

```text
Internet
    |
Cloudflare Edge
    |
Cloudflare Tunnel
    |
Apache Guacamole
    |
Proxmox VE
    |
+--------------------------+
| Windows 10              |
| Windows 11              |
| Ubuntu Desktop XRDP     |
| Ubuntu Server SSH       |
+--------------------------+
```

---

# Technology Stack

| Component | Purpose |
|-----------|---------|
| Proxmox VE | Virtualization |
| Docker | Container Runtime |
| Apache Guacamole | Browser Access |
| guacd | Protocol Proxy |
| MySQL | Guacamole Database |
| Cloudflared | Tunnel Connector |
| Windows 10 | Desktop Labs |
| Windows 11 | Desktop Labs |
| Ubuntu Desktop | Linux GUI Labs |
| Ubuntu Server | Linux Server Labs |

---

# Key Features

## Browser-Based Access

Students connect using a web browser.

No local client installation required.

## Secure Access

Cloudflare Tunnel removes the need for:

- Port forwarding
- Public IP addresses
- Reverse proxy complexity

## Multi-Platform Support

- RDP
- XRDP
- SSH

## Centralized Management

Guacamole provides:

- User accounts
- Connection groups
- Access control
- Session management

---

# Repository Layout

```text
edutek-labs/
│
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── CHANGELOG.md
│
├── docs/
│   ├── 01-Architecture.md
│   ├── 02-Proxmox-Preparation.md
│   ├── 03-Windows10-Golden-Template.md
│   ├── 04-Windows11-Golden-Template.md
│   ├── 05-Ubuntu-Desktop-XRDP.md
│   ├── 06-Ubuntu-Server-SSH.md
│   ├── 07-Apache-Guacamole.md
│   ├── 08-Student-Provisioning.md
│   ├── 09-Validation-Checklists.md
│   ├── 10-Troubleshooting.md
│   └── 11-Maintenance.md
│
├── docker/
│   ├── docker-compose.yml
│   ├── .env.example
│   └── mysql-init.sql
│
└── images/
```

---

# Quick Start

1. Install Proxmox VE
2. Upload Windows and Ubuntu ISOs
3. Build Golden Templates
4. Deploy Guacamole
5. Configure Cloudflare Tunnel
6. Create Student Clones
7. Assign Connections

---

# Deployment Workflow

```text
Prepare Proxmox
      ↓
Create Templates
      ↓
Validate Access
      ↓
Deploy Guacamole
      ↓
Configure Cloudflare
      ↓
Provision Students
      ↓
Operate Labs
```

---

# Documentation Roadmap

## Chapter 01

Architecture Planning

## Chapter 02

Proxmox Preparation

## Chapter 03

Windows 10 Template

## Chapter 04

Windows 11 Template

## Chapter 05

Ubuntu Desktop XRDP

## Chapter 06

Ubuntu Server SSH

## Chapter 07

Apache Guacamole

## Chapter 08

Student Provisioning

## Chapter 09

Validation Checklists

## Chapter 10

Troubleshooting

## Chapter 11

Maintenance

---

# Networking Model

Recommended lab network:

```text
Management VLAN
Student VLAN
Infrastructure VLAN
```

Document all IP allocations.

Avoid overlapping ranges.

---

# Security Model

## Cloudflare Access

Recommended:

Student
→ Cloudflare Access
→ Guacamole
→ Virtual Machine

Benefits:

- MFA
- Identity verification
- Conditional access

## Least Privilege

Students receive access only to:

- Assigned VM
- Assigned connection group

---

# Golden Template Strategy

Templates are immutable.

Workflow:

```text
Golden Template
       ↓
Full Clone
       ↓
Student Machine
```

Never assign templates directly.

---

# Student Provisioning Lifecycle

1. Create clone
2. Rename VM
3. Verify IP
4. Verify access
5. Create Guacamole connection
6. Assign permissions
7. Validate login

---

# Validation Philosophy

Always test in this order:

1. VM powered on
2. Network connectivity
3. Service running
4. Port listening
5. Native client
6. Guacamole

If native access fails:

Guacamole will fail.

---

# Backup Strategy

| Item | Frequency |
|--------|---------|
| Proxmox Config | Weekly |
| Templates | Weekly |
| Database | Daily |
| Student Labs | Daily |

---

# Maintenance

Monthly:

- Update Proxmox
- Update Templates
- Update Docker Images
- Validate Guacamole
- Validate Tunnel
- Test Backups

---

# Troubleshooting Philosophy

Windows:

Power → Network → RDP → Firewall → Guacamole

Ubuntu Desktop:

Power → Network → XRDP → Firewall → Guacamole

Ubuntu Server:

Power → Network → SSH → Firewall → Guacamole

---

# Intended Use Cases

- Universities
- Technical Schools
- Cybersecurity Programs
- Linux Training
- Windows Administration
- DevOps Labs
- Cloud Training

---

# Future Roadmap

- Terraform
- Ansible
- LDAP
- SAML
- Cloudflare Access Automation
- Monitoring
- Multi-Node Proxmox

---

# Contributing

Contributions are welcome.

Please:

- Test procedures
- Update documentation
- Preserve structure
- Submit clear pull requests

---

# License

MIT License

---

# Acknowledgements

Built using:

- Proxmox VE
- Apache Guacamole
- Docker
- Ubuntu
- Cloudflare Tunnel
- MySQL

---

# EduTek Labs

Remote Infrastructure for Modern Technical Education.
