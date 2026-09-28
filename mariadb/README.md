# MariaDB — Docker Compose

Готовый шаблон для быстрого развёртывания MariaDB с healthcheck'ами,
изолированной сетью, лимитами ресурсов и опциональным phpMyAdmin.

## Быстрый старт

```bash
cp .env.example .env
# отредактируйте .env — обязательно смените пароли
docker compose up -d
```

## Что внутри

- **mariadb** — `mariadb:11`, данные хранятся в именованном томе
  `mariadb_data`, healthcheck через встроенный `healthcheck.sh`
  (проверяет соединение и завершённую инициализацию InnoDB).
- **phpmyadmin** (опционально) — веб-интерфейс администрирования,
  включается через Docker Compose profile `tools`:

  ```bash
  docker compose --profile tools up -d
  ```

## Инициализационные скрипты

Файлы `.sql`/`.sh`, положенные в `./init`, будут выполнены один раз при
первом создании тома данных (см. `docker-entrypoint-initdb.d` в
[официальном образе](https://hub.docker.com/_/mariadb)).

## Best practices, заложенные в шаблон

- Обязательные переменные (`MARIADB_ROOT_PASSWORD`, `MARIADB_PASSWORD`) не
  имеют дефолтов "из коробки" — контейнер не запустится с пустым паролем.
- `utf8mb4`/`utf8mb4_unicode_ci` по умолчанию (полная поддержка Unicode,
  включая эмодзи), вместо устаревшего `latin1`/`utf8`.
- Healthcheck и `depends_on: condition: service_healthy` для зависимых
  сервисов.
- Именованные тома вместо bind-mount для данных БД.
- Ограничение размера логов (`json-file`, `max-size`/`max-file`).
- `no-new-privileges` и изолированная сеть `mariadb_net`.
- Лимиты CPU/RAM через `deploy.resources.limits`.
- Базовые тюнинг-параметры (`max-connections`,
  `innodb-buffer-pool-size`) вынесены в `.env` — подстройте под объём
  доступной памяти хоста (обычно ~70-80% от выделенной БД памяти).

## Backup

Простой пример бэкапа:

```bash
docker compose exec mariadb mariadb-dump -u root -p"$MARIADB_ROOT_PASSWORD" "$MARIADB_DATABASE" > backup.sql
```

Restore:

```bash
cat backup.sql | docker compose exec -T mariadb mariadb -u root -p"$MARIADB_ROOT_PASSWORD" "$MARIADB_DATABASE"
```
