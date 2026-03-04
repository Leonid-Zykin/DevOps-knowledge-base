tags: [devops, bash]

## Shell scripting (bash)

### Шебанг и базовый каркас

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

> ⚡ Tip: `set -euo pipefail` + `IFS` — базовый шаблон для надёжных скриптов.

### Переменные и параметры

```bash
NAME="world"
echo "Hello, $NAME"

readonly APP_ENV="${APP_ENV:-prod}"          # значение по умолчанию

echo "Script name: $0"
echo "First arg: $1"
echo "Args count: $#"
echo "All args: $@"
echo "All args (one string): $*"
```

Безопасное использование параметров:

```bash
if [[ $# -lt 1 ]]; then
  echo "Usage: $0 <env>"
  exit 1
fi

ENV="$1"
```

### Условные конструкции

```bash
if [[ -f /etc/os-release ]]; then
  echo "Linux"
elif [[ "$OSTYPE" == "darwin"* ]]; then
  echo "macOS"
else
  echo "Unknown OS"
fi

if [[ "$ENV" == "prod" ]]; then
  echo "Deploy to prod"
fi
```

Проверка файлов:

```bash
[[ -f file ]]   # обычный файл
[[ -d dir ]]    # каталог
[[ -s file ]]   # размер > 0
[[ -x file ]]   # исполняемый
[[ -r file ]]   # читаемый
[[ -w file ]]   # записываемый
```

### Циклы

```bash
for i in {1..5}; do
  echo "i=$i"
done

for file in /var/log/*.log; do
  echo "Processing $file"
done

while read -r line; do
  echo "Line: $line"
done < file.txt
```

### Функции

```bash
deploy_service() {
  local name="$1"
  local version="${2:-latest}"     # значение по умолчанию

  echo "Deploy $name:$version"
  # ...
}

deploy_service "api" "1.2.3"
deploy_service "worker"           # version=latest
```

Возврат кода:

```bash
is_running() {
  local pid="$1"
  if kill -0 "$pid" 2>/dev/null; then
    return 0
  else
    return 1
  fi
}
```

### Массивы и ассоциативные массивы

```bash
numbers=(1 2 3 4)
echo "${numbers[0]}"
echo "${numbers[@]}"
echo "${#numbers[@]}"      # длина

for n in "${numbers[@]}"; do
  echo "n=$n"
done

declare -A COLORS=(
  ["prod"]="red"
  ["stage"]="yellow"
  ["dev"]="green"
)

echo "${COLORS[prod]}"
for env in "${!COLORS[@]}"; do
  echo "$env -> ${COLORS[$env]}"
done
```

### Обработка ошибок и traps

```bash
set -euo pipefail

cleanup() {
  local exit_code=$?
  echo "Cleaning up (exit=$exit_code)"
  rm -f /tmp/mytmpfile || true
}

trap cleanup EXIT
trap 'echo "Interrupted"; exit 130' INT
trap 'echo "Terminated"; exit 143' TERM
```

> ⚡ Tip: `trap EXIT` с `$?` — удобный способ логировать итоговый код выхода и чистить временные ресурсы.

### Подстановки и команды

```bash
DATE="$(date +%Y-%m-%d)"
HOSTNAME="$(hostname -s)"

OUT="$(command 2>&1)"       # сохранить stdout+stderr
RC="$?"                     # код возврата

echo "Command returned $RC, output: $OUT"
```

### Работа с файлами и каталогами в скриптах

```bash
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
ROOT_DIR="$(cd "$SCRIPT_DIR/.." && pwd)"

mkdir -p "$ROOT_DIR/logs"
LOG_FILE="$ROOT_DIR/logs/run-$(date +%Y%m%d-%H%M%S).log"

exec > >(tee -a "$LOG_FILE") 2>&1   # логировать всё в файл и stdout
```

### Аргументы и парсинг опций

```bash
usage() {
  echo "Usage: $0 -e <env> [-d]"
  exit 1
}

ENV=""
DEBUG=0

while getopts ":e:d" opt; do
  case "$opt" in
    e) ENV="$OPTARG" ;;
    d) DEBUG=1 ;;
    *) usage ;;
  esac
done

shift $((OPTIND - 1))

[[ -z "$ENV" ]] && usage

echo "ENV=$ENV, DEBUG=$DEBUG, rest args: $*"
```

### Примеры DevOps‑скриптов

**Проверка статуса сервисов:**

```bash
#!/usr/bin/env bash
set -euo pipefail

SERVICES=(nginx docker kubelet)

for s in "${SERVICES[@]}"; do
  if systemctl is-active --quiet "$s"; then
    echo "$s: OK"
  else
    echo "$s: FAILED"
  fi
done
```

**Ротация логов в каталоге:**

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG_DIR="/var/log/myapp"
MAX_AGE_DAYS=7

find "$LOG_DIR" -type f -name "*.log" -mtime +"$MAX_AGE_DAYS" -print -delete
```

Смотрите также `[[processes-and-services]]` и `[[linux-cheatsheet]]` для команд `systemctl`, `journalctl` и общей работы в shell.

## Gotchas

- **Отсутствие кавычек вокруг переменных**  
  - **Проблема**: пробелы, спецсимволы и пустые значения ломают скрипт (`rm -rf $DIR/*`).  
  - **Решение**: всегда оборачивать переменные в `"..."`: `rm -rf "$DIR"/*`.

- **`set -e` и команды, ожидающие ошибки**  
  - **Проблема**: `set -e` прерывает скрипт на любой ненулевой код, даже если это ожидаемо.  
  - **Решение**: явно гасить ошибки для таких команд: `cmd || true` или проверять `$?`.

- **Использование `/bin/sh` вместо bash‑специфичного синтаксиса**  
  - **Проблема**: массивы, `[[ ]]`, ассоциативные массивы не работают под `/bin/sh`.  
  - **Решение**: корректный шебанг `#!/usr/bin/env bash` и явное требование bash.

- **Неинициализированные переменные**  
  - **Проблема**: опечатка в имени переменной становится пустой строкой и ломает команды.  
  - **Решение**: использовать `set -u` и параметры по умолчанию: `"${VAR:-default}"`.

> ⚠️ Warning: Любые скрипты, работающие с `rm`, `chown`, `chmod`, сетевыми настройками или перезапуском сервисов, должны проходить ревью и тестироваться в non‑prod окружениях. Ошибка в одной строке может стоить целого кластера.

