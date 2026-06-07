# IP-адресация и маршрутизация в Linux

## Просмотр IP-адресов
```bash
ip addr show               # все интерфейсы и IP
ip a                       # сокращённый вариант
ip -4 a                    # только IPv4
ip -6 a                    # только IPv6
ifconfig                   # старый способ (требует net-tools)
Просмотр маршрутов
bash
ip route show              # таблица маршрутизации
ip route                   # сокращённо
route -n                   # старый способ
ip route get 8.8.8.8       # узнать, через какой интерфейс идёт пакет
Добавление маршрутов
bash
# Добавить маршрут к подсети
sudo ip route add 192.168.2.0/24 via 192.168.1.1 dev eth0

# Добавить маршрут по умолчанию (шлюз)
sudo ip route add default via 192.168.1.1 dev eth0

# Добавить маршрут к конкретному хосту
sudo ip route add 10.0.0.5 via 192.168.1.100 dev eth0
#Удаление маршрутов
bash
sudo ip route del 192.168.2.0/24
sudo ip route del default
sudo ip route del 10.0.0.5
#Добавление IP-адреса на интерфейс
bash
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip addr add 192.168.1.101/24 dev eth0 label eth0:1   # вторичный IP с меткой
#Удаление IP-адреса
bash
sudo ip addr del 192.168.1.100/24 dev eth0
#Поднятие/опускание интерфейса
bash
sudo ip link set eth0 up
sudo ip link set eth0 down
#Просмотр ARP-таблицы (IP → MAC)
bash
ip neigh           # ARP-таблица
arp -n             # старый способ
ip neigh show dev eth0   # для конкретного интерфейса
#Управление ARP-таблицей
bash
sudo ip neigh add 192.168.1.50 lladdr aa:bb:cc:dd:ee:ff dev eth0   # добавить статическую запись
sudo ip neigh del 192.168.1.50 dev eth0                            # удалить запись
sudo ip neigh flush dev eth0                                        # очистить ARP для интерфейса
#Настройка интерфейса с помощью ifup/ifdown (Debian/Ubuntu)
bash
sudo ifdown eth0
sudo ifup eth0
#Постоянная настройка сети (Ubuntu Netplan)
yaml
# Файл: /etc/netplan/00-installer-config.yaml
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
        addresses: [8.8.8.8, 8.8.4.4]
#Применение изменений:

bash
sudo netplan apply
Постоянная настройка сети (CentOS/RHEL)
bash
# /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
BOOTPROTO=static
IPADDR=192.168.1.147
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
ONBOOT=yes
Просмотр статистики сетевых интерфейсов
bash
ip -s link show eth0        # счётчики RX/TX пакетов и ошибок
netstat -i                  # статистика интерфейсов
ss -tlnp                    # слушающие TCP-порты
ss -ulnp                    # слушающие UDP-порты
#Диагностика подключения
bash
ping -c 4 8.8.8.8           # проверка доступности
traceroute -n 8.8.8.8       # маршрут до хоста
mtr -n 8.8.8.8              # комбинация ping + traceroute (требует установки)
#IP-адресация: приватные и публичные диапазоны
Диапазон	Класс	Примечание
10.0.0.0 – 10.255.255.255	A	Крупные сети
172.16.0.0 – 172.31.255.255	B	Средние сети
192.168.0.0 – 192.168.255.255	C	Малые сети (домашние)
127.0.0.0 – 127.255.255.255	-	Loopback (localhost)
#CIDR-маски (количество хостов в подсети)
Маска	CIDR	Кол-во хостов
255.255.255.0	/24	254
255.255.255.128	/25	126
255.255.255.192	/26	62
255.255.255.224	/27	30
255.255.255.240	/28	14
255.255.255.248	/29	6
255.255.255.252	/30	2
#Краткая шпаргалка по ip route
bash
ip route show table all     # все таблицы маршрутизации
ip route add 192.168.2.0/24 via 192.168.1.1
ip route add 10.0.0.0/8 via 192.168.1.2 dev eth1
ip route replace           # заменить маршрут, если существует
ip route show cache        # кэш маршрутизации (не всегда доступен)
#Полезные алиасы (~/.bashrc)
bash
alias myip='ip -4 -br addr show | grep -v LOOPBACK'
alias routeinfo='ip route show | column -t'
