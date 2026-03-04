tags: [devops, linux]

## Linux Cheatsheet — Основные команды

> ⚡ Tip: Почти все команды поддерживают `--help` и man‑страницы: `man <cmd>` или `CMD --help`.

### Навигация и информация о системе

```bash
pwd                 # текущая директория
ls -lah             # список с правами, размерами и человекочитаемыми единицами
ls -lah /etc        # смотреть конфиги
du -sh *            # размер папок/файлов в текущем каталоге
df -hT              # свободное место по файловым системам
uname -a            # версия ядра и архитектура
cat /etc/os-release # информация о дистрибутиве
whoami              # текущий пользователь
id                  # uid/gid и группы
uptime              # аптайм и средняя нагрузка
free -h             # использование памяти
top                 # интерактивный мониторинг
htop                # более удобный top (если установлен)
```

### Поиск файлов и содержимого

```bash
find /var/log -type f -name "*.log"                 # найти все .log
find . -maxdepth 2 -type d -name ".git"             # искать каталоги .git
find . -type f -mtime -1                            # файлы, измененные за последние сутки
find . -size +100M                                  # файлы больше 100MB
find . -type f -name "*.log" -delete                # удалить все *.log (осторожно)
find . -type f -name "*.yml" -exec grep -Hn "port" {} \;

grep -R "ERROR" /var/log/                           # рекурсивный поиск строки
grep -Rni "timeout" .                               # регистр-независимый поиск, с номерами строк
grep -R --include="*.log" "OOMKilled" /var/log      # фильтр по маске
grep -oE "https?://[^ ]+" file.txt                  # вытащить все URL

rg "pattern" .                                      # ripgrep — быстрее grep (если установлен)
```

### awk — выбор и обработка полей

```bash
echo "a b c" | awk '{print $2}'                     # вывести второе поле
ps aux | awk '$3 > 50 {print $0}'                   # процессы с CPU > 50%
df -h | awk 'NR==1 || $5+0 > 80'                    # только заголовок и FS > 80%
awk -F: '{print $1,$3,$7}' /etc/passwd              # поля через разделитель ":"
awk '{sum+=$1} END {print sum}' numbers.txt         # сумма чисел в первом столбце
```

### sed — потоковое редактирование

```bash
sed -n '1,20p' file.log                             # вывести строки 1–20
sed -n '/ERROR/p' file.log                          # вывести только строки с ERROR
sed 's/error/ERROR/g' file.log > file_fixed.log     # заменить error → ERROR
sed -i 's/DEBUG/INFO/g' app.conf                    # inplace замена (linux sed)
sed -e '/^#/d' -e '/^$/d' app.conf                  # удалить комментарии и пустые строки
```

### Работа с текстом и потоками

```bash
cat file                                           # вывести файл
less -S file                                       # постраничный просмотр, без переноса
tail -n 100 file.log                               # последние 100 строк
tail -f file.log                                   # «живой» лог
head -n 20 file.log                                # первые 20 строк
wc -l file                                         # количество строк
sort file                                          # сортировка
sort -u file                                       # уникальные строки
uniq -c                                            # подсчитать повторы (ожидает отсортированный ввод)
tr -d '\r' < file > file.lf                        # убрать символы CR (Windows → Unix)
cut -d':' -f1,3 /etc/passwd                        # поля по разделителю
```

### Управление пользователями и группами

```bash
useradd -m -s /bin/bash deploy                      # создать пользователя deploy
passwd deploy                                       # задать пароль
usermod -aG docker deploy                           # добавить в группу docker
groupadd devops                                     # создать группу
id deploy                                           # посмотреть UID/GID и группы
sudo -l                                             # какие команды разрешены пользователю
visudo                                              # редактировать sudoers (через editor)
```

### Архивы и сжатие

```bash
tar -czf backup.tar.gz /etc                        # создать архив
tar -xzf backup.tar.gz -C /tmp                     # распаковать в /tmp
tar -tvf backup.tar.gz                             # список содержимого

zip -r logs.zip /var/log/myapp                     # zip архив
unzip logs.zip -d /tmp/logs                        # распаковать zip
```

### Сеть (быстрые команды)

```bash
ip a                                               # интерфейсы и адреса
ip r                                               # таблица маршрутизации
ss -tulpn                                          # слушающие сокеты
ping -c 4 8.8.8.8                                  # проверить доступность
curl -v https://example.com                        # отладка HTTP(S)
dig +short example.com                             # DNS‑резолв
traceroute example.com                             # маршрут до хоста
```

### Управление пакетами (apt/yum/dnf)

```bash
apt update && apt upgrade -y                       # обновление (Debian/Ubuntu)
apt install -y htop jq                             # установка пакетов
apt search nginx                                   # поиск пакета
apt show nginx                                     # инфо о пакете

yum install -y htop                                # установка (RHEL/CentOS)
dnf install -y htop                                # установка (новые RHEL/Fedora)
rpm -qa | grep nginx                               # список установленных пакетов
```

### Перенаправления и пайплайны

```bash
cmd > out.log                                      # stdout в файл (перезаписать)
cmd >> out.log                                     # stdout в файл (дописать)
cmd 2> err.log                                     # stderr в файл
cmd > all.log 2>&1                                 # stdout+stderr в один файл
cmd1 | cmd2 | cmd3                                 # пайплайн команд
```

### Время, планирование, тайминг

```bash
date                                               # текущая дата/время
TZ="UTC" date                                      # время в UTC
timedatectl                                        # настройки времени/таймзоны
time sleep 2                                       # измерить время выполнения команды
at 02:00                                           # выполнить команды один раз в указанное время
cron: crontab -e                                   # редактировать cron для текущего пользователя
```

### Разное, но часто нужное

```bash
history | tail -n 30                               # последние команды
history | grep ssh                                 # искать в истории
alias ll='ls -lah'                                 # короткий алиас
unalias ll                                         # удалить алиас
which nginx                                        # путь к бинарю
whereis nginx                                      # бинарь+ман‑страницы
strace -p <PID>                                    # системные вызовы процесса
lsof -p <PID>                                      # открытые файлы процесса
lsof -i :8080                                      # кто слушает порт 8080
```

> ⚡ Tip: Для продвинутого поиска используйте `[[networking]]`, `[[filesystem-and-permissions]]` и `[[processes-and-services]]` — там более глубокие разделы по сети, правам и процессам.

## Gotchas (Типичные подводные камни)

- **Перезапись файлов при `>`**  
  - **Проблема**: `cmd > file` без ожидания может стереть важный лог.  
  - **Решение**: сначала использовать `>>` или делать резервную копию: `cp file file.bak`.

- **`rm -rf` в неправильной директории**  
  - **Проблема**: удаление не того каталога.  
  - **Решение**: перед `rm -rf` всегда делайте `pwd` и/или используйте `ls` для проверки.

- **`sed -i` несовместимость между Linux/BSD**  
  - **Проблема**: на macOS синтаксис `sed -i` отличается.  
  - **Решение**: в кросс‑платформенных скриптах избегайте `-i` или проверяйте OS.

- **`find -delete` без проверки**  
  - **Проблема**: удаление большего числа файлов, чем планировалось.  
  - **Решение**: сначала запускать `find` без `-delete`, проверять результат, затем добавлять `-delete`.

- **Команды, требующие sudo**  
  - **Проблема**: неполные результаты или ошибки доступа.  
  - **Решение**: если команда работает со всей системой (`lsof`, `ss`, `journalctl`), запускать через `sudo`.

> ⚠️ Warning: Никогда не выполняйте команды вида `rm -rf /` или `rm -rf *` без точного понимания текущей директории и шаблонов. В проде — особенно опасно, используйте более безопасные варианты и бэкапы.

