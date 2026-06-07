# NAT — трансляция сетевых адресов (SNAT / DNAT)

## Что такое NAT

**NAT (Network Address Translation)** — технология подмены IP-адресов в сетевых пакетах при прохождении через маршрутизатор или шлюз.

Основная задача NAT:

* предоставить доступ в интернет устройствам с приватными IP-адресами;
* публиковать внутренние сервисы во внешнюю сеть;
* скрывать внутреннюю структуру сети.

---

# Основные типы NAT

| Тип                        | Что изменяется      | Направление              | Применение              |
| -------------------------- | ------------------- | ------------------------ | ----------------------- |
| **SNAT** (Source NAT)      | Исходный IP-адрес   | Из локальной сети наружу | Выход в интернет        |
| **DNAT** (Destination NAT) | IP-адрес назначения | Из внешней сети внутрь   | Проброс портов          |
| **MASQUERADE**             | Частный случай SNAT | Из локальной сети наружу | Динамический внешний IP |

---

# Включение IP Forwarding

Для работы любого NAT маршрутизатор должен пересылать пакеты между интерфейсами.

## Временное включение

Работает до перезагрузки системы:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

---

## Постоянное включение

Добавить параметр в файл `/etc/sysctl.conf`:

```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
```

Применить настройки:

```bash
sudo sysctl -p
```

Проверить:

```bash
sysctl net.ipv4.ip_forward
```

Ожидаемый результат:

```text
net.ipv4.ip_forward = 1
```

---

# SNAT — выход локальной сети в интернет

## Маскарадинг (динамический внешний IP)

Наиболее распространённый вариант для домашних роутеров и DHCP-подключений.

```bash
sudo iptables -t nat -A POSTROUTING \
-o eth0 \
-j MASQUERADE
```

---

## SNAT с фиксированным IP

Если внешний IP-адрес не меняется:

```bash
sudo iptables -t nat -A POSTROUTING \
-o eth0 \
-j SNAT --to-source 1.2.3.4
```

---

# Как работает SNAT

Предположим:

| Устройство         | IP              |
| ------------------ | --------------- |
| Клиент             | `192.168.1.100` |
| Шлюз               | `192.168.1.1`   |
| Внешний IP шлюза   | `1.2.3.4`       |
| Сервер в интернете | `8.8.8.8`       |

### Шаг 1

Клиент отправляет пакет:

```text
SRC: 192.168.1.100
DST: 8.8.8.8
```

### Шаг 2

Шлюз выполняет SNAT:

```text
SRC: 1.2.3.4
DST: 8.8.8.8
```

### Шаг 3

Ответ приходит на внешний IP шлюза:

```text
SRC: 8.8.8.8
DST: 1.2.3.4
```

### Шаг 4

Шлюз восстанавливает исходный адрес клиента:

```text
SRC: 8.8.8.8
DST: 192.168.1.100
```

---

# DNAT — проброс портов

Позволяет публиковать внутренние сервисы во внешнюю сеть.

---

## Проброс веб-сервера

Внешний порт `8080` → внутренний сервер `192.168.1.100:80`

```bash
sudo iptables -t nat -A PREROUTING \
-p tcp \
--dport 8080 \
-j DNAT \
--to-destination 192.168.1.100:80
```

---

## Проброс SSH

Внешний порт `2222` → внутренний сервер `192.168.1.50:22`

```bash
sudo iptables -t nat -A PREROUTING \
-p tcp \
--dport 2222 \
-j DNAT \
--to-destination 192.168.1.50:22
```

---

## Проброс диапазона портов

```bash
sudo iptables -t nat -A PREROUTING \
-p tcp \
--dport 8000:8100 \
-j DNAT \
--to-destination 192.168.1.100:8000-8100
```

---

# Обязательное правило FORWARD

После DNAT трафик должен быть разрешён в цепочке `FORWARD`.

```bash
sudo iptables -A FORWARD \
-p tcp \
-d 192.168.1.100 \
--dport 80 \
-j ACCEPT
```

Без этого правила DNAT может работать, но пакеты будут блокироваться фильтрацией.

---

# Полный пример: доступ локальной сети в интернет

## Шаг 1. Включить маршрутизацию

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

---

## Шаг 2. Разрешить форвардинг

Предположим:

| Интерфейс | Назначение      |
| --------- | --------------- |
| `eth0`    | Внутренняя сеть |
| `eth1`    | Интернет        |

```bash
sudo iptables -A FORWARD \
-i eth0 -o eth1 \
-j ACCEPT
```

Разрешить обратный трафик:

```bash
sudo iptables -A FORWARD \
-i eth1 -o eth0 \
-m state --state ESTABLISHED,RELATED \
-j ACCEPT
```

---

## Шаг 3. Включить NAT

```bash
sudo iptables -t nat -A POSTROUTING \
-o eth1 \
-j MASQUERADE
```

---

# Полный пример: публикация внутреннего веб-сервера

## Включить IP Forwarding

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

---

## Настроить DNAT

```bash
sudo iptables -t nat -A PREROUTING \
-p tcp \
--dport 8080 \
-j DNAT \
--to-destination 192.168.1.100:80
```

---

## Разрешить форвардинг

```bash
sudo iptables -A FORWARD \
-p tcp \
-d 192.168.1.100 \
--dport 80 \
-j ACCEPT
```

---

# Просмотр правил NAT

Показать все NAT-правила:

```bash
sudo iptables -t nat -L -v -n
```

---

Только DNAT:

```bash
sudo iptables -t nat -L PREROUTING -n
```

---

Только SNAT:

```bash
sudo iptables -t nat -L POSTROUTING -n
```

---

# Удаление правил NAT

## Удаление по номеру

Сначала узнать номера правил:

```bash
sudo iptables -t nat -L PREROUTING \
--line-numbers
```

Удалить правило:

```bash
sudo iptables -t nat -D PREROUTING 2
```

---

## Удаление по содержимому

Удалить MASQUERADE:

```bash
sudo iptables -t nat -D POSTROUTING \
-o eth0 \
-j MASQUERADE
```

Удалить DNAT:

```bash
sudo iptables -t nat -D PREROUTING \
-p tcp \
--dport 8080 \
-j DNAT \
--to-destination 192.168.1.100:80
```

---

# Отслеживание соединений (conntrack)

NAT использует таблицу состояний соединений.

## Показать все соединения

```bash
sudo conntrack -L
```

---

## Фильтр по порту

```bash
sudo conntrack -L -p tcp --dport 80
```

---

## Только активные соединения

```bash
sudo conntrack -L | grep ESTABLISHED
```

---

# Проверка работы NAT

## На клиенте

Проверить внешний IP:

```bash
curl ifconfig.me
```

Должен отображаться внешний IP шлюза.

---

## На шлюзе

Наблюдать исходящие пакеты:

```bash
sudo tcpdump -i eth1 -n not port 22
```

Будет видно, что исходный IP заменён на внешний адрес шлюза.

---

# Сохранение правил

Установка:

```bash
sudo apt install iptables-persistent
```

Сохранение текущей конфигурации:

```bash
sudo netfilter-persistent save
```

Загрузка сохранённых правил:

```bash
sudo netfilter-persistent reload
```

---

# MASQUERADE vs SNAT

| MASQUERADE                        | SNAT                                      |
| --------------------------------- | ----------------------------------------- |
| Для динамических IP               | Для статических IP                        |
| IP определяется автоматически     | IP задаётся вручную                       |
| Немного медленнее                 | Немного быстрее                           |
| Используется на домашних роутерах | Используется на серверах и в дата-центрах |

---

# Дополнительный тип NAT — REDIRECT

Позволяет перенаправлять трафик на локальный порт.

Пример:

```bash
sudo iptables -t nat -A OUTPUT \
-p tcp \
--dport 80 \
-j REDIRECT --to-port 8080
```

Все обращения к порту `80` будут перенаправлены на локальный порт `8080`.

---

# Шпаргалка по NAT

| Тип                     | Цепочка                 | Действие                      |
| ----------------------- | ----------------------- | ----------------------------- |
| SNAT (динамический IP)  | `POSTROUTING`           | `-j MASQUERADE`               |
| SNAT (фиксированный IP) | `POSTROUTING`           | `-j SNAT --to-source IP`      |
| DNAT                    | `PREROUTING`            | `-j DNAT --to-destination IP` |
| REDIRECT                | `OUTPUT` / `PREROUTING` | `-j REDIRECT --to-port PORT`  |

---

# Запомнить

### NAT наружу

```bash
sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
```

### Проброс порта

```bash
sudo iptables -t nat -A PREROUTING \
-p tcp --dport 8080 \
-j DNAT --to-destination 192.168.1.100:80
```

### Проверить NAT

```bash
sudo iptables -t nat -L -v -n
```

### Проверить таблицу соединений

```bash
sudo conntrack -L
```
