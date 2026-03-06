## Module 03 — Git Quiz

Сначала отвечайте по памяти, затем сверяйтесь со:

- `[[DevOps/02_Git/git-cheatsheet]]`
- `[[DevOps/02_Git/branching-strategies]]`
- `[[DevOps/02_Git/hooks-and-automation]]`
- `[[DevOps/02_Git/troubleshooting]]`

---

### Вопрос 1 — HEAD и ветка

Что такое `HEAD` в Git?

a) Текущий коммит, на который указывает активная ветка  
b) Название основной ветки  
c) Список всех веток  
d) Указатель на первый коммит в репозитории

---

### Вопрос 2 — feature‑ветка

Какая последовательность команд создаст новую ветку `feature/login` **от текущей ветки** и сразу на неё переключится?

a) `git branch feature/login && git checkout feature/login`  
b) `git switch -c feature/login`  
c) `git new feature/login`  
d) `git merge feature/login`

---

### Вопрос 3 — merge vs rebase

Какое утверждение о `git merge` и `git rebase` верно?

a) `git merge` и `git rebase` делают одно и то же  
b) `git merge` переписывает историю, а `git rebase` её сохраняет  
c) `git rebase` переписывает историю ветки, а `git merge` создаёт новый merge‑коммит  
d) `git rebase` используется только для удалённых веток

---

### Вопрос 4 — reset / revert

Какой командой **безопаснее всего** отменить один конкретный коммит в общей ветке `main`, уже запушенной в remote?

a) `git reset --hard HEAD~1`  
b) `git revert <hash>`  
c) `git reset --soft HEAD~1`  
d) `git push --force`

---

### Вопрос 5 — reflog

Для чего используется `git reflog`?

a) Для показа только удалённых веток  
b) Для сохранения резервной копии репозитория  
c) Для просмотра истории перемещений HEAD и восстановления «потерянных» коммитов  
d) Для удаления старых коммитов

---

### Вопрос 6 — stash

Что делает команда:

```bash
git stash -u
```

a) Сохраняет только изменения в tracked‑файлах  
b) Сохраняет изменения в tracked‑файлах и удаляет untracked  
c) Сохраняет изменения в tracked‑файлах и добавляет untracked файлы в stash  
d) Удаляет все незакоммиченные изменения

---

### Вопрос 7 — git clean

Что будет делать команда:

```bash
git clean -fd
```

a) Ничего, это dry‑run  
b) Удалит все untracked файлы и директории (без игнорируемых)  
c) Удалит все файлы, включая закоммиченные  
d) Удалит только игнорируемые файлы

---

### Вопрос 8 — Git hooks

Где по умолчанию хранятся локальные Git hooks в репозитории?

a) `.git/hooks`  
b) `.githooks/`  
c) `.git/hooks.d/`  
d) `.gitconfig`

---

### Вопрос 9 — Отправка feature‑ветки

Вы создали новую ветку `feature/api` и сделали несколько коммитов. Какой командой корректнее всего первый раз отправить её на remote и связать с origin?

a) `git push`  
b) `git push origin`  
c) `git push -u origin feature/api`  
d) `git push --force`

---

### Вопрос 10 — Опасные команды

Какой набор команд **наиболее опасен**, если вы не уверены, что делаете, особенно на общей ветке?

a) `git status`, `git diff`  
b) `git reset --hard`, `git push --force`, `git clean -fdx`  
c) `git log`, `git branch`  
d) `git commit`, `git merge`

---

> [!note]- Answers
> **Вопрос 1**  
> a) Текущий коммит, на который указывает активная ветка.  
> `HEAD` — указатель на коммит (обычно через ветку).  
>
> **Вопрос 2**  
> b) `git switch -c feature/login`.  
> Это современный и короткий способ создания и переключения на ветку.  
>
> **Вопрос 3**  
> c) `git rebase` переписывает историю ветки, а `git merge` создаёт новый merge‑коммит.  
> См. сравнительный разбор в `[[DevOps/02_Git/branching-strategies]]`.  
>
> **Вопрос 4**  
> b) `git revert <hash>`.  
> `revert` создаёт новый коммит, не ломая историю общей ветки, в отличие от `reset --hard`. См. `[[DevOps/02_Git/troubleshooting]]`.  
>
> **Вопрос 5**  
> c) Для просмотра истории перемещений HEAD и восстановления «потерянных» коммитов.  
> Это ваш последний шанс вернуть изменения после неудачного reset/rebase.  
>
> **Вопрос 6**  
> c) Сохраняет изменения в tracked‑файлах и добавляет untracked файлы в stash.  
> По умолчанию `git stash` не включает untracked, флаг `-u` это меняет.  
>
> **Вопрос 7**  
> b) Удалит все untracked файлы и директории (без игнорируемых).  
> Флаг `-x` дополнительно удалял бы игнорируемые (`.gitignore`). См. предупреждения в `[[DevOps/02_Git/troubleshooting]]`.  
>
> **Вопрос 8**  
> a) `.git/hooks`.  
> Именно там Git ищет локальные hook‑скрипты по умолчанию (или в каталоге, указанном в `core.hooksPath`).  
>
> **Вопрос 9**  
> c) `git push -u origin feature/api`.  
> Флаг `-u` связывает локальную ветку с remote и упрощает последующие `git push`/`git pull`.  
>
> **Вопрос 10**  
> b) `git reset --hard`, `git push --force`, `git clean -fdx`.  
> Все они могут безвозвратно удалить данные/историю, если использовать без понимания. Обсуждаются в `[[DevOps/02_Git/troubleshooting]]`.  

