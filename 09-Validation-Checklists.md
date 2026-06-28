# Chapter 09 -- Validation Checklists

> EduTek Labs Enterprise Deployment Manual

## Objective

Validate every component of the EduTek Labs environment before releasing
virtual machines to students.

------------------------------------------------------------------------

# Validation Philosophy

Never assume a VM is ready.

Every template and every clone must pass validation.

The recommended validation order is:

``` text
VM Boots
      ↓
Networking
      ↓
Guest Agent
      ↓
Native Protocol
      ↓
Guacamole
      ↓
Template
```

------------------------------------------------------------------------

# Windows 10 Validation

## Operating System

-   [ ] Windows activated
-   [ ] Windows Update complete
-   [ ] Computer renamed

## Drivers

-   [ ] VirtIO Storage
-   [ ] VirtIO Network
-   [ ] Balloon Driver
-   [ ] QEMU Guest Agent

Open Device Manager.

Expected:

-   No Unknown Devices

------------------------------------------------------------------------

## Networking

``` cmd
ipconfig
```

Verify:

-   IPv4 assigned
-   Gateway correct
-   DNS correct

------------------------------------------------------------------------

## Guest Agent

``` powershell
Get-Service qemu-ga
```

Expected:

    Running

Verify Guest Agent data appears in Proxmox.

------------------------------------------------------------------------

## RDP

Verify service:

``` powershell
Get-Service TermService
```

Verify port:

``` cmd
netstat -an | find "3389"
```

Expected:

    LISTENING

Native validation:

``` text
mstsc
```

Successful login required.

------------------------------------------------------------------------

## Guacamole

-   [ ] Browser connection works
-   [ ] Mouse works
-   [ ] Keyboard works
-   [ ] Clipboard works

------------------------------------------------------------------------

# Windows 11 Validation

Repeat all Windows 10 checks plus:

-   [ ] TPM 2.0 detected
-   [ ] Secure Boot enabled
-   [ ] OVMF booting correctly

------------------------------------------------------------------------

# Ubuntu Desktop Validation

## System

``` bash
hostnamectl
```

Verify Ubuntu 24.04.

------------------------------------------------------------------------

## Guest Agent

``` bash
systemctl status qemu-guest-agent
```

Expected:

    active (running)

------------------------------------------------------------------------

## XRDP

``` bash
systemctl status xrdp
```

``` bash
ss -tulpn | grep 3389
```

Expected:

    LISTEN

------------------------------------------------------------------------

## Native RDP

Windows:

    mstsc

Linux:

    Remmina

Connection must succeed before Guacamole.

------------------------------------------------------------------------

## Guacamole

Verify:

-   Login
-   Desktop loads
-   Clipboard
-   Keyboard

------------------------------------------------------------------------

# Ubuntu Server Validation

## SSH

``` bash
systemctl status ssh
```

Verify port:

``` bash
ss -tulpn | grep 22
```

Native SSH:

``` bash
ssh student@SERVER-IP
```

------------------------------------------------------------------------

## Guacamole SSH

Verify:

-   Login
-   Terminal
-   Resize
-   Clipboard

------------------------------------------------------------------------

# Proxmox Validation

Verify:

-   [ ] Guest Agent data
-   [ ] Correct storage
-   [ ] Correct VM ID
-   [ ] Snapshot policy
-   [ ] Backup policy

------------------------------------------------------------------------

# Guacamole Validation

Verify every connection:

  VM               Protocol   Status
  ---------------- ---------- --------
  Windows 10       RDP        PASS
  Windows 11       RDP        PASS
  Ubuntu Desktop   XRDP       PASS
  Ubuntu Server    SSH        PASS

------------------------------------------------------------------------

# Release Checklist

Before students receive access:

-   [ ] All templates updated
-   [ ] All clones tested
-   [ ] Documentation updated
-   [ ] Passwords verified
-   [ ] Connection Groups assigned
-   [ ] Student accounts validated

------------------------------------------------------------------------

# Common Validation Failures

  Failure               Verify
  --------------------- --------------------------------------------------
  RDP unavailable       TermService, Firewall, Port 3389
  XRDP unavailable      xrdp service, `.xsession`, `ssl-cert`, Port 3389
  SSH unavailable       ssh service, Port 22, UFW
  Guest Agent missing   qemu-guest-agent service
  Guacamole fails       Native protocol first

------------------------------------------------------------------------

# Next Chapter

Proceed to **10-Troubleshooting.md** for detailed recovery procedures
covering Windows, Ubuntu, XRDP, SSH, Guacamole, Proxmox, and networking.
