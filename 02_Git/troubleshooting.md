tags: [devops, git-troubleshooting]

## Git Troubleshooting — reset, revert, reflog, stash edge cases

> ⚡ Tip: Перед любыми разрушительными действиями (`reset --hard`, перепись истории) смотрите `git status`, `git log --oneline` и `git reflog`.

### Быстрый чек‑лист

1. Вы потеряли коммит → смотрите `git reflog`.
2. Надо откатить коммит, не ломая историю → `git revert`.
3. Надо «переписать» локальную историю до push → `git reset --soft/--mixed/--hard`.
4. Конфликты merge/rebase → `git status`, решаем руками, `add` + `continue`.

### reset: типы и отличие

```bash
git reset --soft <commit>   # только HEAD, изменения остаются staged
git reset --mixed <commit>  # HEAD + индекс, изменения остаются в рабочей копии (по умолчанию)
git reset --hard <commit>   # HEAD + индекс + рабочая копия → ВСЁ к состоянию коммита
```

Примеры:

```bash
git reset --soft HEAD~1     # «раскатать» последний коммит в staged
git reset HEAD~1            # убрать последний коммит, изменения в рабочей копии
git reset --hard HEAD~1     # полностью удалить последний коммит и изменения
```

> ⚠️ Warning: `reset --hard` без резервной ссылки (commit id, reflog) практически необратим, если коммит ещё не был запушен.

### revert: безопасный откат в общей истории

```bash
git revert <commit>         # создаёт новый коммит, отменяющий изменения указанного
git revert HEAD             # откат последнего коммита
git revert HEAD~3..HEAD     # серия revert'ов
```

Используйте `git revert`, если:

- коммиты уже **запушены** в общую ветку;
- нужно сохранить историю и не ломать чужие ветки.

### reflog: поиск «потерянных» коммитов

```bash
git reflog                  # история перемещений HEAD
git reflog show main        # reflog конкретной ветки
```

Пример восстановления после `reset --hard`:

```bash
git reflog
# найти хеш нужного состояния (например, abc1234)
git reset --hard abc1234
```

### Восстановление удалённой ветки

```bash
git branch -D feature/foo                # удалили локально
git reflog                               # найти последний HEAD ветки
git branch feature/foo <commit>          # пересоздать
```

Если ветка была запушена:

```bash
git checkout -b feature/foo origin/feature/foo
```

### Работа со stash в edge-кейсах

Список и просмотр:

```bash
git stash list
git stash show -p stash@{0}
```

Применение к другой ветке:

```bash
git checkout feature/bar
git stash apply stash@{0}      # может вызвать конфликты
```

Разделение stash по файлам:

```bash
git stash show -p stash@{0} > patch.diff
git apply patch.diff           # выборочно применить изменения
```

Удаление:

```bash
git stash drop stash@{0}
git stash clear
```

### Конфликты при merge/rebase

Общий паттерн:

```bash
git status                                # какие файлы конфликтуют
git diff                                  # посмотреть конфликт
# правим файлы руками, убираем <<<<<<<, =======, >>>>>>>
git add <files>

# для merge:
git commit

# для rebase:
git rebase --continue
```

Отмена операции:

```bash
git merge --abort
git rebase --abort
```

### Сброс локальных изменений к origin

**Полный откат ветки к состоянию origin/main:**

```bash
git fetch origin
git reset --hard origin/main
```

**Откат только untracked / ignored:**

```bash
git clean -fd                    # удалить untracked файлы и директории
git clean -fdx                   # включая игнорируемые (ОСТОРОЖНО)
```

> ⚠️ Warning: `git clean -fdx` удаляет ВСЁ неотслеживаемое, включая `node_modules`, кэши, локальные конфиги. Используйте только осознанно.

### Ситуации и решения

**1. Сделал commit в неправильную ветку**

```bash
# находимся в ветке feature/foo, но надо было в feature/bar
git branch feature/bar HEAD       # создать новую ветку с нужным коммитом
git checkout feature/foo
git reset --hard HEAD~1           # убрать коммит из текущей ветки
git checkout feature/bar          # продолжать работу тут
```

**2. Надо объединить несколько коммитов в один (squash)**

```bash
git rebase -i HEAD~3              # выбрать последние 3 коммита
# пометить вторые/третьи как "squash" или "fixup"
```

**3. Удалить последний коммит из удалённой ветки (с force push)**

```bash
git reset --hard HEAD~1
git push --force-with-lease       # только если вы понимаете последствия
```

### Связанные шпаргалки

- `[[git-cheatsheet]]` — базовые команды.
- `[[branching-strategies]]` — стратегии ветвления и работы с ветками.

## Gotchas

- **Сброс не той ветки**  
  - **Проблема**: `git reset --hard` в ветке `main` вместо локальной фичи.  
  - **Решение**: перед reset всегда проверять `git branch --show-current` и делать `git status`.

- **Очистка `stash` без проверки**  
  - **Проблема**: `git stash clear` без бэкапа → потеря несохранённых изменений.  
  - **Решение**: перед очисткой сохранять нужные изменения в патчи (`git stash show -p > patch.diff`) или коммиты.

- **Неправильный выбор между reset и revert**  
  - **Проблема**: reset на общих ветках ломает историю коллегам.  
  - **Решение**: на общих ветках используйте `revert`; `reset` — только до push или в личных ветках.

- **Слепой `push --force`**  
  - **Проблема**: перезапись удалённых изменений, сделанных коллегами.  
  - **Решение**: всегда использовать `--force-with-lease` и перед этим `git fetch && git log origin/branch..HEAD`.

> ⚠️ Warning: Все операции восстановления (через `reflog`, reset --hard, фильтрация истории) лучше сначала отрепетировать в отдельном временном клон‑репозитории, чтобы не испортить рабочий origin и чужие ветки.

