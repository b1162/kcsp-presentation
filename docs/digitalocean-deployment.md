# DigitalOcean Production Deployment Guide

This guide covers deploying the **WhatsApp Security Awareness Simulation Platform** on DigitalOcean using Docker and Docker Compose with HTTPS and automated SQLite backups.

---

## Architecture Overview

```
DigitalOcean Droplet (Ubuntu 22.04 LTS / 24.04 LTS)
│
├── Caddy / Nginx (Automatic Let's Encrypt TLS / HTTPS)
│       │  Port 80 / 443
│       ▼
├── Docker Container: kcsp-platform (Port 3000)
│       ├── Next.js Full-Stack Web App
│       ├── WhatsApp Provider Engine (WASender / Mock)
│       └── Tracking & Simulation Service
│
├── Persistent Volume: /var/lib/kcsp/data (prod.db SQLite in WAL mode)
│
└── Cron Service
        └── /etc/cron.daily/backup-kcsp -> infrastructure/digitalocean/backup-sqlite.sh
```

---

## 1. Droplet Provisioning

1. Create a DigitalOcean Droplet:
   - **Image**: Ubuntu 22.04 or 24.04 LTS
   - **Plan**: Basic Droplet - 1 vCPU / 2 GB RAM ($12-18/mo) is sufficient for thousands of participants.
   - **Region**: Choose a region compliant with your corporate data residency policies (e.g. Frankfurt, Amsterdam for EU GDPR).
2. Configure DNS:
   - Point your awareness domain (e.g. `simulation.yourcompany.com`) to the Droplet's Public IPv4 address.

---

## 2. Server Preparation

SSH into the Droplet:

```bash
ssh root@YOUR_DROPLET_IP

# Update packages and install Docker + Compose
apt-get update && apt-get upgrade -y
apt-get install -y git curl ufw sqlite3 gzip

curl -fsSL https://get.docker.com | sh
usermod -aG docker ubuntu || true

# Setup Firewall
ufw allow OpenSSH
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable
```

---

## 3. Clone and Configure Platform

```bash
git clone https://github.com/your-org/whatsapp-security-awareness.git /opt/kcsp
cd /opt/kcsp

# Create persistent storage directory
mkdir -p /opt/kcsp/data
chown -R 1000:1000 /opt/kcsp/data

# Configure environment variables
cp .env.example .env
nano .env
```

Ensure the following are set in `.env`:
```env
NEXT_PUBLIC_APP_URL="https://simulation.yourcompany.com"
SESSION_SECRET="generate-a-strong-random-64-character-string"
PRIVACY_SALT="generate-a-strong-random-salt-string"
DATABASE_URL="file:/app/data/prod.db"
WASENDER_API_KEY="your-wasender-api-key"
WASENDER_WEBHOOK_SECRET="your-webhook-secret"
NODE_ENV="production"
```

---

## 4. Run with Docker Compose

```bash
cd /opt/kcsp
docker compose up -d --build

# Verify container is healthy
docker compose ps
curl http://localhost:3000/api/health
```

---

## 5. Reverse Proxy with Caddy (Automatic HTTPS)

Install Caddy on the host:
```bash
apt-get install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-stable.list
apt-get update && apt-get install caddy -y
```

Configure `/etc/caddy/Caddyfile`:
```caddyfile
simulation.yourcompany.com {
    reverse_proxy localhost:3000

    # Security Headers
    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
        X-Content-Type-Options "nosniff"
        X-Frame-Options "SAMEORIGIN"
        X-XSS-Protection "1; mode=block"
        Referrer-Policy "strict-origin-when-cross-origin"
    }
}
```

Reload Caddy:
```bash
systemctl reload caddy
```
Caddy automatically obtains and renews Let's Encrypt certificates.

---

## 6. Automated Backup Setup

Setup daily automated backups of the SQLite database:

```bash
chmod +x /opt/kcsp/infrastructure/digitalocean/backup-sqlite.sh

# Add to crontab
(crontab -l 2>/dev/null; echo "0 3 * * * BACKUP_DIR=/var/backups/kcsp DB_FILE=/opt/kcsp/data/prod.db /opt/kcsp/infrastructure/digitalocean/backup-sqlite.sh >> /var/log/kcsp-backup.log 2>&1") | crontab -
```
