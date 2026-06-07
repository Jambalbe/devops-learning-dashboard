# ✅ Модуль 1.5: Сети (расширенный блок)

**Дата прохождения:** 2026-06-07

## Чеклист навыков

### IP-адресация и маршрутизация
- [x] Понимание CIDR (/24, /16, /8)
- [x] Публичные vs приватные IP
- [x] Таблица маршрутизации (ip route)
- [x] Шлюз по умолчанию
- [x] ARP таблица (ip neigh)

### Диагностика и анализ трафика
- [x] ping, traceroute
- [x] ss (просмотр портов и соединений)
- [x] tcpdump (захват и анализ пакетов)
- [x] Понимание TCP handshake (SYN, SYN-ACK, ACK)
- [x] Анализ флагов TCP

### Фаервол (UFW и iptables)
- [x] UFW (разрешение портов, включение)
- [x] iptables: цепочки INPUT, OUTPUT, FORWARD
- [x] iptables: политики (DROP, ACCEPT)
- [x] Добавление/удаление правил (-A, -I, -D)
- [x] Логирование блокировок
- [x] conntrack (отслеживание состояния соединений)

### NAT (Network Address Translation)
- [x] SNAT vs DNAT (разница и применение)
- [x] MASQUERADE (маскарадинг)
- [x] IP forwarding (включение, проверка)
- [x] Проброс портов (port forwarding)

### DNS
- [x] /etc/resolv.conf и его структура
- [x] systemd-resolved (127.0.0.53)
- [x] resolvectl (статус, сброс кэша)
- [x] dig, nslookup, host
- [x] /etc/hosts (локальные записи)
- [x] Локальный DNS-сервер (dnsmasq)

### TLS/SSL и HTTPS
- [x] Генерация ключей (openssl)
- [x] Самоподписанный сертификат
- [x] Настройка HTTPS в nginx
- [x] Понимание TLS handshake
- [x] HTTP → HTTPS редирект

## Доказательства
- [x] curl https://192.168.1.147 работает
- [x] tcpdump показывает TCP handshake
- [x] iptables правила сохранены в configs/
- [x] dig @127.0.0.1 test.lab.local работает
- [x] Браузер открывает HTTPS (с предупреждением)

## Файлы конфигураций
- [nginx-https.conf](../configs/nginx-https.conf)
- [iptables-rules.v4](../configs/iptables-rules.v4)
- [dnsmasq.conf](../configs/dnsmasq.conf)

## Заметки
- Самоподписанный сертификат вызывает предупреждение в браузере — это нормально
- В production используется Let's Encrypt
- Все эксперименты на локальной ВМ (не в интернете)
