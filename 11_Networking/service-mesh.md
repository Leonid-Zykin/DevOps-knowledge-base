tags: [devops, service-mesh]

## Service Mesh — Istio, Linkerd, sidecar, mTLS, traffic

> ⚡ Tip: Service Mesh выносит сетевые функции (mTLS, retries, timeouts, routing) в инфраструктуру через sidecar‑прокси (обычно Envoy).

### Базовые фичи mesh

- автоматический mTLS между сервисами;
- circuit breaking, retries, timeouts;
- canary/blue‑green/traffic‑splitting;
- observability (metrics/logs/traces).

### Istio пример (VirtualService/ DestinationRule)

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: api
spec:
  host: api.prod.svc.cluster.local
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
```

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: api
spec:
  hosts:
    - api.prod.svc.cluster.local
  http:
    - route:
        - destination:
            host: api.prod.svc.cluster.local
            subset: v1
          weight: 90
        - destination:
            host: api.prod.svc.cluster.local
            subset: v2
          weight: 10
```

### mTLS в mesh

Istio PeerAuthentication (пример):

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: prod
spec:
  mtls:
    mode: STRICT
```

Связанные: `[[04_Kubernetes/networking]]`, `[[10_Security/network-security]]`, `[[08_Monitoring/prometheus]]`.

## Gotchas

- **Включение mTLS без понимания существующих клиентов**  
  - **Проблема**: сторонние сервисы/инструменты перестают коннектиться.  
  - **Решение**: поэтапный rollout (PERMISSIVE → STRICT), тестовые окружения, документация.

- **Сложность дебага с sidecar'ами**  
  - **Проблема**: трафик идёт через прокси, обычные `curl`/`tcpdump` не всегда очевидны.  
  - **Решение**: использовать mesh‑специфичные инструменты (istioctl, kiali), отдельные debug‑pods.

- **Дублирование логики retries/timeouts в приложении и mesh**  
  - **Проблема**: непредсказуемые задержки и каскадные ретраи.  
  - **Решение**: выбрать, где основная политика (mesh или client‑lib), и согласовать настройки.

> ⚠️ Warning: Введение service mesh в прод‑кластер без пилотного проекта и команды, понимающей его устройство, часто приводит к резкому росту сложности, непредсказуемым сетевым эффектам и outage. Всегда начинайте с небольшого набора сервисов и тщательно мониторьте.

