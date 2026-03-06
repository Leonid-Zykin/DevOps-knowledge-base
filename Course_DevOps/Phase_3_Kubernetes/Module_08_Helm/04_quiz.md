## Module 08 — Helm Quiz

Ориентируйтесь на `[[DevOps/04_Kubernetes/helm]]`.

---

### Вопрос 1 — Release

Что такое Helm Release?

a) Версия чарта  
b) Установленный экземпляр чарта в кластере с уникальным именем  
c) Файл values.yaml  
d) Репозиторий чартов

---

### Вопрос 2 — Приоритет values

Какой порядок приоритета при переопределении values?

a) values.yaml > --set > -f file  
b) --set > -f file > values.yaml  
c) -f file > values.yaml > --set  
d) Все равны

---

### Вопрос 3 — helm template

Что делает `helm template release-name ./chart`?

a) Устанавливает чарт в кластер  
b) Рендерит шаблоны в итоговые YAML без установки  
c) Удаляет релиз  
d) Обновляет зависимости

---

### Вопрос 4 — Условный ресурс

Как в шаблоне сделать так, чтобы Ingress создавался только при `ingress.enabled: true`?

a) Никак, Helm не поддерживает условия  
b) `{{- if .Values.ingress.enabled }}` ... `{{- end }}`  
c) Только через отдельный чарт  
d) Через annotations

---

### Вопрос 5 — Зависимости

Где в чарте описываются зависимости (dependencies)?

a) В values.yaml  
b) В Chart.yaml в секции dependencies  
c) В templates/  
d) В .helmignore

---

### Вопрос 6 — helm upgrade

Что делает `helm upgrade myapp ./mychart`?

a) Удаляет релиз и создаёт заново  
b) Обновляет существующий релиз myapp новыми манифестами/values  
c) Только меняет values без применения  
d) Откатывает к предыдущей версии

---

### Вопрос 7 — helm rollback

Как откатить релиз к предыдущей ревизии?

a) `helm delete myapp`  
b) `helm rollback myapp 1`  
c) `helm reset myapp`  
d) `helm undo myapp`

---

### Вопрос 8 — include в шаблонах

Для чего используется `{{ include "mychart.fullname" . }}`?

a) Для импорта файлов  
b) Для вызова именованного шаблона (define) и переиспользования логики  
c) Для загрузки ConfigMap  
d) Для подключения Secret

---

### Вопрос 9 — GitOps и Helm

Почему для прода рекомендуется хранить values в Git, а не передавать через --set?

a) --set медленнее  
b) Чтобы конфигурация была версионирована и воспроизводима  
c) --set не поддерживается в Argo CD  
d) Git быстрее

---

### Вопрос 10 — helm uninstall

Что делает `helm uninstall myapp -n prod`?

a) Откатывает релиз  
b) Удаляет релиз и связанные ресурсы из кластера  
c) Только удаляет историю  
d) Обновляет чарт

---

> [!note]- Answers
> **1** — b) Release — установленный экземпляр чарта.  
> **2** — b) --set > -f file > values.yaml.  
> **3** — b) Рендерит шаблоны без установки.  
> **4** — b) `{{- if .Values.ingress.enabled }}` ... `{{- end }}`.  
> **5** — b) В Chart.yaml, секция dependencies.  
> **6** — b) Обновляет существующий релиз.  
> **7** — b) `helm rollback myapp 1`.  
> **8** — b) Вызов define для переиспользования.  
> **9** — b) Версионирование и воспроизводимость.  
> **10** — b) Удаляет релиз и ресурсы.
