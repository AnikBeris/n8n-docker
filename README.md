**------->** [English](/README_en_EN.md) | [Русский](/README.md) **<-------**

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
[![GitHub Stars](https://img.shields.io/github/stars/your-repo?style=flat&logo=github&label=Звёзды&color=orange)](https://github.com/AnikBeris)

</div>

# Техническое руководство по установке N8N в Docker.




> **Отказ от ответственности:** Всё преведенные материалы расчитаны на личное использование.

**Если этот проект оказался полезным для Вас, вы можете оценить его, поставив звёздочку.**:star2:

<p align="left">
  <a href="https://pay.cloudtips.ru/p/7249ba98" target="_blank">
    <img src="./media/buymeacoffe.png" alt="Image">
  </a>
</p>

Пожертвования горячо приветствуются, какими бы маленькими они ни были, и большое спасибо. 😌

| | |
|-------------:|:-------------|
| **Bitcoin (BTC)** |`1Dbwq9EP8YpF3SrLgag2EQwGASMSGLADbh`|
| **Ethereum (ERC20)** | `0x22258ea591966e830199d27dea7c542f31ed5dc5`|
| **Binance Smart Chain (BEP20)** | `0x22258ea591966e830199d27dea7c542f31ed5dc5`|
| **Solana (SOL)** | `yYYXsiVTzsvfvsMnBxfxSZEWTGytjAViE2ojf3hbLeF`|
| **Cloud tips** | [cloudtips](https://pay.cloudtips.ru/p/7249ba98) |
---

![//](./media/n8n-image.png)


# 🚀 Установка n8n


Этот гайд поможет вам развернуть **n8n** на своём сервере, настроить иподготовить для работы.

---

# Шаг 1: 📌 Создание .env файла и подготовка к установке
---

- Создаём новую директорию для **n8n** и переходим в неё:

```bash
mkdir -p ~/n8n/{n8n_data,n8n_db,n8n_files,n8n_node_modules,project_db,project_files,project_photos} && cd ~/n8n
```

## Создай `.env` файл где будем хранить выжная информация

```bash
touch .env && nano .env
```
- Заполняем фаил `.env`:

```bash
# настройки PostgreSQL логин и пароль замените на свой

DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8nuser
DB_POSTGRESDB_PASSWORD=n8npass

# настройки для подключения
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=n8n-db
DB_POSTGRESDB_PORT=5432


# Основные настройки n8n
N8N_HOST=your.domain.com
WEBHOOK_URL=https://your.domain.com
N8N_PORT=5678
N8N_PROTOCOL=https
NODE_ENV=production
N8N_ENCRYPTION_KEY=djwJNK/YUrhLagYsM72cYpu/JdR82a9BjlS3kAVA6ntBuQhAaYL6LhzE0Qan0jLE
GENERIC_TIMEZONE=Europe/Moscow
N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
N8N_SECURE_COOKIE=false  # true если хотите подключение только по доменному имени / если включить потеряется возможность подключения по IP, останется только по доменному имени.
```
- После вставки нажми `Ctrl+D`, чтобы завершить ввод. 🚀


<details>
    <summary>⚠️ Важно: </summary>
 
- **N8N_ENCRYPTION_KEY=** <---- ключ можно сгенерировать на сайте установив количество симвалов `48`[base64](https://generate.plus/en/base64). Если ключ будет не верен или поле будет пустым, то при востановлении файло или перезагрузку, все файлы будут утерены.
![//](./media/Base_64_x48.png)

- **GENERIC_TIMEZONE=** <---- Выставляем внутренее время
- **N8N_HOST=** <---- Доменное имя для входа в **n8n**
- **WEBHOOK_URL=** <---- Доменное имя для работы **WEBHOOK** внутри **n8n**

</details>

---

# Шаг 2: 📌 Создание docker-compose.yml и установка n8n

## Создаём файлик `docker-compose.yml`

```bash
touch docker-compose.yml && nano docker-compose.yml
```
## Заполняем установочный фаил `docker-compose.yml`:

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
    image: n8nio/n8n:latest # :latest последняя версия // :stable стабильная версия // :beta версия в разработке
    container_name: n8n
    healthcheck:
      test: ["CMD-SHELL", "nc -z 127.0.0.1 5678 || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 90s
    hostname: n8n
    user: 0:0 # нужно для устранения проблем внутри контейнера
    security_opt:
      - no-new-privileges:true
    ports:
      - 6789:5678 # порты можно переназначать слева порт сервера через него выходим в сеть, справа порт контейнера. Меняем только порт сервера до двоиточия --> 6789:5678 <--этот не трогаем (пример: - 1231:5678)
    volumes:
      - ./n8n_node_modules:/user/local/lib/node_modules:rw
      - ./n8n_data:/home/node/.n8n:rw
      # рабочии папки Слева то что лежит на сервере, справа то что будет размещено в контейнере
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
- После вставки нажми `Ctrl+D`, чтобы завершить ввод. 🚀

<details>
    <summary> ⚠️ Важно:  </summary>

**n8n** и **база postgres** должны находиться в одной локальной сети.
```yaml
    networks:
      - shared_network

networks:
  shared_network:
    name: shared_network
    driver: bridge
```
</details>

# Шаг 3: 📌 запустим **n8n** командой:

```sh
docker-compose up -d
```

🎉🎉🎉 Теперь **n8n** доступна по адресу: `http://<адрес_сервера_на_котором_запущен_Gitea>:6789`

пример -> `http://192.168.1.1:6789`
пример -> `n8n.anikberis.com`

---


## Лицензия
Этот проект распространяется по [MIT License](https://github.com/your-repo/blob/main/LICENSE).



## Лицензия
Этот проект распространяется по [MIT License](https://github.com/your-repo/blob/main/LICENSE).

---

Для детальной документации ознакомьтесь с [Английским README](/README.md) или [Русским README](/README.ru_RU.md).
