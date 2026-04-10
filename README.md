# ERPNext v15 Deployment

A production-ready ERPNext v15 deployment on Ubuntu Linux, configured with MariaDB, Redis, Nginx, and the Frappe framework.

---

## Overview

ERPNext is an open-source, full-featured ERP system built on the Frappe framework. This deployment covers a complete single-server setup suitable for small-to-medium businesses, including web server configuration, database tuning, and process management.

---

## Stack

| Layer | Technology |
|---|---|
| OS | Ubuntu Linux (22.04 LTS) |
| Web Server | Nginx |
| Application Framework | Frappe v15 |
| ERP Application | ERPNext v15 |
| Database | MariaDB 10.11 |
| Cache / Queue | Redis |
| Runtime | Node.js 18 |
| Package Manager | Yarn |
| Process Manager | Supervisor / Bench |

---

## Prerequisites

- Ubuntu 22.04 LTS server (minimum 4GB RAM, 2 vCPUs recommended)
- A non-root sudo user
- Domain name pointed at your server IP (for Nginx + SSL)

---

## Installation

### 1. Update system and install dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl wget python3-dev python3-pip python3-venv \
  libffi-dev libssl-dev build-essential software-properties-common
```

### 2. Install MariaDB 10.11

```bash
curl -LsS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup \
  | sudo bash -s -- --mariadb-server-version=10.11

sudo apt install -y mariadb-server mariadb-client
sudo systemctl enable --now mariadb
sudo mysql_secure_installation
```

Configure MariaDB for Frappe — add the following to `/etc/mysql/mariadb.conf.d/50-server.cnf` under `[mysqld]`:

```ini
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
```

```bash
sudo systemctl restart mariadb
```

### 3. Install Redis

```bash
sudo apt install -y redis-server
sudo systemctl enable --now redis-server
```

### 4. Install Node.js 18 and Yarn

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g yarn
```

### 5. Install Nginx

```bash
sudo apt install -y nginx
sudo systemctl enable --now nginx
```

### 6. Install Frappe Bench

```bash
sudo pip3 install frappe-bench
```

### 7. Initialize Bench with Frappe v15

```bash
bench init --frappe-branch version-15 frappe-bench
cd frappe-bench
```

### 8. Create a new site

```bash
bench new-site your-domain.com --db-name erpnext_db
```

### 9. Install ERPNext v15

```bash
bench get-app --branch version-15 erpnext
bench --site your-domain.com install-app erpnext
```

### 10. Configure production mode

```bash
sudo bench setup production <your-linux-user>
sudo bench setup nginx
sudo supervisorctl reload
sudo nginx -t && sudo systemctl reload nginx
```

---

## Running the Application

### Development mode

```bash
cd frappe-bench
bench start
```

Access at `http://localhost:8000`

### Production mode

Production is managed by Supervisor and Nginx. Use these commands to manage services:

```bash
# Check status of all processes
sudo supervisorctl status

# Restart all bench workers
sudo supervisorctl restart all

# Reload Nginx after config changes
sudo systemctl reload nginx

# Run database migrations after updates
bench --site your-domain.com migrate
```

### Updating ERPNext

```bash
cd frappe-bench
bench update --pull --patch --build --restart
```

---

## SSL Setup (Let's Encrypt)

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
sudo certbot renew --dry-run   # verify auto-renewal
```

---

## Useful Bench Commands

| Command | Description |
|---|---|
| `bench start` | Start development server |
| `bench --site [site] migrate` | Run pending migrations |
| `bench --site [site] backup` | Create a site backup |
| `bench --site [site] restore [file]` | Restore from backup |
| `bench --site [site] console` | Open Python REPL for the site |
| `bench update` | Pull latest updates and rebuild |
| `bench setup requirements` | Reinstall Python/Node dependencies |

---

## Skills Demonstrated

- **Linux server administration** — package management, service configuration, user permissions
- **Database management** — MariaDB installation, charset configuration, secure setup
- **Web server configuration** — Nginx reverse proxy setup, SSL termination with Let's Encrypt
- **Application deployment** — Frappe Bench workflow, multi-app site management
- **Process management** — Supervisor for managing background workers and queues
- **Node.js ecosystem** — Node 18 runtime, Yarn for frontend asset builds
- **Open-source ERP** — ERPNext v15 installation, site creation, app management
- **DevOps fundamentals** — production hardening, update workflows, backup and restore
