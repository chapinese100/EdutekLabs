# Chapter 07 -- Apache Guacamole Configuration

> EduTek Labs Enterprise Deployment Manual

## Objective

Configure Apache Guacamole so Windows 10, Windows 11, Ubuntu Desktop
(XRDP), and Ubuntu Server (SSH) are securely accessible through a web
browser.

------------------------------------------------------------------------

# Before You Begin

Do **not** configure Guacamole until all of the following have been
validated:

-   Windows RDP works using `mstsc`
-   Ubuntu Desktop XRDP works using `mstsc` or Remmina
-   Ubuntu Server SSH works using `ssh`
-   QEMU Guest Agent is installed on every VM
-   Every VM has a documented IP address

------------------------------------------------------------------------

# Connection Matrix

  OS                   Protocol   Port
  ---------------- ------------ ------
  Windows 10                RDP   3389
  Windows 11                RDP   3389
  Ubuntu Desktop     RDP (XRDP)   3389
  Ubuntu Server             SSH     22

------------------------------------------------------------------------

# Log in as Administrator

Browse to:

    https://<guacamole-host>/

Default administrator:

    Username: guacadmin
    Password: guacadmin

**Immediately change the default password** after first login.

------------------------------------------------------------------------

# Create a Windows RDP Connection

Go to:

    Settings
    └── Connections
        └── New Connection

## General

  Field      Value
  ---------- ----------------
  Name       Windows 10 Lab
  Protocol   RDP

## Network

  Field      Value
  ---------- ---------------
  Hostname   Windows VM IP
  Port       3389

## Authentication

  Field      Value
  ---------- -------------
  Username   student
  Password   VM password
  Domain     Leave blank

## Security

  Field                       Value
  --------------------------- ---------
  Security mode               Any
  Ignore server certificate   Enabled

Save and connect.

------------------------------------------------------------------------

# Create an Ubuntu Desktop Connection

Protocol:

    RDP

Network:

-   Hostname: Ubuntu Desktop IP
-   Port: 3389

Authentication:

-   Username: student
-   Password: Linux password

Security:

-   Security Mode: Any
-   Ignore server certificate: Enabled

Display:

-   Color depth: 24-bit
-   Resize method: Display Update

------------------------------------------------------------------------

# Create an Ubuntu Server Connection

Protocol:

    SSH

Network:

-   Hostname: Ubuntu Server IP
-   Port: 22

Authentication:

-   Username: student
-   Password: Linux password

Optional:

-   Private Key (preferred for production)

------------------------------------------------------------------------

# Connection Groups

Create groups for organization.

Example:

    Windows Labs
    Ubuntu Desktop Labs
    Ubuntu Server Labs
    Networking Labs

Assign each connection to the appropriate group.

------------------------------------------------------------------------

# User Permissions

Create users for students.

Grant access only to the required connection groups.

Avoid sharing the administrator account.

------------------------------------------------------------------------

# Validation

## Windows

-   [ ] Browser opens desktop
-   [ ] Keyboard works
-   [ ] Mouse works
-   [ ] Clipboard works

## Ubuntu Desktop

-   [ ] XRDP login succeeds
-   [ ] Desktop loads
-   [ ] Clipboard works

## Ubuntu Server

-   [ ] SSH prompt appears
-   [ ] Commands execute
-   [ ] Terminal resizes correctly

------------------------------------------------------------------------

# Troubleshooting

## Remote Desktop Server Unreachable

Verify native RDP first.

    mstsc <VM-IP>

Then verify:

``` bash
nc -zv <VM-IP> 3389
```

## SSH Fails

``` bash
ss -tulpn | grep 22
sudo systemctl status ssh
```

## XRDP Fails

``` bash
sudo systemctl status xrdp
sudo tail -100 /var/log/xrdp.log
```

## Windows RDP Fails

Verify:

``` cmd
netstat -an | find "3389"
```

------------------------------------------------------------------------

# Best Practices

-   Test native connectivity before Guacamole.
-   Use templates and full clones.
-   Use connection groups.
-   Change default passwords.
-   Use least-privilege permissions.
-   Document every VM IP.

------------------------------------------------------------------------

# Next Chapter

Proceed to **08-Student-Provisioning.md** to automate and standardize
lab deployment using Proxmox templates.
