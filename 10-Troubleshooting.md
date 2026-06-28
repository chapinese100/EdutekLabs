# Chapter 10 -- Troubleshooting Guide

> EduTek Labs Enterprise Deployment Manual

## Objective

Provide a systematic process to diagnose and resolve the most common
issues encountered with Proxmox, Windows, Ubuntu, XRDP, SSH, Apache
Guacamole, and networking.

------------------------------------------------------------------------

# Troubleshooting Workflow

Always troubleshoot in this order:

``` text
Physical/VM Power
      ↓
Network Connectivity
      ↓
Operating System
      ↓
Required Service
      ↓
Listening Port
      ↓
Native Client
      ↓
Guacamole
```

Never start with Guacamole. First confirm the native protocol (RDP or
SSH) works.

------------------------------------------------------------------------

# Windows 10 / Windows 11

## RDP Does Not Work

### Symptoms

-   `mstsc` cannot connect
-   Guacamole reports *Remote desktop server unreachable*

### Step 1 -- Verify Remote Desktop

Settings → System → Remote Desktop

Confirm it is **Enabled**.

### Step 2 -- Verify Service

``` powershell
Get-Service TermService
```

Expected:

``` text
Running
```

If stopped:

``` powershell
Start-Service TermService
```

### Step 3 -- Verify Firewall

``` powershell
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

### Step 4 -- Verify Port

``` cmd
netstat -an | find "3389"
```

Expected:

``` text
LISTENING
```

### Step 5 -- Test from Another PC

``` text
mstsc <WINDOWS-IP>
```

If native RDP fails, **do not troubleshoot Guacamole yet**.

------------------------------------------------------------------------

# Ubuntu Desktop (XRDP)

## XRDP Service Not Running

``` bash
sudo systemctl status xrdp
```

Restart:

``` bash
sudo systemctl restart xrdp
```

## Port 3389 Missing

``` bash
ss -tulpn | grep 3389
```

No output indicates XRDP is not listening.

## Check Logs

``` bash
sudo tail -100 /var/log/xrdp.log
sudo tail -100 /var/log/xrdp-sesman.log
```

## Verify XFCE

``` bash
dpkg -l | grep xfce4
```

Install if missing:

``` bash
sudo apt install -y xfce4 xfce4-goodies
```

## Verify Session Startup

``` bash
cat ~/.xsession
```

Expected:

``` text
startxfce4
```

## Verify ssl-cert Membership

``` bash
groups xrdp
```

If `ssl-cert` is missing:

``` bash
sudo adduser xrdp ssl-cert
sudo systemctl restart xrdp
```

## Firewall

``` bash
sudo ufw status
sudo ufw allow 3389/tcp
```

------------------------------------------------------------------------

# Ubuntu Server (SSH)

## SSH Service

``` bash
systemctl status ssh
```

Start if required:

``` bash
sudo systemctl enable --now ssh
```

## Verify Port

``` bash
ss -tulpn | grep 22
```

## Firewall

``` bash
sudo ufw allow 22/tcp
```

## Test Natively

``` bash
ssh student@SERVER-IP
```

------------------------------------------------------------------------

# Proxmox

## Guest Agent Missing

Guest Agent not visible in VM Summary?

Inside guest:

``` bash
sudo systemctl status qemu-guest-agent
```

or Windows:

``` powershell
Get-Service qemu-ga
```

## VM Has No Network

Check:

-   Correct bridge (`vmbr0`)
-   VirtIO network adapter
-   DHCP/static IP
-   Gateway

------------------------------------------------------------------------

# Apache Guacamole

## Remote Desktop Server Unreachable

1.  Verify VM is powered on.
2.  Verify port 3389 is listening.
3.  Test native RDP.
4.  Verify hostname/IP in Guacamole.
5.  Confirm credentials.
6.  Set Security Mode to **Any**.
7.  Enable **Ignore server certificate**.

## SSH Connection Fails

1.  Verify SSH works natively.
2.  Confirm port 22.
3.  Check username/password.
4.  Verify firewall.

------------------------------------------------------------------------

# Cloudflare Tunnel

If Guacamole is unreachable externally:

-   Verify `cloudflared` is running.
-   Confirm DNS points to the tunnel.
-   Test Guacamole locally first.
-   Then test through Cloudflare.

------------------------------------------------------------------------

# Diagnostic Commands

## Windows

``` cmd
ipconfig
netstat -an
```

## Ubuntu

``` bash
hostname -I
ip addr
ss -tulpn
journalctl -xe
```

## Proxmox

``` bash
pveversion
ip addr
systemctl status pveproxy
```

------------------------------------------------------------------------

# Recovery Checklist

-   [ ] VM powered on
-   [ ] IP address verified
-   [ ] Guest Agent running
-   [ ] Required service running
-   [ ] Port listening
-   [ ] Firewall allows traffic
-   [ ] Native client succeeds
-   [ ] Guacamole succeeds

------------------------------------------------------------------------

# Best Practices

-   Always verify native connectivity first.
-   Snapshot before major updates.
-   Document every IP address.
-   Keep templates fully patched.
-   Use Full Clones for student labs.

------------------------------------------------------------------------

# Next Chapter

Proceed to **11-Maintenance.md** to learn how to maintain templates,
update Golden Images, back up Proxmox, and prepare labs for future
semesters.
