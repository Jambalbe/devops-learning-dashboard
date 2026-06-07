# IP-адресация и маршрутизация в Linux

## Просмотр IP-адресов

```bash
# Все сетевые интерфейсы и IP-адреса
ip addr show

# Сокращённый вариант
ip a

# Только IPv4
ip -4 a

# Только IPv6
ip -6 a

# Старый способ (требует пакет net-tools)
ifconfig
```

---

## Просмотр маршрутов

```bash
# Таблица маршрутизации
ip route show

# Сокращённый вариант
ip route

# Старый способ
route -n

# Узнать, через какой интерфейс пойдёт пакет
ip route get 8.8.8.8
```

---

## Добавление маршрутов

### Маршрут к подсети

```bash
sudo ip route add 192.168.2.0/24 via 192.168.1.1 dev eth0
```

### Маршрут по умолчанию (Default Gateway)

```bash
sudo ip route add default via 192.168.1.1 dev eth0
```

### Маршрут к конкретному хосту

```bash
sudo ip route add 10.0.0.5 via 192.168.1.100 dev eth0
```

---

## Удаление маршрутов

```bash
sudo ip route del 192.168.2.0/24
sudo ip route del default
sudo ip route del 10.0.0.5
```

---

## Управление IP-адресами

### Добавление IP-адреса

```bash
sudo ip addr add 192.168.1.100/24 dev eth0
```

Добавление дополнительного IP-адреса:

```bash
sudo ip addr add 192.168.1.101/24 dev eth0 label eth0:1
```

### Удаление IP-адреса

```bash
sudo ip addr del 192.168.1.100/24 dev eth0
```

---

## Управление сетевым интерфейсом

### Включить интерфейс

```bash
sudo ip link set eth0 up
```

### Выключить интерфейс

```bash
sudo ip link set eth0 down
```

---

## ARP-таблица (IP → MAC)

### Просмотр

```bash
# ARP-таблица
ip neigh

# Старый способ
arp -n

# Для конкретного интерфейса
ip neigh show dev eth0
```

### Управление ARP-записями

```bash
# Добавить статическую запись
sudo ip neigh add 192.168.1.50 \
lladdr aa:bb:cc:dd:ee:ff dev eth0

# Удалить запись
sudo ip neigh del 192.168.1.50 dev eth0

# Очистить ARP-кэш интерфейса
sudo ip neigh flush dev eth0
```

---

## Настройка интерфейса через ifup/ifdown (Debian/Ubuntu)

```bash
sudo ifdown eth0
sudo ifup eth0
```

---

# Постоянная настройка сети

## Ubuntu (Netplan)

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
      dhcp4: no

      addresses:
        - 192.168.1.147/24

      routes:
        - to: default
          via: 192.168.1.1

      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

Применение настроек:

```bash
sudo netplan apply
```

---

## CentOS / RHEL

Файл:

```text
/etc/sysconfig/network-scripts/ifcfg-eth0
```

Пример конфигурации:

```ini
DEVICE=eth0
BOOTPROTO=static
IPADDR=192.168.1.147
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
ONBOOT=yes
```

---

## Просмотр статистики сетевых интерфейсов

```bash
# Счётчики RX/TX и ошибки
ip -s link show eth0

# Старый способ
netstat -i

# Слушающие TCP-порты
ss -tlnp

# Слушающие UDP-порты
ss -ulnp
```

---

## Диагностика сети

### Проверка доступности узла

```bash
ping -c 4 8.8.8.8
```

### Просмотр маршрута до хоста

```bash
traceroute -n 8.8.8.8
```

### Комбинация ping и traceroute

```bash
mtr -n 8.8.8.8
```

> Требуется установка пакета `mtr`.

---

# Приватные диапазоны IP-адресов

| Диапазон                        | Класс | Назначение                |
| ------------------------------- | ----- | ------------------------- |
| `10.0.0.0 - 10.255.255.255`     | A     | Крупные сети              |
| `172.16.0.0 - 172.31.255.255`   | B     | Средние сети              |
| `192.168.0.0 - 192.168.255.255` | C     | Домашние и небольшие сети |
| `127.0.0.0 - 127.255.255.255`   | —     | Loopback (`localhost`)    |

---

# CIDR-маски и количество хостов

| Маска           | CIDR | Доступно хостов |
| --------------- | ---- | --------------- |
| 255.255.255.0   | /24  | 254             |
| 255.255.255.128 | /25  | 126             |
| 255.255.255.192 | /26  | 62              |
| 255.255.255.224 | /27  | 30              |
| 255.255.255.240 | /28  | 14              |
| 255.255.255.248 | /29  | 6               |
| 255.255.255.252 | /30  | 2               |

---

# Шпаргалка по ip route

```bash
# Все таблицы маршрутизации
ip route show table all

# Добавить маршрут к подсети
ip route add 192.168.2.0/24 via 192.168.1.1

# Добавить маршрут через конкретный интерфейс
ip route add 10.0.0.0/8 via 192.168.1.2 dev eth1

# Заменить существующий маршрут
ip route replace 192.168.2.0/24 via 192.168.1.1

# Показать кэш маршрутов (не всегда доступно)
ip route show cache
```

---

# Полезные алиасы

Добавить в файл `~/.bashrc`:

```bash
# Показать IPv4-адреса
alias myip='ip -4 -br addr show | grep -v LOOPBACK'

# Красивый вывод маршрутов
alias routeinfo='ip route show | column -t'
```

После добавления применить изменения:

```bash
source ~/.bashrc
```

---

## Полезно запомнить

### Узнать свой IP

```bash
ip a
```

### Узнать шлюз по умолчанию

```bash
ip route | grep default
```

### Узнать DNS-серверы

```bash
cat /etc/resolv.conf
```

### Проверить путь до узла

```bash
traceroute 8.8.8.8
```

### Проверить открытые порты

```bash
ss -tulnp
```
