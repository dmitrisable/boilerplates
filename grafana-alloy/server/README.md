# Grafana + Alloy — центральный стек

## Быстрый старт

```bash
cp .env.example .env
# заполните REMOTE_WRITE_URL / LOKI_PUSH_URL и датасорсы Grafana
docker compose up -d
```

Grafana: `http://<host>:3000` (логин/пароль — из `GF_SECURITY_ADMIN_USER` /
`GF_SECURITY_ADMIN_PASSWORD`).

Alloy UI (граф компонентов, статус пайплайнов): `http://<host>:12345`.

## Что делает central Alloy

- Принимает метрики от агентов через `prometheus.receive_http` (порт
  `9999`, путь `/api/v1/metrics/write`).
- Принимает логи от агентов через Loki push API (`loki.source.api`, порт
  `3100`).
- Принимает OTLP (gRPC `4317` / HTTP `4318`) — на случай, если часть
  агентов шлёт данные через OpenTelemetry Collector/SDK.
- Собирает собственные метрики (self-monitoring).
- Пересылает всё во внешний backend (`REMOTE_WRITE_URL`, `LOKI_PUSH_URL`).

## Grafana

Датасорсы `Metrics` (Prometheus-совместимый) и `Logs` (Loki) провижинятся
автоматически из `config/grafana/provisioning/datasources/datasources.yml`
и берут URL/креды из переменных окружения контейнера Grafana.

Дашборды можно класть в
`config/grafana/provisioning/dashboards/json/*.json` — они подхватятся
автоматически (см. `dashboards.yml`).

## Best practices, заложенные в шаблон

- Явные healthcheck'и (`/-/ready` для Alloy, `/api/health` для Grafana) и
  `depends_on: condition: service_healthy`.
- Конфиг Alloy монтируется read-only, секреты — только через переменные
  окружения (`env()` в `.alloy`, `$__env{}` в provisioning Grafana), не
  зашиты в конфигах.
- `GF_USERS_ALLOW_SIGN_UP=false`, обязательный сложный пароль администратора.
- Лимиты CPU/RAM, ограничение размера логов контейнеров.
- `no-new-privileges`, отдельная bridge-сеть.
- Именованные тома для персистентности (`alloy_data`, `grafana_data`).

## Безопасность в продакшене

- Закройте порты `9999`/`3100`/`4317`/`4318` фаерволом так, чтобы они были
  доступны только агентам (VPN/приватная сеть/mTLS через reverse proxy).
- Поставьте перед Grafana и Alloy UI reverse proxy с TLS (например,
  Traefik/nginx) — сами по себе эти сервисы не терминируют HTTPS.
- Ротируйте пароли/токены basic auth для remote_write и Loki push.
