tags: [devops, load-balancing]

## Load Balancing — L4 vs L7, алгоритмы, healthchecks, HAProxy/Nginx

> ⚡ Tip: L4 балансирует по IP/порту (TCP/UDP), L7 — понимает протокол (HTTP/HTTPS/gRPC) и может маршрутизировать по URL/Host/Headers.

### Алгоритмы балансировки

- **roundrobin** — по кругу (дефолт во многих LB);
- **leastconn** — наименьшее количество активных соединений;
- **ip_hash / sticky** — привязка клиента к одному бэкенду;
- **random** — случайный выбор.

### HAProxy (L4/L7 пример)

`haproxy.cfg` (фрагмент):

```cfg
frontend http_in
  bind *:80
  mode http
  default_backend servers

backend servers
  mode http
  balance roundrobin
  option httpchk GET /health
  server s1 10.0.0.10:8080 check
  server s2 10.0.0.11:8080 check
```

L4‑пример:

```cfg
frontend tcp_in
  bind *:5432
  mode tcp
  default_backend db_servers

backend db_servers
  mode tcp
  balance leastconn
  server db1 10.0.0.20:5432 check
  server db2 10.0.0.21:5432 check backup
```

### Nginx (L7)

```nginx
upstream api_backend {
  least_conn;
  server 10.0.0.10:8080 max_fails=3 fail_timeout=30s;
  server 10.0.0.11:8080 max_fails=3 fail_timeout=30s;
}

server {
  listen 80;
  server_name api.example.com;

  location / {
    proxy_pass http://api_backend;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $remote_addr;
  }
}
```

### Healthchecks

- TCP (L4): проверка установления соединения;
- HTTP (L7): `/health`, `/ready`, `/live`.

Best practice:

- разделять liveness/readiness (`[[04_Kubernetes/workloads]]`);
- быстрые и лёгкие health endpoints.

Связанные: `[[09_Cloud/cloud-networking]]`, `[[11_Networking/service-mesh]]`, `[[10_Security/network-security]]`.

## Gotchas

- **Отсутствие healthchecks или слишком агрессивные тайм‑ауты**  
  - **Проблема**: LB держит мёртвые бэкенды или, наоборот, постоянно их выкидывает.  
  - **Решение**: настроить адекватные интервалы, тайм‑ауты и пороги ошибок.

- **Sticky‑сессии без понимания последствий**  
  - **Проблема**: неравномерное распределение нагрузки, проблемы при scale‑down.  
  - **Решение**: по возможности делать приложения stateless, использовать shared‑store/Redis для сессий.

- **Смешивание HTTP/HTTPS логики на L4 LB**  
  - **Проблема**: невозможность терминировать TLS/вешать WAF на чистом L4.  
  - **Решение**: явная архитектура: L4 для TCP/UDP, L7 для HTTP/gRPC с TLS‑терминацией и WAF.

> ⚠️ Warning: Изменения конфигурации балансировщика (особенно в managed LB) без чёткого понимания healthcheck'ов и backend‑политик могут мгновенно вывести из строя весь входящий трафик. Любые новые правила/слои (WAF, CDN) тестируйте поэтапно (canary, shadow traffic).

