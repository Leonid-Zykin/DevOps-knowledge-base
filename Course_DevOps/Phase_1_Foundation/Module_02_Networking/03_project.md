## Module 02 — Mini‑project: Nginx Reverse Proxy

### 1. Цель проекта

**Задача**: развернуть и настроить Nginx как reverse proxy перед простым приложением, используя только Linux‑утилиты и системные сервисы.

Вы разберётесь:

- как выглядит типичная схема «клиент → reverse proxy → backend»;
- как диагностировать проблемы на каждом шаге;
- как управлять Nginx как systemd‑сервисом.

Оценочное время: **3–5 часов**.

---

### 2. Архитектура мини‑проекта

Текстовая схема:

```text
Client (curl/browser)
        |
        v
 [Nginx reverse proxy]  :80
        |
        v
   [Backend app]        :8080
```

- Nginx слушает порт 80;
- все HTTP‑запросы проксируются на backend на 8080;
- backend — простой HTTP‑сервер (можно на Python/Node/Go — неважно, лишь бы был endpoint `/` и `/health`).

---

### 3. Требования к функционалу

1. **Backend**:
   - слушает на `127.0.0.1:8080`;
   - имеет endpoint `/` (возвращает любую страницу) и `/health` (возвращает простой OK/JSON).
2. **Nginx**:
   - слушает на `0.0.0.0:80`;
   - проксирует `/` на `http://127.0.0.1:8080/`;
   - корректно пробрасывает заголовки `Host` и `X-Forwarded-For`.
3. **Systemd**:
   - backend запущен как systemd‑сервис (опционально, но желательно);
   - Nginx управляется через systemd (стандартно).
4. **Диагностика**:
   - вы можете показать, что трафик реально идёт через Nginx;
   - умеете использовать `curl -v`, `ss`, `journalctl` для проверки.

---

### 4. Пошаговая инструкция

#### Шаг 1 — Подготовка backend‑приложения

В качестве backend можно использовать, например, Python:

```bash
mkdir -p ~/projects/reverse-proxy-demo/backend
cd ~/projects/reverse-proxy-demo/backend
```

Создайте `app.py` (пример на Flask или простой HTTPServer). Главное — чтобы он:

- слушал `127.0.0.1:8080`;
- отвечал на `/` и `/health`.

Запустите backend в foreground и проверьте:

```bash
curl http://127.0.0.1:8080/
curl http://127.0.0.1:8080/health
```

#### Шаг 2 — Оборачивание backend в systemd‑сервис (желательно)

Создайте unit‑файл, например `/etc/systemd/system/demo-backend.service`:

```ini
[Unit]
Description=Demo Backend HTTP Server
After=network.target

[Service]
User=YOUR_USER
WorkingDirectory=/home/YOUR_USER/projects/reverse-proxy-demo/backend
ExecStart=/usr/bin/python3 app.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Далее:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now demo-backend
sudo systemctl status demo-backend
curl http://127.0.0.1:8080/health
```

Если всё работает, backend‑часть готова.

#### Шаг 3 — Установка и базовая проверка Nginx

На Ubuntu:

```bash
sudo apt update
sudo apt install -y nginx
sudo systemctl status nginx
```

Проверьте:

```bash
curl -I http://127.0.0.1/
ss -tulpn | grep nginx
```

#### Шаг 4 — Настройка reverse proxy

Создайте новый конфиг, например `/etc/nginx/sites-available/reverse-proxy-demo`:

```nginx
server {
    listen 80;
    server_name _; # для локальных тестов

    access_log /var/log/nginx/reverse-proxy-access.log;
    error_log  /var/log/nginx/reverse-proxy-error.log;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Отключите дефолтный сайт и включите новый:

```bash
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/reverse-proxy-demo /etc/nginx/sites-enabled/reverse-proxy-demo
sudo nginx -t
sudo systemctl reload nginx
```

Проверьте:

```bash
curl -v http://127.0.0.1/
curl -v http://127.0.0.1/health
```

Вы должны увидеть ответы backend‑сервера.

#### Шаг 5 — Проверка с других машин (если есть)

Если у сервера есть внешний IP:

```bash
curl -v http://<server_public_ip>/
```

Убедитесь, что:

- порт 80 открыт в firewall/SG;
- backend не доступен напрямую снаружи (если вы этого не хотите).

---

### 5. Acceptance criteria

Мини‑проект считается завершённым, если:

1. **Backend**:
   - устойчиво слушает `127.0.0.1:8080`;
   - отдаёт разумный ответ на `/` и `/health`.
2. **Nginx**:
   - слушает на 80 порту;
   - `curl http://127.0.0.1/` возвращает backend‑страницу;
   - `curl http://127.0.0.1/health` возвращает health‑ответ backend’а.
3. **Диагностика**:
   - вы можете показать вывод `ss -tulpn | grep 80` и `ss -tulpn | grep 8080`;
   - можете показать основные строки доступа/ошибок в `/var/log/nginx/reverse-proxy-access.log` и `...-error.log`;
   - при остановке backend’а (`systemctl stop demo-backend`) запросы через Nginx начинают отдавать 502/504, и вы видите это в логах.
4. **Документация**:
   - у вас есть короткий README/заметка с:
     - командами запуска/остановки;
     - расположением конфигов;
     - примером curl‑запросов.

---

### 6. Stretch goals

1. **Отдельный upstream‑блок**  
   Вынесите backend в `upstream` и добавьте несколько «бэкендов» (пусть даже один и тот же хост) с алгоритмом `least_conn` (см. паттерны в `[[DevOps/11_Networking/load-balancing]]`).

2. **Health‑check’и Nginx → backend**  
   Используйте директивы `proxy_next_upstream` и `max_fails`/`fail_timeout`, чтобы Nginx не ходил в упавший backend.

3. **Базовая защита**  
   Добавьте лимит по размеру тела запроса и простейший rate‑limit (заодно свяжите с идеями из `[[DevOps/10_Security/network-security]]`).

4. **Анализ логов**  
   Попробуйте собрать базовую статистику по access‑логам с помощью `awk`/`goaccess` или отправить их в Loki/ELK (`[[DevOps/08_Monitoring/loki-and-logging]]`, `[[DevOps/08_Monitoring/elk-stack]]`).

---

### 7. Связь с дальнейшими модулями

- В Docker‑модуле вы обернёте такой стенд в контейнеры (Nginx + backend) и разберёте docker‑сети (`[[DevOps/03_Docker/networking-and-volumes]]`, `[[DevOps/03_Docker/docker-compose]]`).
- В Kubernetes‑модуле замените Nginx‑reverse‑proxy на Ingress Controller и Service (`[[DevOps/04_Kubernetes/networking]]`).
- В модулях по мониторингу и логированию научитесь снимать метрики и логи с такого reverse‑proxy стенда (`[[DevOps/08_Monitoring/prometheus]]`, `[[DevOps/08_Monitoring/grafana]]`, `[[DevOps/08_Monitoring/loki-and-logging]]`).

Если вы довели мини‑проект до рабочей схемы с диагностикой и stretch‑целями — вы уже близки к реальным задачам DevOps по настройке входящего трафика и проксей.

