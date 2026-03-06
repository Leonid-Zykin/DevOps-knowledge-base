## Module 03 — Git for DevOps

### Почему это важно

Git — основа всей работы DevOps: хранение кода, инфраструктуры (Terraform, Ansible), Kubernetes‑манифестов, CI/CD конфигов. Без уверенного владения Git вы не сможете безопасно делать релизы, откаты и разбирать инциденты.

Мы опираемся на:

- `[[DevOps/02_Git/git-cheatsheet]]`
- `[[DevOps/02_Git/branching-strategies]]`
- `[[DevOps/02_Git/hooks-and-automation]]`
- `[[DevOps/02_Git/troubleshooting]]`

---

## 1. Модель данных Git

Git — это не просто «система версий файлов», а **граф коммитов**:

- каждый коммит — снимок (`tree`) и ссылка на родителя;
- ветка — всего лишь **указатель на коммит**;
- `HEAD` — текущий указатель (обычно на ветку).

Ключевые моменты:

- изменения хранятся **локально** до `push`;
- вы можете экспериментировать в локальных ветках без влияния на ремоут.

Полезные команды:

```bash
git status -sb
git log --oneline --graph --decorate --all
```

См. `[[DevOps/02_Git/git-cheatsheet]]`.

---

## 2. Базовый рабочий цикл

Типичный цикл:

1. `git clone` или `git pull` — получить свежие изменения.
2. Работа в ветке:
   - `git checkout -b feature/xyz` или `git switch -c feature/xyz`.
3. Изменения:
   - `git add` → `git commit`.
4. Публикация:
   - `git push -u origin feature/xyz`.
5. Code review:
   - Pull Request / Merge Request.

Из шпаргалки `[[DevOps/02_Git/git-cheatsheet]]`:

```bash
git add .
git commit -m "feat: add healthcheck"
git push -u origin feature/healthcheck
```

---

## 3. Ветки и стратегии ветвления

Основные объекты:

- локальные ветки: `git branch`, `git switch`;
- удалённые ветки: `git branch -r`, `origin/main`.

Стратегии (`[[DevOps/02_Git/branching-strategies]]`):

- **Trunk-based** (часто с GitHub Flow):
  - основная ветка `main`;
  - маленькие feature‑ветки от `main`;
  - частые merge через PR (часто squash).
- **Git Flow**:
  - `main` + `develop` + `feature/*` + `release/*` + `hotfix/*`;
  - подходит для релизов «волнами».

Для DevOps‑репозиториев чаще подходит **Trunk-based**:

- меньше долгоживущих веток;
- проще держать CI/CD и инфраструктуру в актуальном состоянии.

---

## 4. merge vs rebase

### merge

```bash
git checkout main
git merge feature/xyz
```

- сохраняет историю так, как она была (включая merge‑коммит);
- безопасен для общих веток;
- история может быть «шумной» от merge‑коммитов.

### rebase

```bash
git checkout feature/xyz
git rebase main
```

- переписывает историю feature‑ветки так, как будто она создавалась от актуального `main`;
- делает историю линейной;
- **нельзя** делать rebase ветки, которую уже используют другие (см. `[[DevOps/02_Git/branching-strategies]]` и `[[DevOps/02_Git/troubleshooting]]`).

> [!warning] Rebase на общих ветках
> После `git push` в общую ветку rebase/amend могут сломать историю коллегам. Используйте rebase в основном для **локальных feature‑веток** до их публикации.

---

## 5. Откаты: reset, revert, reflog

Из `[[DevOps/02_Git/troubleshooting]]`:

- `git reset` — изменить локальную историю (двигает HEAD);
- `git revert` — создать новый коммит, отменяющий изменения предыдущего;
- `git reflog` — показать историю перемежений HEAD (последний шанс спасти «потерянные» коммиты).

Примеры:

```bash
# раскатать последний коммит обратно в рабочую копию
git reset HEAD~1

# безопасно откатить коммит в общей ветке
git revert <commit>

# найти потерянный коммит
git reflog
git reset --hard <old_head>
```

---

## 6. Stash для временных изменений

Когда нужно временно отложить незакоммиченные изменения:

```bash
git stash          # без untracked
git stash -u       # включая untracked
git stash list
git stash show -p stash@{0}
git stash apply stash@{0}
git stash pop
```

Подробнее — `[[DevOps/02_Git/troubleshooting]]`.

---

## 7. Git hooks и автоматизация

Hooks позволяют запускать скрипты в ответ на события:

- `pre-commit` — перед коммитом;
- `pre-push` — перед `git push`;
- `commit-msg` — проверка сообщения коммита.

Рекомендуется использовать:

- форматирование/линтеры;
- проверку на «забытые» debug‑выводы;
- быстрые тесты.

Из `[[DevOps/02_Git/hooks-and-automation]]`:

```bash
mkdir -p .githooks
git config core.hooksPath .githooks
```

Пример `pre-commit`:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Running pre-commit checks..."

if git diff --cached | grep -E "console\.log|TODO"; then
  echo "Found debug prints or TODOs in staged changes."
  exit 1
fi
```

> [!tip] Важные проверки — в CI
> Hooks легко отключить. Критичные проверки (линтеры, тесты, secret‑scan) обязательно должны дублироваться в CI (`[[DevOps/05_CI-CD/gitlab-ci]]`, `[[DevOps/05_CI-CD/github-actions]]`).

---

## 8. Git и DevOps: что хранить в репозитории

DevOps‑инженер делает Git‑репозиторий источником правды:

- код приложений;
- инфраструктура: Terraform (`[[DevOps/06_Terraform/terraform-cheatsheet]]`), Ansible (`[[DevOps/07_Ansible/ansible-cheatsheet]]`);
- Kubernetes‑манифесты и Helm‑charts (`[[DevOps/04_Kubernetes/helm]]`);
- CI/CD конфиги (`.gitlab-ci.yml`, workflows, Jenkinsfile) — `[[DevOps/05_CI-CD/concepts]]`.

**Не хранить**:

- секреты `.env`, ключи, пароли — см. `[[DevOps/10_Security/secrets-management]]`;
- большие бинарные артефакты (образы, большие архивы) — хранить в registry/artefact storage.

---

## 9. Типичные практики в командах

- Рабочий стандарт:
  - feature‑ветки `feature/...`;
  - fix‑ветки `fix/...`;
  - squash‑merge или rebase‑merge в main;
  - правила для сообщений коммитов (напр., Conventional Commits).
- Pull/Merge Request:
  - обязательный review;
  - связка с issue/тикетом;
  - автоматический запуск CI.

Git‑часть тесно связана с модулем CI/CD (`[[DevOps/05_CI-CD/concepts]]`).

---

## Что Middle DevOps должен знать по Git (чек‑лист)

- **Базовые операции**
  - [ ] Уверенно использую `git status`, `git diff`, `git log`, `git add`, `git commit`, `git push/pull`.
  - [ ] Понимаю, что такое HEAD, ветка, origin/main.

- **Ветки и история**
  - [ ] Умею создавать и мёржить feature‑ветки (`git switch -c`, `git merge`).
  - [ ] Понимаю, когда использовать `merge`, а когда `rebase`, и не делаю rebase общих веток.

- **Откаты**
  - [ ] Умею откатывать коммиты в общей ветке через `git revert`.
  - [ ] Умею восстанавливать «потерянные» коммиты через `git reflog` и `git reset` (см. `[[DevOps/02_Git/troubleshooting]]`).

- **Automation**
  - [ ] Умею подключать и настраивать git‑hooks (`[[DevOps/02_Git/hooks-and-automation]]`).
  - [ ] Понимаю взаимосвязь Git и CI/CD (какие ветки/теги что деплоят).

Если вы уверенно ставите галочки по этому списку — готовы к интенсивной практике и мини‑проекту в рамках модуля.

