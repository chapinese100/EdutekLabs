# Chapter 03 -- Windows 10 Golden Template

> EduTek Labs Enterprise Deployment Manual

## Objective

Create a Windows 10 Pro virtual machine in Proxmox, configure it for
remote access with RDP, validate connectivity, and convert it into a
reusable Golden Template for Apache Guacamole.

------------------------------------------------------------------------

# Phase 1 -- Create the VM

## Recommended VM Configuration

  Setting           Value
  ----------------- --------------------
  VM Generation     q35
  BIOS              OVMF (UEFI)
  EFI Disk          Enabled
  TPM               Optional (Win10)
  SCSI Controller   VirtIO SCSI Single
  Disk              80 GB+
  CPU Type          host
  vCPU              2--4
  Memory            4096--8192 MB
  Network           VirtIO

### GUI

1.  Datacenter → Node → **Create VM**
2.  Name: `WIN10-TEMPLATE`
3.  Select Windows 10 ISO.
4.  Use the settings above.
5.  Add the **VirtIO Driver ISO** as a second CD/DVD after creation.

Validation: - VM appears in Proxmox. - No hardware warnings.

------------------------------------------------------------------------

# Phase 2 -- Install Windows

Boot the VM.

When Windows asks where to install:

1.  Click **Load driver**
2.  Browse the VirtIO CD: `vioscsi\w10\amd64`
3.  Load the storage driver.
4.  Select the virtual disk and complete installation.

------------------------------------------------------------------------

# Phase 3 -- Install VirtIO Drivers

Inside Windows:

Open the VirtIO CD and run:

``` text
virtio-win-guest-tools.exe
```

Install all default components.

Validation:

Open **Device Manager**.

Expected:

-   No Unknown Devices
-   VirtIO network adapter installed
-   QEMU Guest Agent installed

------------------------------------------------------------------------

# Phase 4 -- Install Windows Updates

Settings → Windows Update

Repeat until:

-   No pending updates.
-   Restart whenever prompted.

------------------------------------------------------------------------

# Phase 5 -- Rename Computer

Settings → System → About → Rename this PC

Example:

``` text
WIN10-TEMPLATE
```

Restart.

------------------------------------------------------------------------

# Phase 6 -- Create Lab User

Settings → Accounts → Other users

Create:

``` text
Username: student
```

Grant Administrator only if required by your labs.

------------------------------------------------------------------------

# Phase 7 -- Configure Networking

Preferred:

-   DHCP reservation on router.

Alternative:

Configure a static IPv4 address.

Verify:

``` cmd
ipconfig
```

------------------------------------------------------------------------

# Phase 8 -- Enable Remote Desktop

Settings → System → Remote Desktop

Enable:

-   Remote Desktop ✔
-   Require Network Level Authentication ✔

------------------------------------------------------------------------

# Phase 9 -- Windows Firewall

Open:

Windows Defender Firewall → Allow an app.

Verify:

-   Remote Desktop is allowed on Private networks.

Or run PowerShell as Administrator:

``` powershell
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

------------------------------------------------------------------------

# Phase 10 -- Verify RDP Service

``` powershell
Get-Service TermService
```

Expected:

``` text
Status : Running
```

------------------------------------------------------------------------

# Phase 11 -- Verify Port 3389

Command Prompt:

``` cmd
netstat -an | find "3389"
```

Expected:

``` text
TCP    0.0.0.0:3389    LISTENING
```

------------------------------------------------------------------------

# Phase 12 -- Native RDP Test

From another computer:

``` text
mstsc
```

Connect to:

``` text
<Windows10-IP>:3389
```

Do **not** continue until native RDP works.

------------------------------------------------------------------------

# Phase 13 -- Test from Guacamole Server

From the Linux host running Guacamole:

``` bash
nc -zv <Windows10-IP> 3389
```

Expected:

``` text
Connection succeeded
```

------------------------------------------------------------------------

# Phase 14 -- Create Guacamole Connection

Protocol: **RDP**

Network:

-   Hostname: Windows IP
-   Port: 3389

Authentication:

-   Username: student
-   Password: `<password>`{=html}

Security:

-   Mode: Any
-   Ignore server certificate: Enabled

Save and connect.

------------------------------------------------------------------------

# Phase 15 -- Install QEMU Guest Agent

Verify in Windows:

``` powershell
Get-Service qemu-ga
```

Expected:

``` text
Running
```

Verify in Proxmox:

VM → Summary → Guest Agent information is populated.

------------------------------------------------------------------------

# Phase 16 -- Prepare for Template

-   Delete temporary files.
-   Empty Recycle Bin.
-   Confirm Windows Update is complete.
-   Shut down the VM.

Convert:

Right-click VM → **Convert to Template**

------------------------------------------------------------------------

# Validation Checklist

-   [ ] Windows installed
-   [ ] VirtIO drivers installed
-   [ ] Device Manager clean
-   [ ] QEMU Guest Agent running
-   [ ] Lab user created
-   [ ] Static IP or DHCP reservation configured
-   [ ] RDP enabled
-   [ ] Firewall configured
-   [ ] Port 3389 listening
-   [ ] Native RDP works
-   [ ] Guacamole RDP works
-   [ ] Template created

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting VirtIO drivers.
-   Using an unsupported Windows edition without Remote Desktop.
-   Testing Guacamole before testing native RDP.
-   Forgetting to install the QEMU Guest Agent.

# Next Chapter

Proceed to **04-Windows11-Golden-Template.md**.
