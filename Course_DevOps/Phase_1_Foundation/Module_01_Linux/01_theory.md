## Module 01 — Linux Basics & Scripting

### Почему это важно

Для DevOps‑инженера Linux — это «домашняя операционная система». Практически всё: Docker, Kubernetes, CI‑агенты, мониторинг, логирование и облачные сервисы — крутятся на Linux. Без уверенной работы в терминале, понимания процессов, прав и скриптов вы будете постоянно упираться в базовые проблемы.

> [!info] Откуда брать команды
> В этом модуле мы опираемся на шпаргалки:
> - `[[DevOps/01_Linux/linux-cheatsheet]]`
> - `[[DevOps/01_Linux/filesystem-and-permissions]]`
> - `[[DevOps/01_Linux/processes-and-services]]`
> - `[[DevOps/01_Linux/shell-scripting]]`

---

## 1. Модель «всё — файл» и файловая иерархия

- В Linux почти всё представлено как файлы/каталоги:
  - обычные файлы, директории;
  - сокеты, named pipes;
  - устройства `/dev/*`;
  - псевдо‑файловые системы `/proc`, `/sys`.

Ключевые директории:

- `/` — корень;
- `/etc` — конфигурация системы и сервисов;
- `/var` — изменяемые данные (логи, очереди, кэш);
- `/usr` — приложения и библиотеки;
- `/home` — домашние директории пользователей;
- `/tmp` — временные файлы.

Базовые команды (см. `[[DevOps/01_Linux/linux-cheatsheet]]`):

```bash
pwd
ls -lah
du -sh *
df -hT
```

---

## 2. Права, владельцы и ACL

Linux‑права — первый слой безопасности:

- Владелец (user), группа (group), остальные (other).
- Права: `r` (read), `w` (write), `x` (execute/enter).

Примеры:

```bash
ls -l
chmod 640 file      # rw-r-----
chmod 755 script.sh # rwxr-xr-x
chown app:app /srv/app
```

Расширенные ACL позволяют гибко раздавать доступ:

```bash
getfacl file
setfacl -m u:deploy:rwx file
```

Подробно — `[[DevOps/01_Linux/filesystem-and-permissions]]`.

> [!warning] Массовые chmod/chown
> Команды вида `chmod -R` и `chown -R` по крупным деревьям (`/`, `/var`, `/etc`) часто ломают систему. Всегда сначала делайте `find`/`ls`, чтобы увидеть, какие файлы попадут под операцию.

---

## 3. Пользователи, группы и sudo

DevOps‑инженер часто создаёт системных пользователей/группы для сервисов:

```bash
useradd -m -s /bin/bash deploy
passwd deploy
groupadd devops
usermod -aG devops deploy
id deploy
```

`sudo` контролирует, кому разрешено выполнять привилегированные команды:

```bash
sudo -l
sudo visudo
```

Хорошая практика — запускать сервисы под отдельными пользователями, а не под `root`.

---

## 4. Процессы, сервисы и systemd

### 4.1. Процессы и сигналы

Смотрим процессы:

```bash
ps aux
ps -eo pid,ppid,user,cmd,%cpu,%mem --sort=-%cpu | head
pgrep nginx
```

Управляем сигналами:

```bash
kill <PID>      # SIGTERM — аккуратное завершение
kill -9 <PID>   # SIGKILL — жёсткое убийство
kill -HUP <PID> # часто «перечитать конфиг»
```

См. `[[DevOps/01_Linux/processes-and-services]]`.

> [!warning] Не злоупотребляйте `kill -9`
> Сначала пробуйте корректное завершение (`systemctl stop`, `kill` без `-9`). SIGKILL не даёт приложению очистить ресурсы и может оставлять «мусор».

### 4.2. systemd и сервисы

`systemd` — основной init/сервис‑менеджер на современных дистрибутивах.

Базовые команды:

```bash
systemctl status nginx
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl enable nginx
systemctl list-units --type=service
```

Пример простого unit‑файла (`/etc/systemd/system/myapp.service`):

```ini
[Unit]
Description=My App
After=network.target

[Service]
User=app
Group=app
WorkingDirectory=/srv/myapp
ExecStart=/usr/bin/python3 app.py
Restart=on-failure
RestartSec=5
EnvironmentFile=/etc/myapp.env

[Install]
WantedBy=multi-user.target
```

После изменения:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
```

Логи сервиса:

```bash
journalctl -u myapp -f
```

Подробно — `[[DevOps/01_Linux/processes-and-services]]`.

---

## 5. Логи и диагностика

Логи — первое место, куда вы смотрите при проблемах.

- Системные логи:

```bash
ls -lah /var/log
journalctl -xe
journalctl -u nginx --since "10 min ago"
```

- Логи приложений:

```bash
tail -n 100 app.log
tail -f app.log
grep -R "ERROR" /var/log/myapp
```

Из `[[DevOps/01_Linux/linux-cheatsheet]]`:

```bash
less -S file.log
rg "pattern" .
```

---

## 6. Сеть на уровне хоста

Даже до Kubernetes/облака вы должны уметь диагностировать сеть на голом сервере:

```bash
ip a          # интерфейсы и IP
ip r          # маршруты
ss -tulpn     # открытые порты
ping -c 4 8.8.8.8
curl -v https://example.com
dig example.com
```

Для более глубокого разбора — `[[DevOps/01_Linux/networking]]` и `[[DevOps/11_Networking/troubleshooting]]`.

---

## 7. Автоматизация задач: cron и systemd timers

Периодические задачи:

```bash
crontab -e
crontab -l
ls /etc/cron.daily
```

Формат cron:

```text
* * * * * cmd
┬ ┬ ┬ ┬ ┬
│ │ │ │ └─ день недели
│ │ │ └─── месяц
│ │ └───── день месяца
│ └─────── час
└───────── минута
```

Пример:

```bash
*/5 * * * * /usr/local/bin/healthcheck.sh
```

Альтернатива — systemd timers (см. `[[DevOps/01_Linux/processes-and-services]]`), которые лучше логируются и интегрируются с unit‑файлами.

---

## 8. Bash‑скрипты: надёжный каркас

Шаблон из `[[DevOps/01_Linux/shell-scripting]]`:

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'

log() { echo "[$(date -Is)] $*"; }

main() {
  log "Script started"
  # ...
}

main "$@"
```

Ключевые моменты:

- `set -euo pipefail` — строгий режим:
  - `-e` — выход при ошибке;
  - `-u` — ошибка при обращении к неинициализированной переменной;
  - `-o pipefail` — пайплайн падает, если любая команда вернула не 0.
- Всегда **кавычки вокруг переменных**: `"${VAR}"`.
- Используйте функции и `local` переменные.

Пример функции:

```bash
deploy_service() {
  local name="$1"
  local version="${2:-latest}"
  echo "Deploy $name:$version"
}
```

---

## 9. Обработка ошибок и traps

Для скриптов, которые трогают файловую систему/сервисы, обязательна очистка:

```bash
cleanup() {
  local exit_code=$?
  echo "Cleaning up (exit=$exit_code)"
  rm -f /tmp/mytmpfile || true
}

trap cleanup EXIT
trap 'echo "Interrupted"; exit 130' INT
trap 'echo "Terminated"; exit 143' TERM
```

Такой паттерн позволит вам:

- гарантированно закрывать ресурсы (temp‑файлы, lock‑файлы);
- логировать код выхода.

См. `[[DevOps/01_Linux/shell-scripting]]`.

---

## 10. Логгирование и структура прод‑скриптов

DevOps‑скрипты должны быть:

- **идемпотентными** — повторный запуск не ломает состояние;
- **шумящими в stdout/stderr** — логи легко перенаправить в файл/систему логирования;
- **конфигурируемыми** через переменные окружения/флаги.

Пример структуры:

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
LOG_DIR="$SCRIPT_DIR/logs"
mkdir -p "$LOG_DIR"
LOG_FILE="$LOG_DIR/healthcheck-$(date +%Y%m%d-%H%M%S).log"

exec > >(tee -a "$LOG_FILE") 2>&1

main() {
  # тело скрипта
}

main "$@"
```

---

## 11. Связь с будущими модулями

- В Docker‑модуле вы будете часто использовать **bash** для entrypoint‑скриптов (`[[DevOps/03_Docker/dockerfile-best-practices]]`).
- В Kubernetes‑модулях вы будете анализировать **сервисы и логи** на нодах (`[[DevOps/04_Kubernetes/troubleshooting]]`).
- В Ansible/Terraform‑модулях вам пригодятся знания о правах, пользователях и systemd.

---

## Что Middle DevOps должен уметь по Linux (чек‑лист)

- **Файловая система и права**
  - [ ] Понимаю структуру каталогов Linux и назначение `/etc`, `/var`, `/usr`, `/home`, `/tmp`.
  - [ ] Умею читать и настраивать права и владельцев (`chmod`, `chown`, `umask`, ACL) — `[[DevOps/01_Linux/filesystem-and-permissions]]`.

- **Пользователи и безопасность**
  - [ ] Создаю и настраиваю пользователей/группы для сервисов.
  - [ ] Настраиваю `sudo` и понимаю риски работы под `root`.

- **Процессы и сервисы**
  - [ ] Уверенно использую `ps`, `top/htop`, `kill`, `strace`, `lsof`.
  - [ ] Пишу и изменяю systemd‑unit'ы, использую `systemctl`, `journalctl` — `[[DevOps/01_Linux/processes-and-services]]`.

- **Сеть**
  - [ ] Читаю сетевую конфигурацию через `ip a`, `ip r`, `ss`, `curl`, `dig` — `[[DevOps/01_Linux/networking]]`.

- **Автоматизация**
  - [ ] Пишу надёжные bash‑скрипты с `set -euo pipefail`, функциями, `trap`.
  - [ ] Использую cron/systemd timers для плановых задач.

Если вы можете честно поставить галочку каждому пункту — ваша Linux‑база уже на уровне, достаточном для уверенного перехода к Docker и Kubernetes.

