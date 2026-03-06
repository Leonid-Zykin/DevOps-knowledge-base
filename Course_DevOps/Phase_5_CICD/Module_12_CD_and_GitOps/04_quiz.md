## Module 12 — CD & GitOps Quiz

Ориентируйтесь на `[[DevOps/05_CI-CD/argocd]]`, `[[DevOps/05_CI-CD/concepts]]`.

---

### Вопрос 1 — GitOps

Что является «источником истины» в GitOps?

a) Kubernetes API  
b) Git-репозиторий с манифестами и конфигурацией  
c) Argo CD  
d) Docker registry

---

### Вопрос 2 — Argo CD Application

Что связывает Argo CD Application?

a) Только namespace  
b) Git repo + path ↔ cluster/namespace  
c) Только Helm chart  
d) Только Docker образ

---

### Вопрос 3 — Sync

Что делает Argo CD при Sync?

a) Пушит изменения в Git  
b) Приводит кластер к состоянию из Git  
c) Создаёт новый репозиторий  
d) Удаляет все ресурсы

---

### Вопрос 4 — prune

Что делает `prune: true` в syncPolicy?

a) Обновляет только образы  
b) Удаляет из кластера ресурсы, убранные из Git  
c) Откатывает все изменения  
d) prune не существует

---

### Вопрос 5 — selfHeal

Что делает `selfHeal: true`?

a) Лечит сломанные поды  
b) Откатывает ручные изменения в кластере (drift) к состоянию из Git  
c) Перезапускает приложение  
d) selfHeal не существует

---

### Вопрос 6 — Drift

Что такое drift в контексте GitOps?

a) Изменение ветки в Git  
b) Расхождение между состоянием в Git и состоянием в кластере (ручные изменения)  
c) Смена registry  
d) Обновление образа

---

### Вопрос 7 — App of Apps

Что такое App of Apps?

a) Несколько приложений в одном namespace  
b) Один root Application, который деплоит другие Applications  
c) Много экземпляров одного приложения  
d) App of Apps не используется

---

### Вопрос 8 — argocd app rollback

Что делает `argocd app rollback <app> <revision>`?

a) Удаляет приложение  
b) Откатывает приложение к указанной ревизии sync  
c) Откатывает Git к предыдущему коммиту  
d) Перезапускает sync

---

### Вопрос 9 — Секреты в GitOps

Как рекомендуется хранить секреты при GitOps?

a) В открытом виде в Git  
b) SealedSecrets, SOPS, External Secrets — не в открытом виде  
c) Только в Argo CD  
d) Секреты не нужны

---

### Вопрос 10 — CI и GitOps

Какую роль играет CI в GitOps-пайплайне?

a) CI не используется  
b) CI собирает образы и обновляет конфигурацию в Git (теги, values); Argo CD синхронизирует кластер  
c) CI напрямую деплоит в кластер  
d) CI только запускает тесты

---

> [!note]- Answers
> **1** — b) Git-репозиторий.  
> **2** — b) Git repo + path ↔ cluster/namespace.  
> **3** — b) Приводит кластер к состоянию из Git.  
> **4** — b) Удаляет ресурсы, убранные из Git.  
> **5** — b) Откатывает drift к состоянию из Git.  
> **6** — b) Расхождение Git ↔ кластер.  
> **7** — b) Root Application деплоит другие Applications.  
> **8** — b) Откат к указанной ревизии sync.  
> **9** — b) SealedSecrets, SOPS, External Secrets.  
> **10** — b) CI обновляет Git; Argo CD синхронизирует кластер.
