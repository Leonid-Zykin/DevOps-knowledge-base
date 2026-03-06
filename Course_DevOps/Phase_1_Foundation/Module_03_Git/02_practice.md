## Module 03 — Git Practice

Все задания выполняйте в отдельном тестовом репозитории, чтобы ничего не сломать в реальных проектах.  
См. шпаргалки:

- `[[DevOps/02_Git/git-cheatsheet]]`
- `[[DevOps/02_Git/branching-strategies]]`
- `[[DevOps/02_Git/hooks-and-automation]]`
- `[[DevOps/02_Git/troubleshooting]]`

---

### Task 1 — Создание репозитория и первый коммит

- **Goal**: Пройти полный цикл init → commit → log.
- **Steps**:
  1. Создайте директорию `~/git-lab` и перейдите в неё.
  2. Выполните `git init` и настройте имя/почту (если ещё не).
  3. Создайте файл `README.md` с любым содержимым.
  4. Выполните:

```bash
git status -sb
git add README.md
git commit -m "chore: initial commit"
git log --oneline
```

- **Expected result**: Один коммит в истории с понятным сообщением.
- **How to verify**:
  - `git status -sb` показывает чистое дерево;
  - `git log --oneline` отображает ваш коммит.

---

### Task 2 — Работа с feature‑ветками

- **Goal**: Привыкнуть работать через отдельные ветки.
- **Steps**:
  1. Создайте ветку `feature/config` от `main`:

```bash
git switch -c feature/config
```

  2. Добавьте файл `config.yaml`, сделайте в нём 1–2 изменения с отдельными коммитами.
  3. Вернитесь в `main`, убедитесь, что там нет этих изменений.
- **Expected result**: Вы видите разницу между main и feature‑веткой.
- **How to verify**:
  - `git log --oneline --graph --decorate --all` показывает разные ветки;
  - `git diff main..feature/config` показывает изменения.

---

### Task 3 — merge и решение конфликта

- **Goal**: Отработать merge‑конфликт в безопасной среде.
- **Steps**:
  1. В ветке `main` создайте файл `note.txt` со строкой:

```text
Environment: dev
```

  2. Закоммитьте.
  3. В ветке `feature/config` измените ту же строку на:

```text
Environment: stage
```

  4. Вернитесь в `main` и попробуйте `git merge feature/config`.
  5. Разрешите конфликт руками, выбрав, например, `Environment: stage` и добавив комментарий.
  6. Завершите merge.
- **Expected result**: merge‑коммит с разрешённым конфликтом.
- **How to verify**:
  - `git status` чистый;
  - содержимое `note.txt` соответствует вашему решению;
  - `git log --oneline` показывает merge‑коммит.

---

### Task 4 — rebase локальной ветки

- **Goal**: Безопасно применить rebase для локальной ветки.
- **Steps**:
  1. Создайте новую ветку `feature/rebase-demo` от `main`.
  2. Сделайте 2–3 маленьких коммита (например, добавьте файлы `a.txt`, `b.txt`).
  3. Вернитесь в `main` и сделайте ещё один коммит, меняющий любой файл.
  4. Вернитесь в `feature/rebase-demo` и выполните:

```bash
git rebase main
```

  5. При необходимости решите конфликты и продолжите rebase.
- **Expected result**: История `feature/rebase-demo` «накатывается» поверх актуального main.
- **How to verify**:
  - `git log --oneline --graph --decorate --all` показывает линейную историю;
  - коммиты вашей feature идут после последних коммитов main.

> [!warning] Этот rebase делайте только до публикации ветки (ещё не было `git push`). После push используйте merge.

---

### Task 5 — Откат через git revert

- **Goal**: Научиться безопасно отменять коммиты в общей ветке.
- **Steps**:
  1. В ветке `main` сделайте коммит, который добавляет файл `bad-change.txt`.
  2. Представьте, что этот коммит уже в общей ветке (вообразите `git push`).
  3. Выполните:

```bash
git log --oneline  # найдите хеш коммита
git revert <hash>
```

  4. Посмотрите diff.
- **Expected result**: Появился новый коммит, отменяющий изменения `bad-change.txt`.
- **How to verify**:
  - `git status` чистый;
  - `git log --oneline` показывает новый revert‑коммит;
  - `git diff HEAD~1..HEAD` показывает отмену изменений.

---

### Task 6 — Восстановление потерянного коммита (reflog)

- **Goal**: Потренироваться доставать коммиты из «небытия».
- **Steps**:
  1. В отдельной ветке сделайте 1–2 коммита, удалите один из файлов.
  2. Выполните:

```bash
git reset --hard HEAD~1
```

  3. Убедитесь, что последний коммит «пропал».
  4. Посмотрите `git reflog`, найдите хеш старого HEAD.
  5. Выполните `git reset --hard <старый_HEAD>`.
- **Expected result**: Восстановленный коммит и файл.
- **How to verify**:
  - Файл снова существует;
  - `git log --oneline` показывает «потерянный» коммит.

См. `[[DevOps/02_Git/troubleshooting]]`.

---

### Task 7 — git stash в edge‑кейсе

- **Goal**: Понять поведение stash при переходе между ветками.
- **Steps**:
  1. В ветке `main` измените файл `README.md`, не коммитя изменения.
  2. Выполните:

```bash
git stash
git switch -c feature/stash-demo
git stash apply
```

  3. При необходимости решите конфликты.
- **Expected result**: Изменения успешно перенеслись в новую ветку.
- **How to verify**:
  - `git diff` показывает изменения в `feature/stash-demo`;
  - `git stash list` показывает, что stash всё ещё есть (если использовали apply, а не pop).

---

### Task 8 — Простой pre-commit hook

- **Goal**: Добавить минимальную автоматизацию перед коммитом.
- **Steps**:
  1. Создайте каталог `.githooks` и настройте `core.hooksPath`:

```bash
mkdir -p .githooks
git config core.hooksPath .githooks
```

  2. Создайте `.githooks/pre-commit` со скриптом, который:
     - запрещает коммит, если в staged‑изменениях есть строки с `DEBUG` или `console.log`.
  3. Сделайте hook исполняемым (`chmod +x .githooks/pre-commit`).
  4. Попробуйте закоммитить файл с `console.log` и убедитесь, что hook срабатывает.
- **Expected result**: pre-commit блокирует нежелательные изменения.
- **How to verify**:
  - При наличии `console.log` в staged‑diff коммит не проходит;
  - при чистом diff коммит проходит.

Подробнее — `[[DevOps/02_Git/hooks-and-automation]]`.

---

### Task 9 — Имитация CI‑workflow локально

- **Goal**: Привыкнуть к идее «что уйдёт на CI/remote».
- **Steps**:
  1. Создайте скрипт `pre-push-check.sh`, который:
     - выполняет `git status -sb`;
     - показывает коммиты, которые уйдут на origin/main:

```bash
git log --oneline origin/main..HEAD
```

  2. Свяжите скрипт с `pre-push` hook через `.githooks/pre-push`.
  3. Смоделируйте ситуацию, когда в вашей ветке несколько коммитов, и посмотрите, что покажет hook.
- **Expected result**: Перед каждым push вы явно видите список коммитов.
- **How to verify**:
  - При `git push` скрипт выводит ожидаемую информацию;
  - вы можете объяснить, почему это полезно в реальных проектах.

---

### Task 10 — Break it & fix it: git clean / reset

- **Goal**: Осознать поведение разрушительных команд.
- **Steps**:
  1. В тестовом репозитории создайте несколько неотслеживаемых файлов и каталогов.
  2. Посмотрите, что покажет:

```bash
git status -sb
git clean -nfd
```

  3. Убедившись, что ничего важного нет, выполните `git clean -fd` и посмотрите результат.
  4. Аналогично потренируйтесь с:

```bash
git reset --hard HEAD
```

  5. Зафиксируйте в отдельном файле `GIT_DANGER_NOTES.md`, какие команды нельзя запускать без понимания последствий.
- **Expected result**: Осознанное отношение к `clean`, `reset --hard` и т.п.
- **How to verify**:
  - Можете назвать как минимум 3 «опасные» команды из `[[DevOps/02_Git/troubleshooting]]` и объяснить, когда их **не** следует применять.

---

## Итог

После выполнения практики вы должны:

- уверенно создавать и мёржить ветки;
- уметь разрешать merge‑конфликты;
- понимать разницу между reset/revert и использовать их по назначению;
- уметь восстановить потерянный коммит через reflog;
- настроить базовые git‑hooks.

Эти навыки критичны для дальнейших модулей: CI/CD, GitOps (`[[DevOps/05_CI-CD/argocd]]`), Terraform и Kubernetes, где любые ошибки в Git‑истории могут приводить к неправильным деплоям и сложным откатам.

