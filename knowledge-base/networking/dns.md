# DNS — система доменных имён (и настройка в Linux)

## Что такое DNS

**DNS (Domain Name System)** — система преобразования доменных имён в IP-адреса.

Пример:

```text
google.com → 142.250.109.138
```

Без DNS пользователям пришлось бы запоминать IP-адреса вместо удобных доменных имён.

---

# Просмотр DNS-настроек системы

```bash
# Текущие DNS-серверы системы
cat /etc/resolv.conf

# Подробная информация о systemd-resolved
resolvectl status

# Альтернативная команда (старые версии)
systemd-resolve --status
```

---

# Выполнение DNS-запросов вручную

## Утилита dig

### Получить A-запись домена

```bash
dig google.com
```

### Обратное разрешение имени (PTR-запись)

```bash
dig -x 8.8.8.8
```

### Использовать конкретный DNS-сервер

```bash
dig @8.8.8.8 google.com
```

### Получить NS-записи

```bash
dig google.com NS
```

### Получить MX-записи

```bash
dig google.com MX
```

---

## Утилита nslookup

```bash
nslookup google.com
```

---

## Утилита host

```bash
host google.com
```

---

# Файл /etc/hosts

Локальная база сопоставления IP-адресов и имён хостов.

Просмотр содержимого:

```bash
cat /etc/hosts
```

Пример:

```text
127.0.0.1       localhost
192.168.1.100   myserver.local
```

---

## Добавление локальной записи

```bash
echo "192.168.1.147 test.lab.local" | sudo tee -a /etc/hosts
```

Проверка:

```bash
ping test.lab.local
```

---

# Systemd-resolved

Современный DNS-резолвер, используемый во многих дистрибутивах Linux.

## Проверка статуса

```bash
systemctl status systemd-resolved
```

## Просмотр настроек

```bash
resolvectl status
```

## Статистика DNS-кэша

```bash
resolvectl statistics
```

## Очистка DNS-кэша

```bash
sudo resolvectl flush-caches
```

## Выполнить DNS-запрос

```bash
resolvectl query google.com
```

Показывает:

* IP-адреса
* интерфейс
* DNS-сервер, выполнивший запрос

---

# Настройка DNS через Netplan (Ubuntu)

Файл:

```text
/etc/netplan/00-installer-config.yaml
```

Пример конфигурации:

```yaml
network:
  version: 2

  ethernets:
    eth0:
      dhcp4: yes

      dhcp4-overrides:
        use-dns: no

      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4

        search:
          - lab.local
```

Параметр:

```yaml
use-dns: no
```

означает игнорирование DNS-серверов, полученных через DHCP.

---

## Применение конфигурации

```bash
sudo netplan apply
```

---

# Локальный DNS-сервер dnsmasq

## Установка

```bash
sudo apt install dnsmasq -y
```

---

## Конфигурация

Файл:

```text
/etc/dnsmasq.d/lab.conf
```

Пример:

```ini
# Слушать только локально
interface=lo
bind-interfaces

# Не использовать /etc/resolv.conf
no-resolv

# Внешние DNS
server=8.8.8.8
server=8.8.4.4

# Локальный домен
domain=lab.local

# Статические записи
address=/test.lab.local/192.168.1.147
address=/api.lab.local/192.168.1.147

# CNAME
cname=www.lab.local,test.lab.local
```

---

## Запуск сервиса

```bash
sudo systemctl restart dnsmasq
sudo systemctl enable dnsmasq
```

---

## Проверка работы

```bash
dig @127.0.0.1 test.lab.local
```

---

# Порядок разрешения имён

При поиске IP-адреса система обычно использует следующий порядок:

1. `/etc/hosts`
2. `systemd-resolved` (если используется)
3. DNS-серверы из настроек сети
4. `/etc/resolv.conf` (если не является symlink)

---

# Диагностика DNS-проблем

## Проверить доступность DNS-сервера

```bash
dig @8.8.8.8 google.com +tcp
```

---

## Выполнить полную трассировку DNS

```bash
dig +trace google.com
```

Показывает весь путь запроса:

```text
Root DNS → TLD DNS → Authoritative DNS
```

---

## Проверить системное разрешение имени

```bash
getent hosts google.com
```

---

## Перехват DNS-запросов

```bash
sudo tcpdump -i eth0 port 53 -n
```

Позволяет увидеть все DNS-запросы, проходящие через интерфейс.

---

# Очистка DNS-кэша

## systemd-resolved

```bash
sudo resolvectl flush-caches
```

---

## dnsmasq

```bash
sudo systemctl restart dnsmasq
```

---

## nscd

```bash
sudo systemctl restart nscd
```

---

# Популярные публичные DNS-серверы

| Провайдер  | DNS-серверы                        |
| ---------- | ---------------------------------- |
| Google     | `8.8.8.8`, `8.8.4.4`               |
| Cloudflare | `1.1.1.1`, `1.0.0.1`               |
| OpenDNS    | `208.67.222.222`, `208.67.220.220` |
| Quad9      | `9.9.9.9`, `149.112.112.112`       |

---

# Шпаргалка по DNS

| Команда                                   | Назначение                 |
| ----------------------------------------- | -------------------------- |
| `cat /etc/resolv.conf`                    | DNS-серверы системы        |
| `resolvectl status`                       | Статус systemd-resolved    |
| `dig domain`                              | Подробный DNS-запрос       |
| `nslookup domain`                         | Простой DNS-запрос         |
| `host domain`                             | Краткий DNS-запрос         |
| `dig -x IP`                               | Обратное разрешение имени  |
| `dig @server domain`                      | Запрос к конкретному DNS   |
| `sudo resolvectl flush-caches`            | Очистить DNS-кэш           |
| `sudo systemctl restart systemd-resolved` | Перезапустить DNS-резолвер |

---

# Полезные команды

## Узнать используемые DNS-серверы

```bash
resolvectl dns
```

---

## Проверить DNS для конкретного интерфейса

```bash
resolvectl status eth0
```

---

## Проверить авторитетный DNS домена

```bash
dig google.com NS
```

---

## Проверить почтовые серверы домена

```bash
dig google.com MX
```

---

## Проверить TXT-записи

```bash
dig google.com TXT
```

---

## Быстрый вывод только IP-адресов

```bash
dig +short google.com
```

---

## Проверить внешний IP через DNS

```bash
dig +short myip.opendns.com @resolver1.opendns.com
```
