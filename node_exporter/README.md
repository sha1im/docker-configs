# Node-exporter

Compose запускает только `node_exporter` для сбора метрик.

## Сервис

| Сервис | Образ | Назначение | Порт |
|---|---|---|---|
| `node_exporter` | `prom/node-exporter:v1.11.1` | Метрики хоста | `9100` |

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

Это нужно, чтобы контейнер видел метрики хостовой системы.

## Сетевые нюансы

- Используется `network_mode: host`.
- Порт `9100` открывается напрямую на хосте.
- Доступ к `9100/tcp` нужен серверу с Prometheus.
- Docker может добавлять собственные правила `iptables`, поэтому при использовании `ufw` стоит отдельно проверить фактическую доступность порта.

Минимально нужный внешний доступ:

```text
9100/tcp — только от central-server
```

## Запуск

```bash
docker compose up -d
```

## Остановка

```bash
docker compose down
```
