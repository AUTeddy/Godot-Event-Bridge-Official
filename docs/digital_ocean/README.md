# Deploying a Godot Dedicated Server on DigitalOcean (Ubuntu 22.04)

This guide walks you through spinning up a **DigitalOcean Droplet** and running your **Godot dedicated server** on it. It’s written in clean Markdown so you can paste it into StackEdit, GitHub, or your docs.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [1. Create a Droplet (DigitalOcean VPS)](#1-create-a-droplet-digitalocean-vps)
- [2. Prepare the Environment](#2-prepare-the-environment)
- [3. Upload Your Server Build](#3-upload-your-server-build)
  - [Option A: Linux Headless Build (Recommended)](#option-a-linux-headless-build-recommended)
  - [Option B: Run Windows Build with Wine (Not Recommended for Production)](#option-b-run-windows-build-with-wine-not-recommended-for-production)
- [4. Open Firewall Ports](#4-open-firewall-ports)
- [5. Run the Server](#5-run-the-server)
  - [Keep the server running after disconnect](#keep-the-server-running-after-disconnect)
- [6. Optional — Use a Domain Name](#6-optional--use-a-domain-name)
- [Quick Commands Reference](#quick-commands-reference)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

- A **DigitalOcean** account.
- A **Godot exported server build** (Linux headless preferred).
- (Optional) An SSH key set up locally for passwordless login.

> **Note:** Even if you exported a Windows dedicated server, you can (and should) export a **Linux headless** build for better performance and stability on Ubuntu.

---

## 1. Create a Droplet (DigitalOcean VPS)

1. Sign in to DigitalOcean and create a **Droplet**.
2. Choose an OS:
   - **Ubuntu 22.04 LTS** (recommended).
3. Choose resources:
   - **At least 1 GB RAM** for lightweight games. Scale up for heavier logic or more concurrent players.
4. Networking:
   - Enable **IPv4** (and **IPv6** if you need it).
5. Authentication:
   - Add your **SSH key** (recommended) or use a **root password**.

---

## 2. Prepare the Environment

**SSH into your Droplet:**
```bash
ssh root@YOUR_DROPLET_IP
```

**Update system packages:**
```bash
apt update && apt upgrade -y
```

**Install basic dependencies:**
```bash
apt install -y unzip wget
```

---

## 3. Upload Your Server Build

You have two paths. **Option A (Linux)** is strongly recommended.

### Option A: Linux Headless Build (Recommended)

**Re-export from Godot** as a Linux headless build (pass `--headless --server` when running).

**Upload the build (from your local machine):**
```bash
scp /path/to/your_linux_server_build.zip root@YOUR_DROPLET_IP:/root/
```

**On the server, unzip and make executable:**
```bash
unzip your_linux_server_build.zip
chmod +x ./your_server_binary
```

> Replace `your_server_binary` with the actual filename of your exported binary.

### Option B: Run Windows Build with Wine (Not Recommended for Production)

**Install Wine:**
```bash
apt install -y wine
```

**Upload and run:**
```bash
scp /path/to/your_windows_server_build.zip root@YOUR_DROPLET_IP:/root/
unzip your_windows_server_build.zip
wine your_server.exe --server
```

> **Caution:** Wine adds overhead and complexity. Prefer native Linux builds in production.

---

## 4. Open Firewall Ports

If you plan to enable UFW, allow **OpenSSH** first so you don’t lock yourself out, then open your game ports (e.g., **7777** UDP/TCP).

```bash
ufw allow OpenSSH
ufw allow 7777/tcp
ufw allow 7777/udp
ufw enable
```

> **Note:** Adjust `7777` to your game’s configured port(s).

---

## 5. Run the Server

### Linux headless
```bash
./your_server_binary --headless --server
```

### Windows build via Wine (not recommended)
```bash
wine your_server.exe --server
```

#### Keep the server running after disconnect

**Option 1: `nohup`**
```bash
nohup ./your_server_binary --headless --server > server.log 2>&1 &
tail -f server.log
```

**Option 2: `tmux`**
```bash
apt install -y tmux
tmux
./your_server_binary --headless --server
# Detach: Ctrl+b then d
```

> Logs will accumulate in `server.log` if you used `nohup`. Use `tail -f server.log` to follow logs.

---

## 6. Optional — Use a Domain Name

1. In your domain DNS settings, create an **A record** pointing your domain/subdomain (e.g., `game.example.com`) to your Droplet’s IP.
2. In your client, use `game.example.com` instead of the raw IP.
3. (Optional) If you later expose HTTP services (status pages, etc.), consider a reverse proxy with TLS (Nginx + Certbot).

---

## Quick Commands Reference

**SSH:**
```bash
ssh root@YOUR_DROPLET_IP
```

**Update & deps:**
```bash
apt update && apt upgrade -y
apt install -y unzip wget
```

**Upload build:**
```bash
scp /local/path/to/build.zip root@YOUR_DROPLET_IP:/root/
```

**Unzip & run (Linux):**
```bash
unzip your_linux_server_build.zip
chmod +x ./your_server_binary
./your_server_binary --headless --server
```

**Firewall (UFW):**
```bash
ufw allow OpenSSH
ufw allow 7777/tcp
ufw allow 7777/udp
ufw enable
```

**Keep alive (`nohup`):**
```bash
nohup ./your_server_binary --headless --server > server.log 2>&1 &
```

**Keep alive (`tmux`):**
```bash
tmux
./your_server_binary --headless --server
# Detach: Ctrl+b, then d
```

---

## Troubleshooting

- **I got disconnected after `ufw enable`.**  
  You likely didn’t allow OpenSSH first. From the console (DigitalOcean control panel), run:
  ```bash
  ufw allow OpenSSH
  ufw allow 7777/tcp
  ufw allow 7777/udp
  ufw enable
  ```

- **Binary won’t execute (`Permission denied`).**  
  Make sure it’s executable:
  ```bash
  chmod +x ./your_server_binary
  ```

- **Client can’t connect.**  
  - Confirm the Droplet’s firewall (UFW) and any **cloud firewall** rules allow your port (UDP/TCP as required).
  - Verify the server is actually listening on the expected port.
  - Check `server.log` for errors:
    ```bash
    tail -f server.log
    ```

- **Using Wine shows errors.**  
  Prefer a native **Linux headless** Godot export.

---

**That’s it!** Your Godot dedicated server should now be live on your DigitalOcean Droplet.
