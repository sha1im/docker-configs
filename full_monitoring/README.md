# Central system

Compose запускает полноценную систему мониторинга:

- `node_exporter` — сбор метрик с текущего сервера;
- `prometheus` — сбор и хранение метрик;
- `grafana` — визуализация метрик.

## Сервисы

| Сервис | Образ | Назначение | Порт |
|---|---|---|---|
| `node_exporter` | `prom/node-exporter:v1.11.1` | Метрики хоста | `9100` |
| `prometheus` | `prom/prometheus:v3.12.0` | Сбор метрик | `9090` |
| `grafana` | `grafana/grafana:13.0.2` | Дашборды | `3000` |

## Особенности `node_exporter`

`node_exporter` запущен с доступом к хосту:

```yaml
network_mode: host
pid: host
volumes:
  - /:/host:ro,rslave
command:
  - --path.rootfs=/host
```

Это нужно, чтобы контейнер собирал метрики именно с хостовой системы, а не только изолированного контейнера.

## extra_hosts

В сервисе `prometheus` используется `extra_hosts`:

```yaml
extra_hosts:
  - "host-address:host-gateway"
```

Это нужно, чтобы Prometheus из контейнера мог обращаться к node_exporter, запущенному на хосте в network_mode: host. В prometheus.yml центральный сервер указывается как host-address:9100.

## Переменные окружения

Для Grafana нужен файл `.env` рядом с compose-файлом:

```env
GF_LOGIN=admin
GF_PASSWORD=your_password
```

`.env` не хранить в публичном репозитории.

## Файлы

Ожидаемая структура:

```text
.
├── docker-compose.central.yml
├── .env
└── prometheus/
    └── prometheus.yml
```

## Сетевые нюансы

- `node_exporter` использует сеть хоста, поэтому порт `9100` открывается напрямую на хосте.
- `prometheus` пробрасывает порт `9090:9090`.
- `grafana` пробрасывает порт `3000:3000`.
- Docker может добавлять собственные правила `iptables`, из-за чего доступ к проброшенным портам не всегда полностью контролируется только через `ufw`.

Минимально нужные внешние доступы:

```text
3000/tcp  — Grafana
9090/tcp  — Prometheus, если нужен внешний доступ
9100/tcp  — Node Exporter, обычно только для Prometheus
```

## Запуск

```bash
docker compose up -d
```

## Остановка

```bash
docker compose down
```
