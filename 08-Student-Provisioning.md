# Chapter 08 -- Student Provisioning & Lab Deployment

> EduTek Labs Enterprise Deployment Manual

## Objective

Deploy consistent student lab environments from Proxmox Golden Templates
while maintaining predictable naming, networking, and lifecycle
management.

------------------------------------------------------------------------

# Golden Template Rules

Never power on or modify a template.

Always:

    Template
        ↓
    Full Clone
        ↓
    Student VM

Recommended templates:

  Template                  Purpose
  ------------------------- -----------------
  WIN10-TEMPLATE            Windows 10 labs
  WIN11-TEMPLATE            Windows 11 labs
  UBUNTU-DESKTOP-TEMPLATE   XRDP labs
  UBUNTU-SERVER-TEMPLATE    SSH labs

------------------------------------------------------------------------

# Clone Strategy

GUI:

    Right-click Template
        ↓
    Clone

Use:

-   **Full Clone** (recommended)
-   Give the VM a unique ID
-   Select target storage

Avoid Linked Clones unless you understand their storage dependency.

------------------------------------------------------------------------

# Naming Convention

Recommended:

``` text
WIN10-STUDENT01
WIN10-STUDENT02
WIN11-STUDENT01
UBU-DESKTOP01
UBU-SERVER01
```

Keep names consistent with your inventory spreadsheet.

------------------------------------------------------------------------

# Networking

Preferred:

-   DHCP reservation per VM

Alternative:

-   Static IPv4

Document:

  VM                IP              Student
  ----------------- --------------- ------------
  WIN10-STUDENT01   192.168.1.101   Student 01
  UBU-DESKTOP01     192.168.1.121   Student 02

------------------------------------------------------------------------

# First Boot Checklist

For every clone:

-   Boot the VM
-   Verify IP address
-   Verify Guest Agent
-   Verify RDP/XRDP/SSH
-   Verify Guacamole connection
-   Rename host if required
-   Apply pending updates

------------------------------------------------------------------------

# Guacamole Organization

Create connection groups:

``` text
Course A
Course B
Networking
Cybersecurity
Linux
Windows
```

Assign only the required VMs to each class.

------------------------------------------------------------------------

# Student Accounts

Recommended:

One local account per VM:

``` text
Username: student
```

Change passwords between semesters.

Never share administrator credentials with students.

------------------------------------------------------------------------

# Validation

Windows:

``` cmd
netstat -an | find "3389"
```

Ubuntu Desktop:

``` bash
ss -tulpn | grep 3389
```

Ubuntu Server:

``` bash
ss -tulpn | grep 22
```

Confirm native connectivity before Guacamole.

------------------------------------------------------------------------

# End-of-Semester Procedure

1.  Archive student work if required.
2.  Delete cloned VMs.
3.  Keep templates.
4.  Update templates.
5.  Create fresh clones for the next semester.

------------------------------------------------------------------------

# Best Practices

-   Keep templates patched.
-   Use snapshots before major updates.
-   Record VM IDs and IP addresses.
-   Test one clone before deploying many.
-   Standardize usernames and passwords.

------------------------------------------------------------------------

# Validation Checklist

-   [ ] Template unchanged
-   [ ] Full clone created
-   [ ] Unique VM ID
-   [ ] IP documented
-   [ ] Native access verified
-   [ ] Guacamole access verified
-   [ ] Student account validated

------------------------------------------------------------------------

# Next Chapter

Proceed to **09-Validation-Checklists.md** for comprehensive deployment
verification before releasing labs to students.
