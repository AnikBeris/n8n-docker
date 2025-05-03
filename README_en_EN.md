**------->**[English](/README_en_EN.md) | [Русский](/README.md) **<-------**

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="./media/logo-dark.png">
    <img alt="Project Logo" src="./media/logo-light.png" width="512" height="auto">
  </picture>
</p>

---

<div align="center">

[![GitHub](https://img.shields.io/badge/GitHub-blue?style=flat&logo=github)](https://github.com/AnikBeris)
[![License](https://img.shields.io/badge/License-purple?style=flat&logo=github)](https://github.com/AnikBeris/AutoRoleChannelBot/blob/main/LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/your-repo?style=flat&logo=github&label=Stars&color=orange)](https://github.com/AnikBeris)

</div>

# Technical Guide: Installing N8N in Docker

> **Disclaimer:** This guide is intended for personal use.

**If you find this project helpful, consider giving it a star!** :star2:

<p align="left">
  <a href="https://pay.cloudtips.ru/p/7249ba98" target="_blank">
    <img src="./media/buymeacoffe.png" alt="Support Image">
  </a>
</p>

Donations of any size are warmly appreciated. Thank you! 😌

| | |
|-------------:|:-------------|
| **Bitcoin (BTC)** | `1Dbwq9EP8YpF3SrLgag2EQwGASMSGLADbh` |
| **Ethereum (ERC20)** | `0x22258ea591966e830199d27dea7c542f31ed5dc5` |
| **Binance Smart Chain (BEP20)** | `0x22258ea591966e830199d27dea7c542f31ed5dc5` |
| **Solana (SOL)** | `yYYXsiVTzsvfvsMnBxfxSZEWTGytjAViE2ojf3hbLeF` |
| **Cloud Tips** | [cloudtips](https://pay.cloudtips.ru/p/7249ba98) |

---

![//](./media/n8n-image.png)

# 🚀 Installing n8n

This guide will help you deploy **n8n** on your server using Docker, configure it, and prepare it for use.

---

# Step 1: 📌 Install n8n via Docker Compose
---

- Create a new directory for **n8n** and move into it:

```bash
mkdir -p ~/n8n/{n8n_data,n8n_db,n8n_files,n8n_node_modules,project_db,project_files,project_photos} && cd ~/n8n
```

- Create a `.env ` file to store important configuration values

```bash
touch .env && nano .env
```

- Fill in the `.env` file:

```bash
# PostgreSQL settings – replace with your own credentials
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8nuser
DB_POSTGRESDB_PASSWORD=n8npass

# Connection settings
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=n8n-db
DB_POSTGRESDB_PORT=5432

# Main n8n configuration
N8N_HOST=your.domain.com
WEBHOOK_URL=https://your.domain.com
N8N_PORT=5678
N8N_PROTOCOL=https
NODE_ENV=production
N8N_ENCRYPTION_KEY=djwJNK/YUrhLagYsM72cYpu/JdR82a9BjlS3kAVA6ntBuQhAaYL6LhzE0Qan0jLE
GENERIC_TIMEZONE=Europe/Moscow
N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
N8N_SECURE_COOKIE=false
```

-🚀 Press Ctrl+D to save and exit.

<details> <summary>⚠️ Important:</summary>

- **N8N_ENCRYPTION_KEY** – generate a secure key with 48 characters using [base64](https://generate.plus/en/base64) generator. If this key is missing or invalid, your data may be lost after a restart or recovery.

![//](./media/Base_64_x48.png)

- **GENERIC_TIMEZONE** – sets internal time.

- **N8N_HOST** – your domain name for accessing n8n.

- **WEBHOOK_URL** – domain for internal n8n webhook processing.

</details>

Create `docker-compose.yml` with your deployment configuration

```bash
touch docker-compose.yml && nano docker-compose.yml

```

- Fill in `docker-compose.yml`:

```yaml
version: '3.8'

services:
  db:
    image: postgres:17
    container_name: n8n-DB
    hostname: n8n-db
    security_opt:
      - no-new-privileges:true
    healthcheck:
      test: ["CMD", "pg_isready", "-q", "-d", "${DB_POSTGRESDB_DATABASE}", "-U", "${DB_POSTGRESDB_USER}"]
      timeout: 45s
      interval: 10s
      retries: 10
    volumes:
      - ./n8n_db:/var/lib/postgresql/data:rw
    environment:
      TZ: ${GENERIC_TIMEZONE}
      POSTGRES_DB: ${DB_POSTGRESDB_DATABASE}
      POSTGRES_USER: ${DB_POSTGRESDB_USER}
      POSTGRES_PASSWORD: ${DB_POSTGRESDB_PASSWORD}
    restart: on-failure:5
    networks:
      - shared_network

  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    healthcheck:
      test: ["CMD-SHELL", "nc -z 127.0.0.1 5678 || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 90s
    hostname: n8n
    user: 0:0
    security_opt:
      - no-new-privileges:true
    ports:
      - 6789:5678
    volumes:
      - ./n8n_node_modules:/user/local/lib/node_modules:rw
      - ./n8n_data:/home/node/.n8n:rw
      - ./n8n_files:/files:rw
      - ./project_photos:/data/photos:rw
      - ./project_files:/data/files:rw
      - ./project_db:/data/db:rw
    environment:
      N8N_HOST: ${N8N_HOST}
      WEBHOOK_URL: ${WEBHOOK_URL}
      N8N_PORT: ${N8N_PORT}
      N8N_PROTOCOL: ${N8N_PROTOCOL}
      NODE_ENV: ${NODE_ENV}
      N8N_ENCRYPTION_KEY: ${N8N_ENCRYPTION_KEY}
      GENERIC_TIMEZONE: ${GENERIC_TIMEZONE}
      N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS: ${N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS}
      N8N_SECURE_COOKIE: ${N8N_SECURE_COOKIE}
      TZ: ${GENERIC_TIMEZONE}
      DB_TYPE: ${DB_TYPE}
      DB_POSTGRESDB_DATABASE: ${DB_POSTGRESDB_DATABASE}
      DB_POSTGRESDB_HOST: ${DB_POSTGRESDB_HOST}
      DB_POSTGRESDB_PORT: ${DB_POSTGRESDB_PORT}
      DB_POSTGRESDB_USER: ${DB_POSTGRESDB_USER}
      DB_POSTGRESDB_PASSWORD: ${DB_POSTGRESDB_PASSWORD}
    restart: on-failure:5
    depends_on:
      db:
        condition: service_healthy
    networks:
      - shared_network

networks:
  shared_network:
    name: shared_network
    driver: bridge
```

- Press Ctrl+D to save and exit. 🚀

<details> <summary> ⚠️ Important: </summary>
Both n8n and the PostgreSQL database must be on the same local network.

```yaml
networks:
  shared_network:
    name: shared_network
    driver: bridge

```

</details>

- Now start n8n with:

```bash
docker-compose up -d
```


 🎉 🎉 🎉 
 n8n should now be available at: http://<your_server_ip_or_domain>:6789
Example: http://192.168.1.1:6789 or https://n8n.anikberis.com