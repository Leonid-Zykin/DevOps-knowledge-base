tags: [devops, networking-protocols]

## Протоколы — TCP/IP, HTTP/2, gRPC, DNS, DHCP

> ⚡ Tip: При отладке сети почти всегда смотри на: IP/DNS → TCP → TLS → HTTP/gRPC.

### TCP/IP

- **TCP**: надёжный потоковый протокол (порядок, доставка, контроль перегрузки).
- **UDP**: без установления соединения, без гарантий, но быстрее (DNS, VoIP).

Основные параметры:

```bash
ss -tulpn                            # слушающие сокеты
ss -tn 'sport = :443'                # TCP 443
```

Флаги TCP (SYN/ACK/FIN/RST) можно смотреть через `tcpdump` (`[[01_Linux/networking]]`).

### HTTP/1.1 vs HTTP/2

- HTTP/1.1: каждый запрос — отдельное соединение (с keep-alive), head-of-line blocking.
- HTTP/2: одно соединение, мультиплексирование запросов, сжатие заголовков (HPACK), server push.

Проверка:

```bash
curl -I --http2 https://example.com
curl -v --http2-prior-knowledge https://example.com
```

### gRPC

- поверх HTTP/2, бинарный протокол (Protobuf), bidirectional streaming.

Отладка:

- использовать `grpcurl`:

```bash
grpcurl -plaintext localhost:50051 list
grpcurl -plaintext localhost:50051 my.Service/Method
```

### DNS

Запрос A‑записей:

```bash
dig example.com
dIG +short example.com
dig A example.com @8.8.8.8
```

NS и MX:

```bash
dig NS example.com
dig MX example.com
```

Trace:

```bash
dig +trace example.com
```

См. также `[[01_Linux/networking]]`, `[[09_Cloud/cloud-networking]]`.

### DHCP

- выдаёт IP, маску, gateway, DNS;
- чаще всего работает как сервис на роутере/хосте.

Диагностика:

```bash
ip a
ip r
cat /etc/resolv.conf
journalctl -u systemd-networkd -u NetworkManager
```

Связанные: `[[11_Networking/load-balancing]]`, `[[11_Networking/service-mesh]]`, `[[11_Networking/troubleshooting]]`.

## Gotchas

- **Игнорирование MTU и фрагментации**  
  - **Проблема**: странные тайм‑ауты и обрывы туннелей/VPN.  
  - **Решение**: проверять MTU на интерфейсах, использовать PMTUD и настройки MSS.

- **Смешение HTTP/1.1 и HTTP/2 ожиданий**  
  - **Проблема**: попытки дебага HTTP/2 трафика текстовыми средствами.  
  - **Решение**: использовать браузерные DevTools, специализированные прокси (mitmproxy) или gRPC‑инструменты.

- **DNS‑кэш на клиентах/прокси**  
  - **Проблема**: изменение A‑записей не сразу влияет на клиентов.  
  - **Решение**: корректно выставлять TTL, учитывать локальные кэши (systemd‑resolved, nscd, приложения).

> ⚠️ Warning: Попытки «починить» сложные сетевые проблемы только изменением приложения часто ведут в тупик. Сначала убедись, что DNS, маршрутизация, firewall, MTU и TCP‑уровень работают корректно (`[[01_Linux/networking]]`, `[[11_Networking/troubleshooting]]`), и только потом смотри на слой HTTP/gRPC.

