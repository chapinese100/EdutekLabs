# Chapter 05 -- Ubuntu Desktop 24.04 Golden Template (XRDP)

> EduTek Labs Enterprise Deployment Manual

## Objective

Build an Ubuntu Desktop 24.04 LTS virtual machine on Proxmox VE,
configure it for XRDP, validate native RDP access, integrate it with
Apache Guacamole, and convert it into a reusable Golden Template.

------------------------------------------------------------------------

# Phase 1 -- Create the VM

## Recommended VM Configuration

  Setting    Value
  ---------- ----------------------
  BIOS       OVMF (UEFI)
  Machine    q35
  CPU Type   host
  vCPU       2--4
  RAM        4096--8192 MB
  Disk       60 GB+ (VirtIO SCSI)
  Network    VirtIO

Validation: - VM created successfully. - Ubuntu Desktop ISO attached.

------------------------------------------------------------------------

# Phase 2 -- Install Ubuntu Desktop

1.  Boot from the Ubuntu 24.04 Desktop ISO.
2.  Select **Install Ubuntu**.
3.  Choose Normal Installation.
4.  Enable third-party software (optional).
5.  Erase the virtual disk.
6.  Create an administrator account (example: `student`).

After installation:

``` bash
sudo apt update
sudo apt full-upgrade -y
sudo reboot
```

------------------------------------------------------------------------

# Phase 3 -- Install QEMU Guest Agent

``` bash
sudo apt install -y qemu-guest-agent
sudo systemctl enable --now qemu-guest-agent
```

Verify:

``` bash
systemctl status qemu-guest-agent
```

Expected:

    Active: active (running)

In Proxmox, verify **Guest Agent** information appears under the VM
Summary.

------------------------------------------------------------------------

# Phase 4 -- Configure Networking

Preferred: - DHCP reservation on your router.

Alternative: - Configure a static IPv4 address in Ubuntu Settings.

Verify:

``` bash
ip addr
hostname -I
```

Record the IP address.

------------------------------------------------------------------------

# Phase 5 -- Install XFCE (Recommended)

Although Ubuntu Desktop ships with GNOME, XFCE provides a more reliable
XRDP experience.

``` bash
sudo apt install -y xfce4 xfce4-goodies
```

Verify:

``` bash
dpkg -l | grep xfce4
```

------------------------------------------------------------------------

# Phase 6 -- Install XRDP

``` bash
sudo apt install -y xrdp
sudo systemctl enable --now xrdp
```

Check the service:

``` bash
systemctl status xrdp
```

Expected:

    Active: active (running)

------------------------------------------------------------------------

# Phase 7 -- Configure XRDP Session

For the user that will log in (example: `student`):

``` bash
echo "startxfce4" > ~/.xsession
chmod +x ~/.xsession
```

Verify:

``` bash
cat ~/.xsession
```

Expected:

    startxfce4

------------------------------------------------------------------------

# Phase 8 -- Fix SSL Permissions

Ubuntu 24.04 commonly requires the `xrdp` service account to be in the
`ssl-cert` group.

``` bash
sudo adduser xrdp ssl-cert
sudo systemctl restart xrdp
```

Verify:

``` bash
groups xrdp
```

Expected to include:

    ssl-cert

------------------------------------------------------------------------

# Phase 9 -- Configure Firewall

``` bash
sudo ufw allow 3389/tcp
sudo ufw enable
sudo ufw status
```

Expected:

    3389/tcp ALLOW

------------------------------------------------------------------------

# Phase 10 -- Verify XRDP

Verify the listening port:

``` bash
ss -tulpn | grep 3389
```

Expected:

    LISTEN ... :3389

------------------------------------------------------------------------

# Phase 11 -- Native RDP Test

Before configuring Guacamole, test XRDP with a native RDP client.

Windows:

    mstsc

Linux:

    Remmina

Connect to:

    <Ubuntu-IP>:3389

Do **not** continue until this succeeds.

------------------------------------------------------------------------

# Phase 12 -- Test from the Guacamole Server

``` bash
nc -zv <Ubuntu-IP> 3389
```

Expected:

    Connection succeeded

------------------------------------------------------------------------

# Phase 13 -- Configure Apache Guacamole

Create a new connection.

Protocol:

    RDP

Network:

-   Hostname: Ubuntu IP
-   Port: 3389

Authentication:

-   Username: student
-   Password: your password

Security:

-   Mode: Any
-   Ignore server certificate: Enabled

Save and connect.

------------------------------------------------------------------------

# Phase 14 -- Prepare the Golden Template

Before converting:

-   Ubuntu fully updated
-   QEMU Guest Agent running
-   XRDP verified
-   Native RDP verified
-   Guacamole verified

Shutdown the VM.

Convert in Proxmox:

    Right-click VM → Convert to Template

------------------------------------------------------------------------

# Validation Checklist

-   [ ] Ubuntu Desktop installed
-   [ ] QEMU Guest Agent running
-   [ ] Networking configured
-   [ ] XFCE installed
-   [ ] XRDP installed
-   [ ] `.xsession` configured
-   [ ] `xrdp` added to `ssl-cert`
-   [ ] Firewall allows 3389
-   [ ] Port 3389 listening
-   [ ] Native RDP successful
-   [ ] Guacamole RDP successful
-   [ ] Template created

------------------------------------------------------------------------

# Common Mistakes

-   Using Guacamole before native RDP works.
-   Forgetting to add `xrdp` to `ssl-cert`.
-   Missing `.xsession`.
-   Skipping system updates before templating.

------------------------------------------------------------------------

# Troubleshooting Quick Reference

## XRDP not running

``` bash
sudo systemctl restart xrdp
sudo systemctl status xrdp
```

## Port 3389 missing

``` bash
ss -tulpn | grep 3389
```

## View logs

``` bash
sudo tail -100 /var/log/xrdp.log
sudo tail -100 /var/log/xrdp-sesman.log
```

## Check firewall

``` bash
sudo ufw status
```

# Next Chapter

Proceed to **06-Ubuntu-Server-SSH.md**.
