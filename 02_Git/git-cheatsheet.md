tags: [devops, git]

## Git Cheatsheet — основные команды

> ⚡ Tip: Почти все команды Git поддерживают детальный вывод через флаг `-v`/`--verbose`. Используйте его при отладке.

### Настройка пользователя и репозитория

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global core.editor "vim"              # редактор по умолчанию
git config --global init.defaultBranch main        # ветка по умолчанию

git config --list                                  # показать конфиг
git config --show-origin --list                    # с указанием файлов
```

### Создание и клон репозитория

```bash
git init                                           # инициализировать новый репозиторий
git init --bare /srv/git/repo.git                  # bare-репо на сервере

git clone git@github.com:user/repo.git             # клон по SSH
git clone https://github.com/user/repo.git         # клон по HTTPS
git clone --depth 1 git@github.com:user/repo.git   # поверхностный клон
```

### Статус, diff, лог

```bash
git status -sb                                     # краткий статус (ветка + изменения)
git status --ignored                               # показывать игнорируемые файлы

git diff                                           # изменения в рабочей копии
git diff --cached                                  # изменения в индексе (staged)
git diff HEAD~1                                    # diff с предыдущим коммитом

git log --oneline                                  # краткий лог
git log --graph --oneline --decorate --all         # история с графом
git log -p -n 1                                    # последний коммит с diff
git log --stat                                     # статистика изменений
```

### Индекс и коммиты

```bash
git add file1 file2                                # добавить файлы в индекс
git add .                                          # добавить все
git add -p                                         # интерактивно выбрать hunks

git commit -m "short message"                      # обычный коммит
git commit -am "msg"                               # добавить изменённые tracked файлы + commit
git commit --amend                                 # изменить последний коммит (не после push!)

git restore file                                   # откатить изменения в рабочей копии
git restore --staged file                          # убрать файл из индекса
```

### Ветки и теги

```bash
git branch                                         # список локальных веток
git branch -r                                      # список remote-веток
git branch -a                                      # все ветки
git branch feature/login                           # создать ветку

git checkout feature/login                         # переключиться
git checkout -b feature/login                      # создать и переключиться

git switch feature/login                           # альтернатива checkout
git switch -c feature/login                        # создать и переключиться

git tag                                            # список тегов
git tag v1.0.0                                     # создать тег
git tag -a v1.0.0 -m "Release 1.0.0"               # аннотированный тег
git push origin v1.0.0                             # отправить тег
git push origin --tags                             # отправить все теги
```

### Удалённые репозитории

```bash
git remote -v                                      # список remotes
git remote add origin git@github.com:user/repo.git # добавить origin
git remote set-url origin git@github.com:user/repo.git
git remote show origin                             # подробности
```

### Получение и отправка изменений

```bash
git fetch origin                                   # забрать изменения (без merge)
git fetch --all --prune                            # обновить все remotes + подчистить

git pull                                           # fetch + merge
git pull --rebase                                  # fetch + rebase

git push                                           # отправить текущую ветку
git push -u origin main                            # установить upstream
git push --force-with-lease                        # безопасный force-push
```

### Слияние и rebase

```bash
git merge feature/login                            # влить ветку в текущую
git merge --no-ff feature/login                    # создать merge-коммит

git rebase main                                    # «переписать» историю поверх main
git rebase -i HEAD~5                               # интерактивный rebase последних 5 коммитов
git rebase --abort                                 # отменить rebase
git rebase --continue                              # продолжить после решения конфликтов
```

### Конфликты merge/rebase

```bash
git status                                         # какие файлы в конфликте
git diff                                           # посмотреть конфликтующие фрагменты

# После ручного решения конфликтов:
git add <conflicted-files>
git commit                                         # для merge
git rebase --continue                              # для rebase
```

### Stash

```bash
git stash                                          # сохранить незакоммиченные изменения
git stash -u                                       # включая untracked файлы
git stash list                                     # список stash'ей
git stash show -p stash@{0}                        # diff конкретного stash
git stash apply stash@{0}                          # применить (оставить в списке)
git stash pop                                      # применить и удалить
git stash drop stash@{0}                           # удалить конкретный
git stash clear                                    # удалить все
```

### .gitignore и .gitattributes

```bash
cat .gitignore
*.log
node_modules/
.env
```

```bash
cat .gitattributes
*.sh text eol=lf
*.bat text eol=crlf
*.png binary
```

### Быстрая проверка перед пушем

```bash
git status -sb
git diff --cached
git log --oneline origin/main..HEAD   # какие коммиты уйдут на сервер
```

Смотрите также `[[branching-strategies]]` и `[[hooks-and-automation]]` для стратегий ветвления и автоматизации pre-commit.

## Gotchas

- **`git commit --amend` и переписанная история**  
  - **Проблема**: amend уже запушенного коммита меняет историю, ломая чужие ветки.  
  - **Решение**: не использовать `--amend` после `push`; вместо этого создавать новый коммит.

- **`git push --force` без `--force-with-lease`**  
  - **Проблема**: можно перезаписать чужую работу в удалённой ветке.  
  - **Решение**: использовать `git push --force-with-lease`, который проверяет, что remote‑ветка не изменилась.

- **Случайный коммит секретов (.env, ключи)**  
  - **Проблема**: секреты попадают в историю навсегда (считайте скомпрометированными).  
  - **Решение**: немедленно отозвать секреты, добавить их в `.gitignore`, рассмотреть `git filter-repo`/`BFG` для чистки истории. См. `[[secrets-management]]`.

- **Работа в `main`/`master` напрямую**  
  - **Проблема**: сложно делать код-ревью и откаты, высокие риски.  
  - **Решение**: работать в feature-ветках, использовать Pull/Merge Requests и понятную стратегию (`[[branching-strategies]]`).

> ⚠️ Warning: Любые операции, переписывающие историю (rebase, amend, reset --hard, push --force), на общих ветках требуют осознанного решения, резервной копии (например, через `reflog`) и коммуникации с командой.

