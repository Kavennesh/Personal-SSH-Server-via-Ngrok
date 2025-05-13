# 🐧 Personal SSH Server via Ngrok

This guide helps you set up a globally accessible SSH server on a Debian-based system using **Ngrok**, with no need for port forwarding or a static IP.

---

## ✅ What This Setup Includes

* Fresh Debian installation setup
* SSH server installation
* Ngrok tunnel configuration
* Optional: auto-start on boot via `systemd`
* Adding SSH users

---

## 🧾 Prerequisites

* Debian 11/12 (or Ubuntu Server)
* A Ngrok account ([https://ngrok.com](https://ngrok.com))
* Basic terminal access

---

## 🚀 Step-by-Step Setup

### 🔹 1. Install Debian OS

Install Debian on your server/laptop/VM. During install:

* Set hostname (e.g., `homelab`)
* Create user (e.g., `kavennesh`)
* Enable OpenSSH server if prompted

---

### 🔹 2. Install and Enable SSH Server

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
```

Test it:

```bash
ssh youruser@localhost
```

---

### 🔹 3. Add Users for SSH Access

To add a new user:

```bash
sudo adduser newusername
```

Follow prompts to set the password and details.

To allow the new user to use `sudo`:

```bash
sudo usermod -aG sudo newusername
```

To test SSH access:

```bash
ssh newusername@localhost
```

Ensure the home directory and `.ssh` folder are correctly set up if using key-based login.

---

### 🔹 4. Install Ngrok

```bash
curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc > /dev/null
echo "deb https://ngrok-agent.s3.amazonaws.com stable main" | sudo tee /etc/apt/sources.list.d/ngrok.list > /dev/null
sudo apt update
sudo apt install ngrok
```

---

### 🔹 5. Connect Ngrok to Your Account

```bash
ngrok config add-authtoken YOUR_NGROK_AUTH_TOKEN
```

Get this token from: [https://dashboard.ngrok.com/get-started/setup](https://dashboard.ngrok.com/get-started/setup)

---

### 🔹 6. Start the Tunnel

```bash
tmux
ngrok tcp 22
```

You will see something like:

```
Forwarding tcp://3.tcp.ngrok.io:"Port Number" -> localhost:22
```

Press `Ctrl + B` then `D` to detach and leave it running.

---

### 🔹 7. Connect Remotely (from another system)

```bash
ssh youruser@3.tcp.ngrok.io -p "Port Number"
```

Replace with the exact host/port ngrok shows.

---

## ♻️ Optional: Auto-Start Ngrok on Boot

### Step 1: Create startup script

```bash
sudo nano /usr/local/bin/start-ngrok.sh
```

Paste:

```bash
#!/bin/bash
ngrok tcp 22 --log stdout > /var/log/ngrok.log
```

Make executable:

```bash
sudo chmod +x /usr/local/bin/start-ngrok.sh
```

### Step 2: Create systemd service

```bash
sudo nano /etc/systemd/system/ngrok-ssh.service
```

Paste:

```ini
[Unit]
Description=Ngrok SSH Tunnel
After=network.target

[Service]
ExecStart=/usr/local/bin/start-ngrok.sh
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable ngrok-ssh
sudo systemctl start ngrok-ssh
```

---

## ✅ You’re Done!

You now have a Debian server with:

* Global SSH access via ngrok
* No need for public IP or router changes
* Secure access through encrypted tunnels

---

## 📦 Recommended GitHub Repo Structure

```
ngrok-ssh-setup/
├── README.md
├── start-ngrok.sh
└── systemd/
    └── ngrok-ssh.service
```
