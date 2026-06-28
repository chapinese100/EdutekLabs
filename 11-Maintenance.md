# Chapter 11 -- Maintenance & Operations

> EduTek Labs Enterprise Deployment Manual

## Objective

Maintain a stable, secure, and repeatable lab environment throughout the
academic year while minimizing downtime and ensuring templates remain
current.

------------------------------------------------------------------------

# Maintenance Philosophy

Never modify production student VMs to create the next semester's labs.

Always follow this lifecycle:

``` text
Golden Template
      ↓
Update & Validate
      ↓
Snapshot (Optional)
      ↓
Convert / Keep as Template
      ↓
Create Full Clones
      ↓
Student Labs
```

------------------------------------------------------------------------

# Monthly Maintenance

## Proxmox Host

1.  Verify backups completed successfully.
2.  Check available storage.
3.  Install Proxmox updates.
4.  Reboot during a maintenance window if required.

Commands:

``` bash
apt update
apt full-upgrade -y
pveversion
```

Validation:

-   Latest updates installed.
-   No failed services.

------------------------------------------------------------------------

## Windows Templates

For each Windows template:

1.  Convert the template back to a VM (or keep a maintenance clone).
2.  Start the VM.
3.  Install Windows Updates.
4.  Verify Device Manager has no errors.
5.  Verify RDP.
6.  Verify QEMU Guest Agent.
7.  Shut down.
8.  Convert back to a template.

Validation:

``` cmd
netstat -an | find "3389"
```

Expected:

    LISTENING

------------------------------------------------------------------------

## Ubuntu Desktop Templates

Update:

``` bash
sudo apt update
sudo apt full-upgrade -y
sudo reboot
```

Verify:

``` bash
systemctl status xrdp
systemctl status qemu-guest-agent
ss -tulpn | grep 3389
```

Perform a native RDP test before updating the template.

------------------------------------------------------------------------

## Ubuntu Server Templates

Update:

``` bash
sudo apt update
sudo apt full-upgrade -y
sudo reboot
```

Verify:

``` bash
systemctl status ssh
systemctl status qemu-guest-agent
ss -tulpn | grep 22
```

Confirm SSH works before templating.

------------------------------------------------------------------------

# Apache Guacamole

Monthly checks:

-   Remove unused accounts.
-   Review connection groups.
-   Verify browser access.
-   Test one Windows, one Ubuntu Desktop, and one Ubuntu Server
    connection.

------------------------------------------------------------------------

# Cloudflare Tunnel

Verify:

``` bash
sudo systemctl status cloudflared
```

Confirm external access to Guacamole after updates.

------------------------------------------------------------------------

# Backup Strategy

Recommended:

  Item                    Frequency
  ----------------------- ---------------------
  Proxmox configuration   Weekly
  VM Templates            Weekly
  Student VMs             Daily (if required)
  Guacamole database      Daily

Keep at least one offline backup.

------------------------------------------------------------------------

# Snapshot Strategy

Take a snapshot:

-   Before major OS upgrades
-   Before changing XRDP
-   Before updating Guacamole
-   Before upgrading Proxmox

Do not rely on snapshots as permanent backups.

------------------------------------------------------------------------

# New Semester Workflow

1.  Update every template.
2.  Validate every template.
3.  Create new Full Clones.
4.  Assign hostnames.
5.  Document IP addresses.
6.  Test native access.
7.  Test Guacamole.
8.  Provide access to students.

------------------------------------------------------------------------

# Retirement Workflow

At semester end:

1.  Export student work (if required).
2.  Delete cloned VMs.
3.  Keep Golden Templates.
4.  Archive documentation.
5.  Prepare templates for the next semester.

------------------------------------------------------------------------

# Operational Checklist

## Weekly

-   [ ] Backups successful
-   [ ] Storage healthy
-   [ ] Guacamole reachable

## Monthly

-   [ ] Update templates
-   [ ] Validate Windows RDP
-   [ ] Validate Ubuntu XRDP
-   [ ] Validate Ubuntu SSH
-   [ ] Verify Guest Agents
-   [ ] Review Cloudflare Tunnel

## Semester

-   [ ] Refresh templates
-   [ ] Create new clones
-   [ ] Test all lab access
-   [ ] Update documentation

------------------------------------------------------------------------

# Best Practices

-   Never edit Golden Templates directly while students are using them.
-   Test native RDP or SSH before testing Guacamole.
-   Keep documentation synchronized with infrastructure changes.
-   Standardize usernames, passwords, and naming conventions.
-   Use Full Clones for independent student environments.
-   Review security updates regularly.

------------------------------------------------------------------------

# Congratulations

You have completed the **EduTek Labs Enterprise Deployment Manual**.

By following these chapters you can:

-   Build Windows 10 and Windows 11 Golden Templates.
-   Build Ubuntu Desktop (XRDP) and Ubuntu Server (SSH) templates.
-   Deploy repeatable student labs.
-   Publish secure remote access through Apache Guacamole.
-   Maintain the environment over multiple academic terms using
    repeatable operational procedures.
