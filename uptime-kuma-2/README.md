# Uptime Kuma 2 — Docker Compose

Шаблон для развёртывания Uptime Kuma 2.x с healthcheck'ом, лимитами
ресурсов и персистентным хранилищем.

## Быстрый старт

```bash
cp .env.example .env
docker compose up -d
```

Откройте `http://<host>:3001` и создайте учётную запись администратора при
первом входе.

## Мониторинг Docker-контейнеров (опционально)

Чтобы Uptime Kuma мог создавать мониторы типа "Docker Container", нужно
примонтировать Docker socket. Раскомментируйте строку в
`docker-compose.yml`:

```yaml
- /var/run/docker.sock:/var/run/docker.sock:ro
```

⚠️ Это даёт контейнеру фактический доступ к Docker-демону хоста —
монтируйте, только если действительно нужно, и только read-only.

## Best practices, заложенные в шаблон

- Официальный встроенный healthcheck-скрипт образа (`extra/healthcheck`).
- Именованный том `uptime_kuma_data` вместо bind-mount.
- Ограничение размера логов и CPU/RAM лимиты.
- `no-new-privileges` и отдельная bridge-сеть.
- Docker socket не примонтирован по умолчанию (принцип наименьших
  привилегий).

## Обновление версии

Версия образа зафиксирована через `UPTIME_KUMA_VERSION` в `.env`
(по умолчанию `2` — последний минорный/патч-релиз ветки 2.x). Перед
обновлением мажорной версии сверьтесь с release notes проекта и сделайте
бэкап тома `uptime_kuma_data`.

```bash
docker run --rm -v uptime-kuma-2_uptime_kuma_data:/app/data -v "$PWD":/backup alpine \
  tar czf /backup/uptime-kuma-data-backup.tar.gz -C /app/data .
```
