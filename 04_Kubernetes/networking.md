tags: [devops, kubernetes-networking]

## Kubernetes Networking — Services, Ingress, NetworkPolicy, DNS

> ⚡ Tip: Kubernetes‑сеть плоская — каждый Pod имеет свой IP, Pod'ы могут ходить друг к другу напрямую; Service даёт стабильное имя/IP и балансировку.

### Services: ClusterIP, NodePort, LoadBalancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-svc
spec:
  type: ClusterIP
  selector:
    app: api
  ports:
    - name: http
      port: 80
      targetPort: 8080
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-nodeport
spec:
  type: NodePort
  selector:
    app: api
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080      # порт на ноде
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-lb
spec:
  type: LoadBalancer
  selector:
    app: api
  ports:
    - port: 80
      targetPort: 8080
```

Команды:

```bash
kubectl get svc
kubectl describe svc api-svc
kubectl get endpoints api-svc
```

### DNS в Kubernetes

Формат имён:

- `service.namespace.svc.cluster.local`
- внутри одного ns достаточно `service`;
- между ns: `service.other-ns`.

Проверка:

```bash
kubectl exec -it pod -- nslookup api-svc
kubectl exec -it pod -- dig api-svc +short
```

### Ingress (HTTP/HTTPS маршрутизация)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api-svc
                port:
                  number: 80
  tls:
    - hosts:
        - api.example.com
      secretName: api-tls
```

Проверка:

```bash
kubectl get ingress
kubectl describe ingress app-ingress
```

### NetworkPolicy — ограничение трафика

Базовый пример: разрешить трафик к `db` только от `api`.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-allow-from-api
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api
      ports:
        - protocol: TCP
          port: 5432
```

Разрешить egress только к нужным внешним сервисам:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-egress
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
    - Egress
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              env: prod
      ports:
        - protocol: TCP
          port: 5432
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
            except:
              - 10.0.0.0/8
      ports:
        - protocol: TCP
          port: 443
```

> ⚠️ Warning: NetworkPolicy начинает работать только если CNI‑плагин поддерживает их (Calico, Cilium и т.п.). В некоторых managed‑кластерах по умолчанию политика «allow all».

### Troubleshooting сети

1. **Pod не может достучаться до Service**  
   ```bash
   kubectl get endpoints api-svc
   kubectl describe svc api-svc
   kubectl get pods -l app=api -o wide
   kubectl exec -it debug-pod -- curl -v http://api-svc
   ```
2. **Проблемы с Ingress**  
   ```bash
   kubectl describe ingress app-ingress
   kubectl logs -n ingress-nginx deploy/ingress-nginx-controller
   ```
3. **NetworkPolicy блокирует трафик**  
   ```bash
   kubectl get networkpolicy
   kubectl describe networkpolicy db-allow-from-api
   ```

Связанные заметки: `[[core-concepts]]`, `[[workloads]]`, `[[storage]]`, `[[11_Networking/troubleshooting]]`, `[[11_Networking/service-mesh]]`.

## Gotchas

- **Service без endpoints**  
  - **Проблема**: `kubectl get endpoints` показывает пусто, трафик не идёт.  
  - **Решение**: проверить labels pod'ов и selector'ы Service (см. `[[core-concepts]]`).

- **Ingress без корректного ingressClass/контроллера**  
  - **Проблема**: Ingress создан, но трафик не маршрутизируется.  
  - **Решение**: убедиться, что установлен Ingress Controller и `ingressClassName`/аннотации соответствуют его настройкам.

- **NetworkPolicy, внезапно обрубающая трафик**  
  - **Проблема**: после включения первой политики все неописанные потоки блокируются.  
  - **Решение**: начинать с audit/test‑кластеров, аккуратно описывать как ingress, так и egress, использовать namespaceSelector/podSelector.

> ⚠️ Warning: Изменения NetworkPolicy в прод‑кластере без тестов и без понимания текущих потоков трафика (в т.ч. DNS, метрики, логи, сервис‑мэши) могут мгновенно привести к массовым отказам. Всегда тестируйте в dev/stage и используйте наблюдаемость (`[[08_Monitoring/prometheus]]`, `[[08_Monitoring/loki-and-logging]]`).

