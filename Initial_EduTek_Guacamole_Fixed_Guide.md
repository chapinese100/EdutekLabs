# 🖥️ EduTek Lab Production Guide

**Apache Guacamole + MySQL + Cloudflare Tunnel**
*Beginner Friendly — Fully Corrected & Battle-Tested Edition*

---

## 📋 Table of Contents

- [What We Are Building](#what-we-are-building)
- [Persistent Storage — What Needs It and Why](#-persistent-storage--what-needs-it-and-why)
- [Step 1 — Install Docker](#step-1--install-docker)
- [Step 2 — Create the Project Folder](#step-2--create-the-project-folder)
- [Step 3 — Create the Cloudflare Tunnel](#step-3--create-the-cloudflare-tunnel-dashboard-only)
- [Step 4 — Create the Environment File](#step-4--create-the-environment-file)
- [Step 5 — Create the Docker Compose File](#step-5--create-the-docker-compose-file)
- [Step 6 — Start the Containers](#step-6--start-the-containers)
- [Step 7 — Initialize the Guacamole Database](#step-7--initialize-the-guacamole-database)
- [Step 8 — Verify the Cloudflare Tunnel](#step-8--verify-the-cloudflare-tunnel)
- [Step 9 — Fix DNS in Cloudflare](#step-9--fix-dns-in-cloudflare)
- [Step 10 — Open Guacamole](#step-10--open-guacamole)
- [Step 11 — Enable Drive and Recordings Per Connection](#step-11--enable-drive-and-recordings-per-connection)
- [Recommended Security](#recommended-security--cloudflare-access)
- [Backup and Restore](#backup-and-restore)
- [Useful Commands](#useful-commands)
- [All Corrections Summary](#all-corrections-summary)

---

## What We Are Building

Students connect to virtual machines through a web browser — no VPN, no public IP required.

```
Student Browser → Cloudflare → Tunnel → Guacamole → Proxmox VMs
```

| Container | Role |
|-----------|------|
| `guacd` | Engine that handles RDP / VNC / SSH connections |
| `guacamole` | Web interface students log in to |
| `mysql` | Stores users, passwords, and connection settings |
| `cloudflared` | Secure tunnel to Cloudflare — no open firewall ports needed |

> **Domain used in this guide:** `eduteklabs.org`
> Replace it with your own domain wherever you see it.

---

## 💾 Persistent Storage — What Needs It and Why

Every container that writes data you want to keep needs a volume.
Without a volume, that data is gone the moment the container restarts or is updated.

| Container | Needs Persistent Storage? | What is stored | Volume name |
|-----------|--------------------------|----------------|-------------|
| `guacd` | ✅ **Yes** | Session recordings + shared drive files | `guac_data` |
| `guacamole` | ❌ No | Stateless web app — all data lives in MySQL or guacd | — |
| `mysql` | ✅ **Yes** | All users, passwords, and VM connection settings | `mysql_data` |
| `cloudflared` | ❌ No | Stateless — token comes from `.env`, nothing written to disk | — |

### Why guacd holds both recordings and the shared drive

`guacd` is the component that physically handles the remote desktop connection —
it streams every pixel and keystroke. This means:

- **Recordings** — guacd writes them because it sees the raw session data
- **Shared drive** — guacd handles the actual file transfer protocol to/from the VM,
  so it reads and writes the shared drive folder directly

The `guacamole` web container is stateless — it tells guacd what to do,
but guacd does the actual work and holds the files.

### What these volumes survive

- ✅ Container restarts
- ✅ `docker compose down` (without `-v`)
- ✅ Image updates via `docker compose pull` + `up -d`
- ❌ `docker compose down -v` — **permanently deletes both volumes and all data**

---

## Step 1 — Install Docker

Update your server:

```bash
sudo apt update && sudo apt upgrade -y
```

Install required packages:

```bash
sudo apt install ca-certificates curl wget gnupg -y
```

Add Docker's GPG key:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Add the Docker repository:

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo $VERSION_CODENAME) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Install Docker:

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Verify it worked:

```bash
docker --version
docker compose version
```

> 💡 You should see version numbers for both. If you get `command not found`, restart from the top of this step.

---

## Step 2 — Create the Project Folder

```bash
mkdir ~/guac-lab
cd ~/guac-lab
```

> 💡 Every command from here on should be run from inside `~/guac-lab` unless told otherwise.

---

## Step 3 — Create the Cloudflare Tunnel (Dashboard Only)

> ### ✏️ Correction
> The original guide told you to run `cloudflared tunnel login`, `tunnel create`, and
> `tunnel route dns` on your server. **Those commands do not work here.** They only work
> when `cloudflared` is installed directly as a binary on the machine. When running it
> as a Docker container with a token, **everything is done in the Cloudflare Dashboard.**

### 3a — Log in to Cloudflare

1. Go to [https://one.dash.cloudflare.com](https://one.dash.cloudflare.com)
2. Log in to your account
3. Confirm your domain `eduteklabs.org` is added and active

### 3b — Create the Tunnel

> ### ✏️ Correction — Navigation path updated to match current Cloudflare dashboard (2026)
> Previous versions of this guide said: left sidebar → **Networks** → **Tunnels**.
> That path is outdated. The correct current path confirmed in the official Cloudflare
> docs is: **Zero Trust → Networks → Connectors → Cloudflare Tunnels**.
> **Tunnels** now sits one level deeper under a **Connectors** submenu.

1. Go to [https://one.dash.cloudflare.com](https://one.dash.cloudflare.com)
2. In the left sidebar click **Networks**
3. Click **Connectors** (submenu appears)
4. Click **Cloudflare Tunnels**
5. Click **Create a tunnel**
6. Select **Cloudflared** as the connector type → click **Next**
7. Name your tunnel: `guacamole`
8. Click **Save tunnel**

### 3c — Get Your Token

1. After saving, Cloudflare shows an **Install connector** screen
2. Click the **Docker** tab
3. You will see a long `docker run` command
4. Copy **only** the string after `--token` — it starts with `eyJ` and is very long
5. Save it — you will paste it into your `.env` file in Step 4

> ⚠️ **Do NOT copy the entire `docker run` command.** Copy only the token string starting
> with `eyJ`. No spaces, no quotes around it.

### 3d — Add the Public Hostname

> ### ✏️ Correction — Tab name updated to match current Cloudflare dashboard (2026)
> Previous versions of this guide called this tab **"Hostname routes"** — that was wrong.
> The correct current name confirmed in the official Cloudflare docs is
> **"Published application routes"**. The navigation path has also changed.

1. In the Cloudflare dashboard go to **Networking → Tunnels**
2. Select your **guacamole** tunnel and click **Edit**
3. Click the **Published application routes** tab
4. Click **Add a published application route**
5. Fill in exactly:

| Field | Value |
|-------|-------|
| Subdomain | `guac` |
| Domain | `eduteklabs.org` |
| Service Type | `HTTP` |
| URL | `guacamole:8080` |

6. Click **Save**

> 💡 `guacamole:8080` uses the container name — not `localhost`. All containers share
> the `guacnet` network and reach each other by name.

---

## Step 4 — Create the Environment File

```bash
nano .env
```

Paste the following — replace every `CHANGE_ME` and paste your tunnel token:

```env
MYSQL_ROOT_PASSWORD=CHANGE_ME_ROOT_PASSWORD
MYSQL_DATABASE=guacamole_db
MYSQL_USER=guacuser
MYSQL_PASSWORD=CHANGE_ME_DATABASE_PASSWORD

TUNNEL_TOKEN=PASTE_YOUR_CLOUDFLARE_TOKEN_HERE
```

Save and close: `Ctrl+X` → `Y` → `Enter`

> ⚠️ **Do not use special characters like `@`, `$`, `!`, or `#` in your passwords.**
> They break Docker's environment variable substitution and will cause the tunnel to reject
> its own token even though the token itself is correct. Use only letters and numbers.

---

## Step 5 — Create the Docker Compose File

```bash
nano docker-compose.yml
```

Paste the following exactly:

```yaml
services:

  # ---------------------------------------------------------------------------
  # GUACD - Remote Desktop Proxy Engine
  # ---------------------------------------------------------------------------
  # The backend engine that physically handles RDP, VNC, and SSH connections.
  # Students never interact with this container directly.
  #
  # PERSISTENT STORAGE - guac_data volume mounted at /data
  #
  #   /data/recordings  - Session recordings. guacd writes a binary file for
  #                        every session that Guacamole can replay like a video.
  #                        Without this volume recordings are lost on restart.
  #
  #   /data/drive       - Shared file drive. guacd handles the actual file
  #                        transfer protocol to/from the VM, so it reads and
  #                        writes the shared drive folder directly.
  #                        Without this volume uploaded files are lost on restart.
  # ---------------------------------------------------------------------------
  guacd:
    image: guacamole/guacd:latest
    restart: unless-stopped
    volumes:
      - guac_data:/data
    networks:
      - guacnet


  # ---------------------------------------------------------------------------
  # MYSQL - Database
  # ---------------------------------------------------------------------------
  # Stores everything Guacamole knows: user accounts, hashed passwords,
  # connection definitions (which VM, which protocol, which credentials),
  # and user-to-connection permissions.
  #
  # PERSISTENT STORAGE - mysql_data volume mounted at /var/lib/mysql
  #   This is the most critical volume in the stack.
  #   Losing it means losing ALL users and ALL connection settings.
  # ---------------------------------------------------------------------------
  mysql:
    image: mysql:8.0
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    command:
      - --default-authentication-plugin=mysql_native_password
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - guacnet


  # ---------------------------------------------------------------------------
  # GUACAMOLE - Web Application
  # ---------------------------------------------------------------------------
  # The web interface students open in their browser.
  # Authenticates users against MySQL and sends connection requests to guacd.
  # This container is stateless - all data lives in MySQL or in guacd's volume.
  #
  # NO PERSISTENT VOLUME NEEDED on this container.
  #
  # Key environment variables:
  #
  #   GUACD_HOSTNAME      - Tells Guacamole where guacd lives on the Docker
  #                          network. FIXED: was missing in the original guide.
  #                          Without it Guacamole defaults to localhost, which
  #                          is wrong inside Docker — all sessions fail.
  #
  #   RECORDING_ENABLED   - Installs the session recording playback extension
  #                          so admins can replay sessions from the Guacamole UI.
  #                          Confirmed correct variable name per Apache docs.
  #
  #   RECORDING_SEARCH_PATH - The folder where Guacamole looks for recording
  #                            files to play back. Must match the path used in
  #                            the guacd volume above (/data/recordings).
  #                            Confirmed correct variable name per Apache docs.
  #
  # The guac_data volume is also mounted here so Guacamole can read recordings
  # for in-browser playback without copying files between containers.
  # ---------------------------------------------------------------------------
  guacamole:
    image: guacamole/guacamole:latest
    restart: unless-stopped
    depends_on:
      - mysql
      - guacd
    environment:
      GUACD_HOSTNAME: guacd
      MYSQL_HOSTNAME: mysql
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      RECORDING_ENABLED: "true"
      RECORDING_SEARCH_PATH: /data/recordings
    volumes:
      - guac_data:/data
    networks:
      - guacnet


  # ---------------------------------------------------------------------------
  # CLOUDFLARED - Cloudflare Tunnel
  # ---------------------------------------------------------------------------
  # Creates an outbound-only encrypted tunnel from your server to Cloudflare.
  # Students reach Guacamole through Cloudflare's edge — your server never
  # needs a public IP or open firewall ports.
  #
  # NO PERSISTENT STORAGE NEEDED.
  #   The tunnel token is loaded from the .env file at startup.
  #   Nothing is written to disk. Cloudflare holds all tunnel state.
  #
  # FIXED: command was "tunnel run" in the original guide.
  #   --no-autoupdate prevents cloudflared from trying to update itself inside
  #   the container. The filesystem is read-only so the update crashes the
  #   process. This flag disables that behavior completely.
  # ---------------------------------------------------------------------------
  cloudflared:
    image: cloudflare/cloudflared:latest
    restart: unless-stopped
    command: tunnel --no-autoupdate run
    environment:
      TUNNEL_TOKEN: ${TUNNEL_TOKEN}
    networks:
      - guacnet


# ---------------------------------------------------------------------------
# NAMED VOLUMES
# ---------------------------------------------------------------------------
# Docker manages these on the host at /var/lib/docker/volumes/
#
# These volumes SURVIVE:
#   - Container restarts
#   - Image updates (docker compose pull && docker compose up -d)
#   - docker compose down   (WITHOUT the -v flag)
#
# These volumes are PERMANENTLY DELETED by:
#   - docker compose down -v   <-- never run this unless you want to wipe everything
#
# guac_data  - Recordings and shared drive files (written by guacd, read by guacamole)
# mysql_data - All Guacamole users, passwords, and connection settings
# ---------------------------------------------------------------------------
volumes:
  guac_data:
  mysql_data:


# ---------------------------------------------------------------------------
# NETWORK
# ---------------------------------------------------------------------------
# All containers share this bridge network so they reach each other by name.
#   guacamole  -->  guacd        (hostname "guacd",      port 4822)
#   guacamole  -->  mysql        (hostname "mysql",      port 3306)
#   cloudflared --> guacamole    (hostname "guacamole",  port 8080)
#
# No container is directly exposed to the internet.
# Only cloudflared makes an outbound connection to Cloudflare's edge network.
# ---------------------------------------------------------------------------
networks:
  guacnet:
    driver: bridge
```

Save and close: `Ctrl+X` → `Y` → `Enter`

---

## Step 6 — Start the Containers

```bash
docker compose pull
docker compose up -d
```

Verify all four are running:

```bash
docker ps
```

You should see all four containers with status `Up`:

```
guac-lab-cloudflared-1
guac-lab-guacamole-1
guac-lab-guacd-1
guac-lab-mysql-1
```

> 💡 If any container shows `Restarting` or `Exiting`, check its logs:
> ```bash
> docker logs guac-lab-cloudflared-1
> ```
> Exit code `255` on cloudflared means the token has a problem — see Step 4.

---

## Step 7 — Initialize the Guacamole Database

Guacamole needs its database tables created before it can work. Run these **one at a time**.

Generate the SQL schema file:

```bash
docker run --rm guacamole/guacamole \
  /opt/guacamole/bin/initdb.sh --mysql > initdb.sql
```

Import it into MySQL — replace `YOUR_ROOT_PASSWORD` with your actual password (no space after `-p`):

```bash
docker exec -i $(docker ps -qf name=mysql) \
  mysql -u root -pCarl0s56gjtug guacamole_db < initdb.sql
```

Restart Guacamole so it connects to the initialized database:

```bash
docker restart $(docker ps -qf name=guacamole)
```

---

## Step 8 — Verify the Cloudflare Tunnel

Check the tunnel logs:

```bash
docker logs guac-lab-cloudflared-1
```

Look for a line like this — it means the tunnel is working:

```
Connection registered connIndex=0 ip=... location=...
```

Confirm in the Cloudflare Dashboard: **Zero Trust → Networks → Connectors → Cloudflare Tunnels**
Your tunnel should show a green **HEALTHY** badge.

> ⚠️ If it shows `Restarting (255)` — your token is wrong or a special character in
> your `.env` password corrupted how Docker read the token.
> Fix `.env`, then: `docker compose down && docker compose up -d`

---

## Step 9 — Fix DNS in Cloudflare

> ### ✏️ Real-World Issue
> Cloudflare sometimes creates a duplicate or incorrect DNS record when you save a
> published application route — especially if saved more than once, or if the wrong
> domain was used. Always verify DNS manually after adding a route.

### 9a — Check your DNS records

1. Go to [https://dash.cloudflare.com](https://dash.cloudflare.com)
2. Click your domain: `eduteklabs.org`
3. Click **DNS → Records**

You should see **exactly one** record like this:

| Type | Name | Content | Proxy status |
|------|------|---------|--------------|
| `CNAME` | `guac` | `<tunnel-id>.cfargotunnel.com` | 🟠 Proxied |

### 9b — Clean up wrong or duplicate records

Delete any records that are:
- Wrong domain (e.g. `guac.eduteklabs.com` when your domain is `.org`)
- Duplicate `guac` entries
- Type `Tunnel` instead of `CNAME`

### 9c — Add the CNAME manually if missing

1. Click **+ Add record** and fill in:

| Field | Value |
|-------|-------|
| Type | `CNAME` |
| Name | `guac` |
| Target | `<your-tunnel-id>.cfargotunnel.com` |
| Proxy status | **Proxied** (orange cloud ON) |
| TTL | `Auto` |

> 💡 Find your tunnel ID in **Zero Trust → Networks → Connectors → Cloudflare Tunnels → guacamole → Basic Information**.
> Full target example: `f09ec73a-5a9d-4900-8d33-e93f6673b65c.cfargotunnel.com`

2. Click **Save**

### 9d — Verify DNS resolved

```bash
nslookup guac.eduteklabs.org 1.1.1.1
```

You should see a Cloudflare IP in the answer. `NXDOMAIN` means the record didn't save — repeat 9c.

---

## Step 10 — Open Guacamole

```
https://guac.eduteklabs.org/guacamole
```

> ✏️ The `/guacamole` path is required. Without it you get a blank page or 404.

Default login:

```
Username: guacadmin
Password: guacadmin
```

> ⚠️ Change this password immediately after first login.
> Top-right menu → **guacadmin** → **Settings** → **Change Password**

---

## Step 11 — Enable Drive and Recordings Per Connection

The volumes are mounted and ready. But recordings and the shared drive must be
**enabled per connection** inside the Guacamole UI — they are not on by default.

### Enable session recording on a connection

1. Log in as `guacadmin`
2. Top-right menu → **Settings** → **Connections**
3. Click a connection (or create a new one)
4. Scroll to the **Screen Recording** section
5. Set **Recording path** to:
   ```
   /data/recordings
   ```
6. Set **Recording name** to:
   ```
   ${GUAC_USERNAME}_${GUAC_DATE}_${GUAC_TIME}
   ```
   This names each file by student username, date, and time automatically.
7. Check **Automatically create recording path**
8. Click **Save**

### Enable the shared drive on a connection

1. On the same connection edit screen scroll to **Device Redirection**
2. Check **Enable drive**
3. Set **Drive name** to: `Shared Drive`
4. Set **Drive path** to:
   ```
   /data/drive
   ```
5. Check **Automatically create drive path**
6. Click **Save**

> 💡 Students access the shared drive during a session by pressing `Ctrl+Shift+Alt`
> to open the Guacamole side panel, then clicking the folder icon.

### Verify recordings are being saved

After a test session ends, confirm the file exists:

```bash
docker exec guac-lab-guacd-1 ls -lah /data/recordings
```

You should see a file named with the username, date, and time.

---

## Recommended Security — Cloudflare Access

Cloudflare Access adds a second login in front of Guacamole — students must authenticate
with Cloudflare before they even see the Guacamole login page.

1. In the Cloudflare Dashboard go to **Zero Trust**
2. Click **Access controls → Applications**
3. Click **Create new application → Self-hosted and private**
4. Click **Add public hostname** and fill in:
   - **Domain:** `eduteklabs.org`
   - **Subdomain:** `guac`
5. Under **Access policies**, add a policy for who is allowed (email list, OTP, SSO, etc.)
6. Click **Save**

The full student login flow becomes:

```
Cloudflare Login → Guacamole Login → VM Desktop
```

---

## Backup and Restore

> ⚠️ Two volumes hold critical data: `mysql_data` and `guac_data`.
> Back up regularly. If the server is lost with no backup, everything is gone.

### Backup the database (users and connections)

```bash
docker exec $(docker ps -qf name=mysql) \
  mysqldump -u root -pYOUR_ROOT_PASSWORD guacamole_db \
  > ~/guac-lab/guacamole_backup_$(date +%F).sql
```

### Backup recordings and shared drive files

```bash
docker cp guac-lab-guacd-1:/data ~/guac-lab/guac_data_backup_$(date +%F)
```

### Automate database backup with cron (runs daily at 2am)

```bash
(crontab -l 2>/dev/null; echo "0 2 * * * docker exec \$(docker ps -qf name=mysql) mysqldump -u root -pYOUR_ROOT_PASSWORD guacamole_db > ~/guac-lab/guacamole_backup_\$(date +\%F).sql") | crontab -
```

Verify it was added:

```bash
crontab -l
```

### Restore the database

```bash
docker compose up -d
docker exec -i $(docker ps -qf name=mysql) \
  mysql -u root -pYOUR_ROOT_PASSWORD guacamole_db \
  < ~/guac-lab/guacamole_backup_2026-06-23.sql
docker restart $(docker ps -qf name=guacamole)
```

### Copy backups off the server (run from your LOCAL machine)

```bash
scp -r user@YOUR_SERVER_IP:~/guac-lab/guacamole_backup_*.sql ./
scp -r user@YOUR_SERVER_IP:~/guac-lab/guac_data_backup_* ./
```

---

## Useful Commands

```bash
# See all running containers and their status
docker ps

# Follow cloudflared tunnel logs live
docker logs guac-lab-cloudflared-1 --follow

# Check Guacamole logs for errors
docker logs guac-lab-guacamole-1

# Check guacd logs (recording issues show here)
docker logs guac-lab-guacd-1

# List all Docker volumes — confirm both exist
docker volume ls | grep guac

# See where a volume lives on disk
docker volume inspect guac-lab_mysql_data
docker volume inspect guac-lab_guac_data

# Confirm recording files exist after a test session
docker exec guac-lab-guacd-1 ls -lah /data/recordings

# Confirm shared drive files exist
docker exec guac-lab-guacd-1 ls -lah /data/drive

# Restart the entire stack
docker compose restart

# Pull latest images and restart — data is kept
docker compose pull && docker compose up -d

# Stop everything — both volumes are KEPT
docker compose down
```

```bash
# DANGER — permanently deletes mysql_data AND guac_data (recordings + drive)
docker compose down -v
```

> ⚠️ `docker compose down -v` wipes both volumes instantly and permanently.
> All users, all recordings, and all shared drive files are gone with no way to recover.
> Always back up first. Only use this to start from a completely clean slate.

---

## All Corrections Summary

| # | What was wrong | What was fixed |
|---|----------------|----------------|
| 1 | Steps 5-6 used `cloudflared tunnel login/create/route` | Removed — wrong method for Docker. Use Dashboard token method only |
| 2 | `cloudflared` command was `tunnel run` | Fixed to `tunnel --no-autoupdate run` — prevents startup crash |
| 3 | `GUACD_HOSTNAME: guacd` missing from guacamole service | Added — without it all remote desktop sessions fail |
| 4 | Browser URL was `https://guac.eduteklabs.org` | Fixed to `https://guac.eduteklabs.org/guacamole` — path is required |
| 5 | Passwords used `@` special character | Removed — special characters corrupt Docker env var parsing and break the token |
| 6 | Domain was incorrectly listed as `eduteklabs.com` | Corrected to `eduteklabs.org` throughout |
| 7 | DNS record was duplicated or wrong type after multiple saves | Added Step 9 to manually verify and clean up DNS records |
| 8 | No backup strategy documented | Added full Backup and Restore section with cron automation |
| 9 | No persistent storage for recordings or shared drive | Added `guac_data` volume on guacd covering `/data/recordings` and `/data/drive` |
| 10 | Wrong recording env var names (`RECORDING_PATH`, `ENABLE_DRIVE`) | Fixed to `RECORDING_ENABLED: "true"` and `RECORDING_SEARCH_PATH` per official Apache docs |
| 11 | Box-drawing characters in YAML comments caused parse error on line 23 | Replaced all `═══` with plain `---` comments — valid in all YAML parsers |
| 12 | Cloudflare tab name was wrong ("Hostname routes") | Corrected to **"Published application routes"** per current official Cloudflare docs (2026) |
| 13 | Tunnel creation path was **Networks → Tunnels** | Corrected to **Zero Trust → Networks → Connectors → Cloudflare Tunnels** per official docs |

---

*EduTek Labs — Last updated June 2026*
