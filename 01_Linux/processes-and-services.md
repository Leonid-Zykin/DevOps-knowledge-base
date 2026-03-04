tags: [devops, linux-processes]

## Процессы и сервисы (systemd, journalctl, cron)

### Просмотр процессов

```bash
ps aux                                  # все процессы
ps aux | grep nginx                     # отфильтровать по имени
ps -eo pid,ppid,user,cmd,%cpu,%mem --sort=-%cpu | head   # top по CPU
ps -eo pid,cmd,%mem --sort=-%mem | head                  # top по памяти

pgrep nginx                             # только PID'ы по имени
pkill -f "gunicorn: master"             # послать SIGTERM по шаблону команды
```

### Сигналы (kill)

```bash
kill <PID>                              # по умолчанию SIGTERM (15)
kill -9 <PID>                           # SIGKILL: немедленное убийство
kill -HUP <PID>                         # перезапуск/релоад конфигов (часто для демонов)
kill -TSTP <PID>                        # остановить (Ctrl+Z)
kill -CONT <PID>                        # продолжить выполнение
```

Таблица часто используемых сигналов:

| Сигнал | Номер | Значение                 |
|--------|-------|--------------------------|
| HUP    | 1     | перезапуск/перечитать   |
| INT    | 2     | прерывание (Ctrl+C)     |
| TERM   | 15    | корректное завершение   |
| KILL   | 9     | немедленное убийство    |
| TSTP   | 20    | остановка (Ctrl+Z)      |

### Приоритеты (nice/renice)

```bash
nice -n 10 cmd                           # запустить с пониженным приоритетом
renice 10 -p <PID>                       # изменить nice для процесса
```

### Фоновые задачи (jobs, nohup, tmux)

```bash
cmd &                                    # отправить в фон
jobs -l                                  # список фоновых задач
fg %1                                    # вернуть job 1 на передний план
bg %1                                    # продолжить job 1 в фоне

nohup cmd > cmd.log 2>&1 &               # пережить logout

tmux new -s deploy                       # новая tmux-сессия
tmux attach -t deploy                    # подключиться
tmux ls                                  # список сессий
```

### systemd: управление сервисами

```bash
systemctl status nginx                   # статус сервиса
systemctl start nginx                    # запустить
systemctl stop nginx                     # остановить
systemctl restart nginx                  # перезапустить
systemctl reload nginx                   # перечитать конфиг (если поддерживается)
systemctl enable nginx                   # включить автозапуск
systemctl disable nginx                  # выключить автозапуск
systemctl is-enabled nginx               # проверить автозапуск

systemctl list-units --type=service      # все сервисы
systemctl list-unit-files --type=service # установленные unit-файлы
```

### systemd: unit‑файлы

```bash
systemctl cat nginx                      # показать unit и дроп-ин

mkdir -p /etc/systemd/system/myapp.service.d
cat >/etc/systemd/system/myapp.service <<'EOF'
[Unit]
Description=My App
After=network.target

[Service]
User=appuser
Group=app
WorkingDirectory=/srv/myapp
ExecStart=/usr/bin/python3 app.py
Restart=on-failure
RestartSec=5
EnvironmentFile=/etc/myapp.env
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload                  # перечитать unit‑файлы
systemctl enable --now myapp             # включить и запустить
```

> ⚡ Tip: Для мелких override'ов используйте `systemctl edit myapp` вместо правки основного unit‑файла — это упростит обновления пакетов.

### journalctl: просмотр логов systemd

```bash
journalctl -u nginx                       # логи сервиса nginx
journalctl -u nginx -f                    # follow логов (аналог tail -f)
journalctl -u nginx --since "10 min ago"  # последние 10 минут
journalctl -u nginx --since today         # с начала дня
journalctl -u nginx -n 200                # последние 200 строк

journalctl -xe                            # последние ошибки с деталями
journalctl -b -1                          # логи предыдущей загрузки

journalctl -u myapp.service -o short-iso  # удобный ISO-формат времени
journalctl -u myapp.service -g "ERROR"    # фильтр по строке
```

### cron: периодические задачи

```bash
crontab -e                                # редактировать cron текущего пользователя
crontab -l                                # список задач
sudo crontab -e -u root                   # cron для root

ls /etc/cron.d                            # системные cron‑jobs
ls /etc/cron.daily                        # ежедневные задачи
```

Формат записи `crontab`:

```text
* * * * * user command
┬ ┬ ┬ ┬ ┬
│ │ │ │ └─ день недели (0-7, 0/7 = воскресенье)
│ │ │ └─── месяц (1-12)
│ │ └───── день месяца (1-31)
│ └─────── час (0-23)
└───────── минута (0-59)
```

Примеры:

```bash
*/5 * * * * /usr/local/bin/healthcheck.sh               # каждые 5 минут
0 3 * * * /usr/local/bin/backup.sh                      # ежедневно в 3:00
0 0 * * 0 /usr/local/bin/cleanup.sh                     # каждое воскресенье в 00:00
```

### at / systemd timers

```bash
echo "systemctl restart myapp" | at now + 10 minutes    # выполнить один раз
atq                                                    # очередь заданий
atrm <job_id>                                          # удалить задание
```

Systemd‑таймер (альтернатива cron):

```bash
cat >/etc/systemd/system/myjob.service <<'EOF'
[Unit]
Description=My periodic job

[Service]
Type=oneshot
ExecStart=/usr/local/bin/myjob.sh
EOF

cat >/etc/systemd/system/myjob.timer <<'EOF'
[Unit]
Description=Run myjob every 5 minutes

[Timer]
OnBootSec=5min
OnUnitActiveSec=5min
Unit=myjob.service

[Install]
WantedBy=timers.target
EOF

systemctl daemon-reload
systemctl enable --now myjob.timer
systemctl list-timers
```

### Отладка зависаний и утечек

```bash
top / htop                                  # посмотреть, что ест CPU/память
ps aux | sort -rk 3,3 | head                # top по CPU
ps aux | sort -rk 4,4 | head                # top по памяти

strace -p <PID>                             # отладка системных вызовов
gdb -p <PID>                                # дебагер (если установлен)

ls /proc/<PID>                              # информация о процессе
cat /proc/<PID>/stack                       # стек ядра
cat /proc/<PID>/status                      # статус, лимиты, флаги
```

Смотрите также `[[linux-cheatsheet]]` и `[[networking]]` для общей диагностики и сетевых проблем.

## Gotchas

- **`kill -9` вместо корректного завершения**  
  - **Проблема**: игнорирует очистку ресурсов (lock‑файлы, соединения, tmp‑файлы).  
  - **Решение**: сначала пробовать `kill` (SIGTERM), `systemctl stop`; `-9` — только когда всё остальное не сработало.

- **Забытый `daemon-reload`**  
  - **Проблема**: после правки unit‑файла systemd продолжает использовать старую версию.  
  - **Решение**: всегда выполнять `systemctl daemon-reload` после изменений и только затем `restart`.

- **Cron с другой средой окружения**  
  - **Проблема**: скрипт работает в shell, но не в cron (нет PATH, переменных, pyenv и т.п.).  
  - **Решение**: в cron‑job'ах использовать полный путь к бинарям, явно задавать `PATH` и `HOME` в начале скрипта.

- **Дублирующиеся cron‑задания**  
  - **Проблема**: одна и та же задача заведена в `crontab` пользователя и в `/etc/cron.d`.  
  - **Решение**: централизовать управление cron'ом, документировать задания в `[[incident-response]]` или отдельной operational‑записке.

> ⚠️ Warning: Никогда не редактируйте `/etc/crontab` и unit‑файлы systemd напрямую без резервной копии и истории изменений (Git). Неправильная запись может остановить критичные бэкапы или автозапуск сервисов.

