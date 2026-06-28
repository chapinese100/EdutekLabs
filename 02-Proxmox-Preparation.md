# Chapter 02 -- Preparing Proxmox VE 8.x

> EduTek Labs Enterprise Deployment Manual

## Objective

Prepare a fresh Proxmox VE host so every Windows and Ubuntu VM created
later is ready to become a reusable template for Apache Guacamole.

------------------------------------------------------------------------

# Phase 1 -- Update Proxmox

## Why?

Always update Proxmox before creating templates.

## Step 1

SSH to Proxmox or open the Shell from the web UI.

``` bash
apt update
apt full-upgrade -y
reboot
```

## Verify

``` bash
pveversion
```

Expected:

-   Latest installed version is displayed.
-   No pending updates remain.

------------------------------------------------------------------------

# Phase 2 -- Verify Storage

GUI:

Datacenter → Storage

Verify:

-   local
-   local-lvm

Recommended layout:

  Storage     Purpose
  ----------- ---------------------
  local       ISO images, backups
  local-lvm   VM disks

------------------------------------------------------------------------

# Phase 3 -- Upload Installation Media

Upload:

-   Windows 10 ISO
-   Windows 11 ISO
-   Ubuntu Desktop 24.04 ISO
-   Ubuntu Server 24.04 ISO
-   VirtIO Drivers ISO

GUI:

Datacenter → local → ISO Images → Upload

Validation:

All ISOs appear in the ISO Images list.

------------------------------------------------------------------------

# Phase 4 -- Verify Network Bridge

GUI:

Node → System → Network

Verify bridge:

    vmbr0

Linux CLI:

``` bash
ip addr show vmbr0
```

Expected:

-   Bridge has the management IP.
-   Bridge is UP.

------------------------------------------------------------------------

# Phase 5 -- DNS

GUI:

Node → System → DNS

Recommended:

-   Gateway configured
-   Primary DNS: your router or enterprise DNS
-   Secondary DNS:
    -   1.1.1.1
    -   8.8.8.8

Verify:

``` bash
ping google.com
```

------------------------------------------------------------------------

# Phase 6 -- Time Synchronization

``` bash
timedatectl status
```

Expected:

-   NTP active
-   Correct timezone

------------------------------------------------------------------------

# Phase 7 -- Download VirtIO Drivers

Recommended source:

https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/

Store ISO in:

    local → ISO Images

------------------------------------------------------------------------

# Phase 8 -- Planning Templates

Create one template for each OS.

    Windows 10 Pro
    Windows 11 Pro
    Ubuntu Desktop 24.04
    Ubuntu Server 24.04

Never modify templates directly.

Always clone.

------------------------------------------------------------------------

# Phase 9 -- Validation Checklist

Before Chapter 03:

-   [ ] Proxmox updated
-   [ ] Storage verified
-   [ ] vmbr0 working
-   [ ] DNS working
-   [ ] Internet access confirmed
-   [ ] Windows 10 ISO uploaded
-   [ ] Windows 11 ISO uploaded
-   [ ] Ubuntu Desktop ISO uploaded
-   [ ] Ubuntu Server ISO uploaded
-   [ ] VirtIO ISO uploaded

------------------------------------------------------------------------

# Common Mistakes

-   Creating VMs before updating Proxmox.
-   Forgetting the VirtIO ISO.
-   Incorrect bridge selection.
-   Using templates directly instead of clones.

------------------------------------------------------------------------

# Next Chapter

Proceed to **03-Windows10-Golden-Template.md**.

The next chapter builds a Windows 10 VM from scratch, installs VirtIO
drivers, enables RDP, validates connectivity with mstsc, and prepares
the VM for Apache Guacamole.
