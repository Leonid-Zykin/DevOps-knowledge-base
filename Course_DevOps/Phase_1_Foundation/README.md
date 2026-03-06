## Phase 1 — Foundation (Weeks 1–3)

В этой фазе вы закладываете фундамент: **Linux**, **сетевые основы** и **Git**. Без этих навыков невозможно уверенно двигаться дальше к Docker, Kubernetes и CI/CD.

> [!info] Цель фазы
> К концу Phase 1 вы должны чувствовать себя комфортно в терминале, уметь диагностировать базовые сетевые проблемы и уверенно работать с Git в командном режиме.

---

## Структура Phase 1

- `Module_01_Linux/`
  - `01_theory.md` — принципы работы Linux, процессы, права, сервисы, скрипты (`[[DevOps/01_Linux/linux-cheatsheet]]`, `[[DevOps/01_Linux/filesystem-and-permissions]]`, `[[DevOps/01_Linux/processes-and-services]]`, `[[DevOps/01_Linux/shell-scripting]]`).
  - `02_practice.md` — практикум по CLI, правам, systemd, bash‑скриптам.
  - `03_project.md` — мини‑проект: bash‑скрипт мониторинга.
  - `04_quiz.md` — проверка понимания.

- `Module_02_Networking/`
  - `01_theory.md` — IP, порты, DNS, маршрутизация, основные утилиты (`[[DevOps/01_Linux/networking]]`, `[[DevOps/11_Networking/protocols]]`, `[[DevOps/11_Networking/troubleshooting]]`, `[[DevOps/09_Cloud/cloud-networking]]`).
  - `02_practice.md` — практика: ip/ss/tcpdump/iptables, диагностика реальных проблем.
  - `03_project.md` — мини‑проект: Nginx reverse proxy.
  - `04_quiz.md` — проверка понимания.

- `Module_03_Git/`
  - `01_theory.md` — модель данных Git, ветки, merge/rebase, стратегии (`[[DevOps/02_Git/git-cheatsheet]]`, `[[DevOps/02_Git/branching-strategies]]`, `[[DevOps/02_Git/hooks-and-automation]]`, `[[DevOps/02_Git/troubleshooting]]`).
  - `02_practice.md` — практикум: ветвление, PR, конфликты, reflog.
  - `03_project.md` — workflow команды DevOps‑проекта.
  - `04_quiz.md` — проверка понимания.

---

## Ожидаемые результаты

К концу Phase 1 вы:

- **Linux**
  - Уверенно используете команды из `[[DevOps/01_Linux/linux-cheatsheet]]`.
  - Понимаете права и владельцев (`[[DevOps/01_Linux/filesystem-and-permissions]]`).
  - Управляете сервисами через systemd и читаете логи `journalctl` (`[[DevOps/01_Linux/processes-and-services]]`).
  - Пишете надёжные bash‑скрипты (`[[DevOps/01_Linux/shell-scripting]]`).

- **Networking**
  - Диагностируете сетевые проблемы с помощью `ip`, `ss`, `tcpdump`, `curl`, `dig` (`[[DevOps/01_Linux/networking]]`, `[[DevOps/11_Networking/troubleshooting]]`).
  - Понимаете базовые протоколы (TCP/UDP, HTTP, DNS) (`[[DevOps/11_Networking/protocols]]`).

- **Git**
  - Работаете с ветками, rebase/merge, умеете чинить конфликты (`[[DevOps/02_Git/git-cheatsheet]]`, `[[DevOps/02_Git/branching-strategies]]`).
  - Умеете восстановиться после ошибок через reset/revert/reflog (`[[DevOps/02_Git/troubleshooting]]`).
  - Настраиваете простые pre-commit/pre-push‑хуки (`[[DevOps/02_Git/hooks-and-automation]]`).

---

## Чек‑лист Phase 1

- [ ] Подготовлено рабочее Linux/WSL2 окружение (см. `00_PREREQUISITES.md`).
- [ ] Пройден модуль `Module_01_Linux` и выполнены все практики и мини‑проект.
- [ ] Пройден модуль `Module_02_Networking`, выполнены практики, Nginx reverse proxy работает.
- [ ] Пройден модуль `Module_03_Git`, отработан командный workflow (feature‑ветки, PR, реверт/rollback).
- [ ] Все квизы по модулям пройдены с результатом не менее 80% правильных ответов.

