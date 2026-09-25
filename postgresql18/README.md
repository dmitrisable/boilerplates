# PostgreSQL 18 — Docker Compose

Готовый шаблон для быстрого развёртывания PostgreSQL 18 с healthcheck'ами,
изолированной сетью, лимитами ресурсов и опциональным pgAdmin.

## Быстрый старт

```bash
cp .env.example .env
# отредактируйте .env — обязательно смените пароли
docker compose up -d
```

## Что внутри

- **postgres** — `postgres:18-alpine`, данные хранятся в именованном томе
  `postgres_data`, healthcheck через `pg_isready`.
- **pgadmin** (опционально) — веб-интерфейс администрирования, включается
  через Docker Compose profile `tools`:

  ```bash
  docker compose --profile tools up -d
  ```

## Инициализационные скрипты

Файлы `.sql`/`.sh`, положенные в `./init`, будут выполнены один раз при
первом создании тома данных (см. `docker-entrypoint-initdb.d` в
[официальном образе](https://hub.docker.com/_/postgres)).

## Best practices, заложенные в шаблон

- Обязательные переменные (`POSTGRES_USER`, `POSTGRES_PASSWORD`) не имеют
  дефолтов "из коробки" — контейнер не запустится с пустым паролем.
- Healthcheck и `depends_on: condition: service_healthy` для зависимых
  сервисов.
- Именованные тома вместо bind-mount для данных БД.
- Ограничение размера логов (`json-file`, `max-size`/`max-file`).
- `no-new-privileges` и изолированная сеть `postgres_net`.
- Лимиты CPU/RAM через `deploy.resources.limits` (работают в `docker compose`
  начиная с Compose v2; в Swarm-режиме учитываются полностью).

## Backup

Простой пример бэкапа:

```bash
docker compose exec postgres pg_dump -U "$POSTGRES_USER" "$POSTGRES_DB" > backup.sql
```

Restore:

```bash
cat backup.sql | docker compose exec -T postgres psql -U "$POSTGRES_USER" -d "$POSTGRES_DB"
```
