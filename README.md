# Boilerplates

Набор готовых Docker Compose шаблонов для быстрого развёртывания
сервисов с соблюдением best practices (healthcheck, `.env`, лимиты
ресурсов, изолированные сети, персистентные тома).

## Доступные шаблоны

| Директория | Описание |
| --- | --- |
| [`postgresql18/`](postgresql18) | PostgreSQL 18 + опциональный pgAdmin |
| [`uptime-kuma-2/`](uptime-kuma-2) | Uptime Kuma 2.x — мониторинг доступности |
| [`grafana-alloy/server/`](grafana-alloy/server) | Grafana + центральный Grafana Alloy |
| [`grafana-alloy/agent/`](grafana-alloy/agent) | Grafana Alloy — агент для мониторимых хостов |

## Использование

В каждой директории:

```bash
cp .env.example .env
# отредактируйте .env
docker compose up -d
```

Подробности, схемы и рекомендации по безопасности — в README каждой
директории.
