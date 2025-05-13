# 🌐 Next.js VPS Deployment Guide (Without Docker)

This repository provides a comprehensive step-by-step guide to deploying a **Next.js** application on a **Linux-based VPS** using:

- Custom domain (e.g. `app.yourdomain.com`)
- Specific port (e.g. `4300`)
- Free SSL with Let's Encrypt
- NGINX reverse proxy
- Cloudflare DNS integration
- Swap setup for low-memory VPS
- Monitoring for uptime, SSL, and domain expiry

> 🛠️ This deployment **does not use Docker**, making it lightweight and straightforward for developers who prefer native Linux environments.

---

## 📌 Why Without Docker?

While Docker offers portability and consistency, not every environment or team requires it — especially for small VPS deployments or low-resource servers. In some cases, such as deployments for small businesses (UMKM) in Indonesia, teams may prefer to avoid Docker due to budget constraints and limited server resources.

---

## ✅ Advantages of Not Using Docker

- **Less overhead**: No need to run containers or manage Docker daemon.
- **Lower memory usage**: Ideal for low-RAM VPS (512MB–2GB).
- **Faster setup**: Simpler for beginners or one-off projects.
- **Direct access to logs and processes** via PM2 and Linux tools.

---

## ⚠️ Disadvantages of Not Using Docker

- **Less portability**: Harder to migrate across servers or replicate environments.
- **Manual environment management**: You must install Node.js, NGINX, etc., yourself.
- **Dependency conflicts**: No container isolation means your server environment can become messy over time.
- **No image versioning**: You can't easily roll back to previous builds like Docker images.

---

## 🛡️ Disclaimer

This setup is intended for small-to-medium production use cases. For highly scalable, multi-container, or enterprise-grade deployments, using Docker and orchestration tools like Kubernetes is recommended.

---

## 📬 Contributions Welcome

If you find this guide helpful, feel free to fork, star ⭐, or submit pull requests with improvements!

---

## 📋 Prerequisites

- **VPS** with Ubuntu 20.04 or newer
- **Root privileges** or `sudo` access on your VPS
- **Next.js** application ready for production
- **Custom domain** purchased (e.g., from Niagahoster)

---

## 🔧 1. VPS Initial Setup

Login to your VPS:

```bash
ssh root@your-server-ip
```

Update & install dependencies:

```bash
apt update && apt upgrade -y
apt install nginx ufw curl unzip git build-essential -y
```

---

## 🚀 2. Setup Domain (Example: `app.mydomain.com`)

### Using Cloudflare (Recommended):

1. Set nameservers to Cloudflare from Niagahoster.
2. In Cloudflare DNS tab, add:

   * **Type**: A
   * **Name**: `app`
   * **Value**: Your VPS IP
   * **Proxy status**: DNS only *(disable proxy for now, enable later)*

---

## ⚙️ 3. Deploy Next.js App on Custom Port (e.g., `4300`)

Clone or upload your app:

```bash
git clone https://github.com/your/repo.git app
cd app
```

Install Node.js & PM2:

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
apt install -y nodejs
npm install -g pm2
```

Build & run your Next.js app:

```bash
npm install
npm run build
pm2 start npm --name "nextapp" -- start -p 4300
pm2 save
```

---

## 🌐 4. NGINX Reverse Proxy + Free SSL (Let's Encrypt)

### NGINX Config:

Create file: `/etc/nginx/sites-available/app.mydomain.com`

```nginx
server {
    listen 80;
    server_name app.mydomain.com;

    location / {
        proxy_pass http://localhost:4300;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable site & restart NGINX:

```bash
ln -s /etc/nginx/sites-available/app.mydomain.com /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx
```

### Install Certbot & Get SSL:

```bash
apt install certbot python3-certbot-nginx -y
certbot --nginx -d app.mydomain.com
```

---

## 📈 5. Monitoring & Expiry Alerts

### a. SSL & Domain Expiry Monitoring

Use tools like:

* [UptimeRobot](https://uptimerobot.com)
* [BetterStack](https://betterstack.com)
* [PingAja](https://midteknologi.com/pingaja/)

Monitor `https://app.mydomain.com` for:

* Downtime alerts
* SSL expiry alerts

### b. PM2 Monitoring (Optional)

```bash
pm2 install pm2-server-monit
```

---

## 🧠 6. Low-Memory VPS Optimization

### Create a 2GB Swap File:

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

Check with:

```bash
free -h
```

✅ Prevents crashes during `npm run build` on low-memory VPS (512MB–2GB RAM).

---

## 🌩️ 7. Finalize Cloudflare Settings

Once SSL is working:

* Change **A Record Proxy status** to ✅ Proxied
* Go to **SSL/TLS** tab:

  * Select **Full (strict)**
* Enable:

  * Auto Minify (HTML, CSS, JS)
  * Brotli compression
  * Always Use HTTPS

---

## 🔁 8. Auto-Renew SSL

Certbot sets a cron job automatically.

Check timer:

```bash
systemctl list-timers | grep certbot
```

Manual test:

```bash
certbot renew --dry-run
```

---

## 📋 Summary

| Task                     | Tool/Step                     |
| ------------------------ | ----------------------------- |
| App Deployment           | PM2, Node.js                  |
| Reverse Proxy            | NGINX                         |
| SSL                      | Let's Encrypt via Certbot     |
| Domain & DNS             | Niagahoster → Cloudflare      |
| Monitoring               | UptimeRobot, BetterStack, PM2 |
| Swap for Build Process   | Manual swap creation          |
| Expiry Alerts            | UptimeRobot + Email alerts    |
| Performance Optimization | Cloudflare settings           |
