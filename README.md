# Christian Baker — Personal Blog & Landing Page

Personal website and Ghost blog hosted on a VPS using Docker.

**Stack:** Nginx · Ghost 5 · MySQL 8 · Docker Compose  
**Live:** [christianbakerblog.candyfairstudio.com](https://christianbakerblog.candyfairstudio.com)

---

## Project Structure

```
.
├── docker-compose.yml        # All services
├── .env.example              # Environment variable template
├── .gitignore
├── nginx/
│   └── default.conf          # Reverse proxy — landing page + Ghost
├── landing/
│   ├── index.html            # Custom landing page
│   └── style.css             # Styling
└── ghost/                    # Ghost config overrides (if needed)
```

---

## Prerequisites

- VPS running Ubuntu 22.04+
- Docker installed
- Docker Compose plugin installed (see below)
- Port 80 (and 443 for SSL) open in firewall

---

## 1. Install Docker Compose

Docker Compose is now a plugin (not a separate binary):

```bash
sudo apt update
sudo apt install docker-compose-plugin
docker compose version   # should return v2.x
```

---

## 2. Clone the Repository

```bash
git clone https://github.com/roadtowiganpier/blog.git
cd blog
```

---

## 3. Configure Environment

```bash
cp .env.example .env
nano .env
```

Fill in:
- `GHOST_URL` — use your VPS IP for now: `http://YOUR_IP/blog`
- `MYSQL_ROOT_PASSWORD` — choose a strong password
- `MYSQL_PASSWORD` — choose a strong password

---

## 4. Start the Stack

```bash
docker compose up -d
```

Check everything is running:

```bash
docker compose ps
docker compose logs -f   # watch logs, Ctrl+C to exit
```

- Landing page: `http://YOUR_VPS_IP`
- Ghost blog: `http://YOUR_VPS_IP/blog`
- Ghost admin: `http://YOUR_VPS_IP/blog/ghost`

---

## 5. Initial Ghost Setup

Visit `http://YOUR_VPS_IP/blog/ghost` and complete the setup wizard.  
Create your admin account — use a strong password, this is public-facing.

---

## 6. Add Your Photo

Place your photo at `landing/assets/christian-baker.jpg`, then in `index.html` replace:

```html
<div class="photo-placeholder">...</div>
```

with:

```html
<img src="assets/christian-baker.jpg" alt="Christian Baker" />
```

Nginx serves the `landing/` directory directly — no rebuild needed, just restart:

```bash
docker compose restart nginx
```

---

## 7. Add SSL with Let's Encrypt (when subdomain is live)

Once the DNS A record for `christianbakerblog.candyfairstudio.com` points to your VPS:

```bash
# Get certificate
docker compose run certbot certonly \
  --webroot \
  --webroot-path=/var/www/landing \
  -d christianbakerblog.candyfairstudio.com \
  --email your@email.com \
  --agree-tos

# Update Ghost URL in .env
GHOST_URL=https://christianbakerblog.candyfairstudio.com/blog

# Uncomment the HTTPS server block in nginx/default.conf

# Restart everything
docker compose down && docker compose up -d
```

Set up auto-renewal (add to crontab):
```bash
0 3 * * * docker compose run certbot renew --quiet && docker compose restart nginx
```

---

## 8. Useful Commands

```bash
# Stop everything
docker compose down

# Restart a single service
docker compose restart nginx

# View Ghost logs
docker compose logs ghost

# Backup Ghost content
docker run --rm -v blog_ghost-content:/data -v $(pwd):/backup \
  alpine tar czf /backup/ghost-backup-$(date +%Y%m%d).tar.gz /data

# Update Ghost to latest 5.x
docker compose pull ghost
docker compose up -d ghost
```

---

## Updating the Landing Page

Edit `landing/index.html` and/or `landing/style.css`, then:

```bash
docker compose restart nginx
```

No build step required — Nginx serves the files directly.

---

## Connecting to the Main Site

The main site at [candyfairstudio.com](https://candyfairstudio.com) links to this page.
When the subdomain DNS is configured, update `GHOST_URL` in `.env` and
uncomment the HTTPS block in `nginx/default.conf`.

---

## Tech Notes

- Ghost content (posts, images, themes) is stored in a named Docker volume `blog_ghost-content`
- MySQL data is in `blog_mysql-data`
- Both volumes persist across `docker compose down` / `up` cycles
- Never commit `.env` — it contains database passwords
