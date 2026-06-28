# Chapter 04 -- Windows 11 Golden Template

> EduTek Labs Enterprise Deployment Manual

## Objective

Deploy a Windows 11 Pro VM on Proxmox VE 8.x, configure it for Apache
Guacamole using RDP, validate end-to-end connectivity, and convert it
into a reusable Golden Template.

------------------------------------------------------------------------

# Phase 1 -- Create the VM

## Recommended Hardware

  Setting           Value
  ----------------- --------------------
  Machine           q35
  BIOS              OVMF (UEFI)
  EFI Disk          Enabled
  TPM               TPM 2.0 (Required)
  SCSI Controller   VirtIO SCSI Single
  Disk              100 GB
  CPU Type          host
  vCPU              4
  RAM               8192 MB
  Network           VirtIO

## GUI Steps

1.  Datacenter → Node → **Create VM**
2.  Name: `WIN11-TEMPLATE`
3.  Select the Windows 11 ISO.
4.  Choose **OVMF (UEFI)**.
5.  Add **EFI Disk**.
6.  Add **TPM State**.
7.  Select **q35** machine type.
8.  Use a **VirtIO SCSI** disk.
9.  Finish the wizard.
10. Edit the VM and attach the **VirtIO Drivers ISO** as a second
    CD/DVD.

Validation: - VM boots successfully. - Windows installer starts.

------------------------------------------------------------------------

# Phase 2 -- Install Windows 11

When the installer asks where to install Windows:

1.  Click **Load Driver**.
2.  Browse to the VirtIO CD.
3.  Open:

``` text
vioscsi\w11\amd64
```

4.  Load the driver.
5.  Select the virtual disk.
6.  Complete the installation.

------------------------------------------------------------------------

# Phase 3 -- Install VirtIO Drivers

Inside Windows:

Run:

``` text
virtio-win-guest-tools.exe
```

Install all default components.

Verify in **Device Manager**:

-   No Unknown Devices
-   VirtIO Storage Controller
-   VirtIO Network Adapter
-   QEMU Guest Agent

------------------------------------------------------------------------

# Phase 4 -- Windows Update

Settings → Windows Update

Continue installing updates until:

-   No pending updates
-   No optional driver updates remain (unless required)

Restart as needed.

------------------------------------------------------------------------

# Phase 5 -- Rename the Computer

Settings → System → About → Rename this PC

Example:

``` text
WIN11-TEMPLATE
```

Restart.

------------------------------------------------------------------------

# Phase 6 -- Create the Lab User

Settings → Accounts → Other users

Create:

``` text
Username: student
```

Use a strong password.

Grant Administrator privileges only if required.

------------------------------------------------------------------------

# Phase 7 -- Configure Networking

Preferred: - DHCP reservation

Alternative: - Static IPv4

Verify:

``` cmd
ipconfig
```

Document the assigned IP.

------------------------------------------------------------------------

# Phase 8 -- Install QEMU Guest Agent

Verify:

``` powershell
Get-Service qemu-ga
```

Expected:

``` text
Running
```

Verify in Proxmox that Guest Agent information is populated.

------------------------------------------------------------------------

# Phase 9 -- Enable Remote Desktop

Settings → System → Remote Desktop

Enable:

-   Remote Desktop
-   Require Network Level Authentication (recommended)

------------------------------------------------------------------------

# Phase 10 -- Configure Firewall

Open PowerShell as Administrator:

``` powershell
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

Verify the Remote Desktop firewall rules are enabled.

------------------------------------------------------------------------

# Phase 11 -- Verify the RDP Service

``` powershell
Get-Service TermService
```

Expected:

``` text
Status : Running
```

------------------------------------------------------------------------

# Phase 12 -- Verify Port 3389

``` cmd
netstat -an | find "3389"
```

Expected:

``` text
TCP    0.0.0.0:3389    LISTENING
```

------------------------------------------------------------------------

# Phase 13 -- Native RDP Test

From another computer:

Run:

``` text
mstsc
```

Connect to:

``` text
<Windows11-IP>:3389
```

Do not continue until this succeeds.

------------------------------------------------------------------------

# Phase 14 -- Test from the Guacamole Server

``` bash
nc -zv <Windows11-IP> 3389
```

Expected:

``` text
Connection succeeded
```

------------------------------------------------------------------------

# Phase 15 -- Create the Guacamole Connection

Protocol: **RDP**

Network: - Hostname: Windows 11 IP - Port: 3389

Authentication: - Username: student - Password: your password

Security: - Mode: Any - Ignore server certificate: Enabled

Save and test.

------------------------------------------------------------------------

# Phase 16 -- Prepare the Golden Template

Before converting:

-   Windows fully updated
-   No pending restarts
-   Temporary files removed
-   Recycle Bin emptied
-   Guest Agent running
-   RDP verified

Shutdown the VM.

In Proxmox:

**Right-click VM → Convert to Template**

------------------------------------------------------------------------

# Validation Checklist

-   [ ] Windows 11 installed
-   [ ] TPM 2.0 configured
-   [ ] OVMF enabled
-   [ ] VirtIO drivers installed
-   [ ] Device Manager clean
-   [ ] QEMU Guest Agent running
-   [ ] Windows Update complete
-   [ ] Lab user created
-   [ ] Networking configured
-   [ ] RDP enabled
-   [ ] Firewall configured
-   [ ] Port 3389 listening
-   [ ] Native RDP works
-   [ ] Guacamole RDP works
-   [ ] Template created

------------------------------------------------------------------------

# Common Mistakes

-   Forgetting TPM 2.0.
-   Forgetting to attach the VirtIO ISO.
-   Testing Guacamole before testing native RDP.
-   Leaving pending Windows Updates before templating.

# Next Chapter

Proceed to **05-Ubuntu-Desktop-XRDP.md**.









# Verify Windows RDP Configuration for Apache Guacamole

This section verifies that Remote Desktop Protocol (RDP) is enabled, running, and accessible so Apache Guacamole can connect successfully.

---

## Check 1 - Verify RDP is Enabled

### Purpose

Confirms that Windows allows incoming Remote Desktop connections.

### Command

Run PowerShell as Administrator:

```powershell
Get-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" -Name fDenyTSConnections
```

### Expected Result

```text
fDenyTSConnections : 0
```

### Interpretation

| Value | Meaning      |
| ----- | ------------ |
| 0     | RDP Enabled  |
| 1     | RDP Disabled |

### If Disabled

Enable RDP:

```powershell
Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" -Name fDenyTSConnections -Value 0
```

---

## Check 2 - Verify Remote Desktop Service

### Purpose

Confirms that the Windows RDP service is running.

### Command

```powershell
Get-Service TermService
```

### Expected Result

```text
Status   Name         DisplayName
------   ----         -----------
Running  TermService  Remote Desktop Services
```

### If Stopped

Start the service:

```powershell
Start-Service TermService
```

Configure automatic startup:

```powershell
Set-Service TermService -StartupType Automatic
```

Verify again:

```powershell
Get-Service TermService
```

---

## Check 3 - Verify Windows Firewall Rules

### Purpose

Confirms that Windows Firewall allows incoming RDP connections.

### Command

```powershell
Get-NetFirewallRule -DisplayGroup "Remote Desktop"
```

### Expected Result

Firewall rules should show:

```text
Enabled : True
```

### If Disabled

Enable all Remote Desktop firewall rules:

```powershell
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

### Verification

```powershell
Get-NetFirewallRule -DisplayGroup "Remote Desktop" | Select DisplayName, Enabled
```

---

## Check 4 - Verify Port 3389 is Listening

### Purpose

Confirms that Windows is actively listening for RDP connections.

### Command

```powershell
netstat -ano | findstr :3389
```

### Expected Result

```text
TCP    0.0.0.0:3389    0.0.0.0:0    LISTENING
```

or

```text
TCP    [::]:3389       [::]:0       LISTENING
```

### Alternative Command

```powershell
Get-NetTCPConnection -LocalPort 3389
```

### Expected Result

```text
State : Listen
```

---

## Check 5 - Verify Network Level Authentication (NLA)

### Purpose

Verifies whether Network Level Authentication is enabled.

Apache Guacamole supports NLA and it is recommended for security.

### Command

```powershell
(Get-WmiObject Win32_TSGeneralSetting -Namespace root\cimv2\terminalservices).UserAuthenticationRequired
```

### Expected Results

```text
1
```

NLA Enabled

```text
0
```

NLA Disabled

---

## Check 6 - Verify Local User Accounts

### Purpose

Ensures there is a valid Windows user account available for Guacamole authentication.

### Command

```powershell
Get-LocalUser
```

### Example Output

```text
Administrator
Student1
LabUser
```

### Notes

The selected user must:

* Have a password configured
* Be allowed to log in remotely
* Not be disabled or locked

---

## Check 7 - Verify RDP Access Permissions

### Purpose

Ensures the user account is authorized to use Remote Desktop.

### Command

```powershell
net localgroup "Remote Desktop Users"
```

### Example Output

```text
Alias name     Remote Desktop Users

Members

------------
Student1
LabUser
```

### Add a User

```powershell
net localgroup "Remote Desktop Users" Student1 /add
```

---

## Check 8 - Verify VM IP Address

### Purpose

Identifies the IP address that Guacamole must use.

### Command

```powershell
ipconfig
```

### Example Output

```text
IPv4 Address . . . . . . . . . . : 192.168.1.50
```

### Notes

Use this IP address as the Hostname field in Guacamole.

---

## Check 9 - Test Connectivity from the Guacamole Server

### Purpose

Verifies that the Guacamole server can reach the Windows VM.

### Command

```bash
nc -zv 192.168.1.50 3389
```

### Expected Result

```text
Connection to 192.168.1.50 3389 port [tcp/ms-wbt-server] succeeded
```

### Alternative

```bash
telnet 192.168.1.50 3389
```

If the connection succeeds, Guacamole should be able to communicate with the VM.

---

## Check 10 - Verify Guacamole Connection Settings

### Purpose

Confirms the Apache Guacamole connection configuration.

### Recommended Settings

| Setting            | Value            |
| ------------------ | ---------------- |
| Protocol           | RDP              |
| Hostname           | Windows VM IP    |
| Port               | 3389             |
| Username           | Windows Username |
| Password           | Windows Password |
| Security Mode      | NLA              |
| Ignore Certificate | Enabled          |

### Example

```text
Protocol: RDP
Hostname: 192.168.1.50
Port: 3389
Username: Student1
Password: ********
Security Mode: NLA
```

---

## Quick Verification Script

Run this PowerShell script as Administrator for a fast health check.

```powershell
Write-Host "===== RDP HEALTH CHECK ====="

Write-Host "`nRDP Enabled:"
(Get-ItemProperty `
-Path "HKLM:\System\CurrentControlSet\Control\Terminal Server").fDenyTSConnections

Write-Host "`nRemote Desktop Service:"
Get-Service TermService

Write-Host "`nFirewall Rules:"
Get-NetFirewallRule -DisplayGroup "Remote Desktop" |
Select DisplayName, Enabled

Write-Host "`nPort 3389:"
Get-NetTCPConnection -LocalPort 3389
```

### Healthy Results

```text
fDenyTSConnections = 0
TermService = Running
Firewall Rules = Enabled
Port 3389 = Listening
```

If all checks pass, the Windows VM is ready for Apache Guacamole RDP access.

