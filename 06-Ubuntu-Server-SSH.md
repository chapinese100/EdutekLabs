# Chapter 06 -- Ubuntu Server 24.04 Golden Template (SSH)

> EduTek Labs Enterprise Deployment Manual

## Objective

Create an Ubuntu Server 24.04 LTS template in Proxmox VE that is
securely accessible through SSH and Apache Guacamole.

------------------------------------------------------------------------

# Phase 1 -- Create the VM

## Recommended Hardware

  Setting    Value
  ---------- ---------------------
  BIOS       OVMF (UEFI)
  Machine    q35
  CPU Type   host
  vCPU       2
  RAM        2048--4096 MB
  Disk       40 GB (VirtIO SCSI)
  Network    VirtIO

Attach the Ubuntu Server 24.04 ISO.

Validation: - VM created successfully.

------------------------------------------------------------------------

# Phase 2 -- Install Ubuntu Server

1.  Boot from the ISO.
2.  Select **Install Ubuntu Server**.
3.  Configure:
    -   Language
    -   Keyboard
    -   Time zone
4.  Configure networking:
    -   DHCP (recommended during installation)
5.  Create the administrator account:

Example:

``` text
Username: student
Hostname: ubuntu-server-template
```

6.  **Install OpenSSH Server** when prompted.

Complete the installation and reboot.

------------------------------------------------------------------------

# Phase 3 -- Update the System

``` bash
sudo apt update
sudo apt full-upgrade -y
sudo reboot
```

Validation:

``` bash
lsb_release -a
```

Expected:

    Ubuntu 24.04 LTS

------------------------------------------------------------------------

# Phase 4 -- Install QEMU Guest Agent

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

Verify in Proxmox that Guest Agent information appears.

------------------------------------------------------------------------

# Phase 5 -- Verify SSH

Check the service:

``` bash
systemctl status ssh
```

If OpenSSH was not installed:

``` bash
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
```

Verify:

``` bash
ss -tulpn | grep 22
```

Expected:

    LISTEN ... :22

------------------------------------------------------------------------

# Phase 6 -- Configure Firewall

``` bash
sudo ufw allow 22/tcp
sudo ufw enable
sudo ufw status
```

Expected:

    22/tcp ALLOW

------------------------------------------------------------------------

# Phase 7 -- Configure Networking

Preferred:

-   DHCP reservation

Alternative:

-   Static IPv4

Verify:

``` bash
hostname -I
ip addr
```

Document the IP address.

------------------------------------------------------------------------

# Phase 8 -- Native SSH Test

From another computer:

``` bash
ssh student@<SERVER-IP>
```

Accept the host key.

Verify login succeeds.

Do **not** continue until SSH works natively.

------------------------------------------------------------------------

# Phase 9 -- Test from the Guacamole Server

``` bash
nc -zv <SERVER-IP> 22
```

Expected:

    Connection succeeded

------------------------------------------------------------------------

# Phase 10 -- Configure Apache Guacamole

Create a new connection.

Protocol:

    SSH

Network:

-   Hostname: Server IP
-   Port: 22

Authentication:

-   Username: student
-   Password: your password

Save and connect.

------------------------------------------------------------------------

# Optional -- SSH Key Authentication

Generate a key pair on your workstation:

``` bash
ssh-keygen -t ed25519
```

Copy the public key:

``` bash
ssh-copy-id student@<SERVER-IP>
```

Verify key-based login:

``` bash
ssh student@<SERVER-IP>
```

------------------------------------------------------------------------

# Phase 11 -- Prepare the Golden Template

Before converting:

-   System updated
-   QEMU Guest Agent running
-   SSH verified
-   Firewall configured
-   Guacamole verified

Shutdown the VM.

In Proxmox:

    Right-click VM → Convert to Template

------------------------------------------------------------------------

# Validation Checklist

-   [ ] Ubuntu Server installed
-   [ ] OpenSSH installed
-   [ ] QEMU Guest Agent running
-   [ ] Networking configured
-   [ ] Firewall configured
-   [ ] Port 22 listening
-   [ ] Native SSH successful
-   [ ] Guacamole SSH successful
-   [ ] Template created

------------------------------------------------------------------------

# Troubleshooting

## SSH service not running

``` bash
sudo systemctl restart ssh
sudo systemctl status ssh
```

## Port 22 not listening

``` bash
ss -tulpn | grep 22
```

## Firewall blocks SSH

``` bash
sudo ufw status
sudo ufw allow 22/tcp
```

## Review logs

``` bash
sudo journalctl -u ssh --no-pager -n 50
```

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting to install OpenSSH during setup.
-   Forgetting to open port 22 in UFW.
-   Testing Guacamole before native SSH.
-   Converting the VM to a template before updates complete.

# Next Chapter

Proceed to **07-Apache-Guacamole.md**, where Windows RDP, Ubuntu XRDP,
and Ubuntu Server SSH connections will be configured and secured.
