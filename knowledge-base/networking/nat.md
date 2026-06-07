# NAT — трансляция сетевых адресов (SNAT / DNAT)

## Что такое NAT

**NAT (Network Address Translation)** — технология подмены IP-адресов в заголовках пакетов при прохождении через маршрутизатор.

## Два основных типа NAT

| Тип | Что меняет | Направление | Где применяется |
|-----|------------|-------------|-----------------|
| **SNAT** (Source NAT) | Исходный IP | Входящие пакеты → наружу | Выход в интернет (маскарадинг) |
| **DNAT** (Destination NAT) | Целевой IP | Внешние пакеты → внутрь | Проброс портов, балансировка |

## Включение IP forwarding (обязательно для любого NAT)

```bash
# Временно (до перезагрузки)
sudo sysctl -w net.ipv4.ip_forward=1

# Постоянно
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
SNAT — маскарадинг (выход в интернет)
bash
# Основное правило маскарадинга (динамический IP)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# SNAT с фиксированным IP
sudo iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 1.2.3.4
#Как работает:

Клиент (192.168.1.100) отправляет пакет на шлюз (192.168.1.1)

Шлюз меняет source IP на свой внешний IP (1.2.3.4)

Ответ от сервера приходит на шлюз

Шлюз восстанавливает исходный IP и передаёт клиенту

#DNAT — проброс портов (доступ извне)
bash
# Проброс порта 8080 на внутренний сервер 192.168.1.100:80
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.100:80

# Проброс на другой порт
sudo iptables -t nat -A PREROUTING -p tcp --dport 2222 -j DNAT --to-destination 192.168.1.50:22

# Проброс с несколькими портами
sudo iptables -t nat -A PREROUTING -p tcp --dport 8000:8100 -j DNAT --to-destination 192.168.1.100:8000-8100
#Обязательно дополнить правилом FORWARD:

bash
sudo iptables -A FORWARD -p tcp -d 192.168.1.100 --dport 80 -j ACCEPT
#Полный пример: выход локальной сети в интернет
bash
# 1. Включить IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# 2. Разрешить форвардинг
sudo iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT   # eth0 — внутрь, eth1 — наружу
sudo iptables -A FORWARD -i eth1 -o eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT

# 3. Включить маскарадинг
sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
#Полный пример: проброс порта на внутренний веб-сервер
bash
# 1. Включить IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# 2. DNAT: внешний порт 8080 → внутренний 192.168.1.100:80
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.100:80

# 3. Разрешить форвардинг для этого трафика
sudo iptables -A FORWARD -p tcp -d 192.168.1.100 --dport 80 -j ACCEPT
Просмотр правил NAT
bash
sudo iptables -t nat -L -v -n           # все правила NAT
sudo iptables -t nat -L PREROUTING -n   # только DNAT
sudo iptables -t nat -L POSTROUTING -n  # только SNAT
Удаление правил NAT
bash
# По номеру (сначала посмотреть с --line-numbers)
sudo iptables -t nat -L PREROUTING --line-numbers
sudo iptables -t nat -D PREROUTING 2

# По содержимому
sudo iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -t nat -D PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.100:80
Отслеживание активных NAT-соединений (conntrack)
bash
sudo conntrack -L                    # список всех соединений
sudo conntrack -L -p tcp --dport 80  # фильтр по порту
sudo conntrack -L | grep ESTABLISHED
#Проверка работы NAT
bash
# На клиенте (внутренняя машина)
curl ifconfig.me                     # должен показать внешний IP шлюза

# На шлюзе
sudo tcpdump -i eth1 -n not port 22  # смотреть исходящие пакеты, IP подменён
Сохранение правил NAT (iptables-persistent)
bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
sudo netfilter-persistent reload

#Разница между MASQUERADE и SNAT
MASQUERADE	SNAT --to-source
Подходит для динамических IP (DHCP)	Для статических IP
Определяет внешний IP автоматически	Нужно указывать IP вручную
Чуть медленнее (из-за определения IP)	Быстрее (нет накладных расходов)
Пример: домашний роутер	Пример: сервер в дата-центре

#Краткая шпаргалка
Тип	Цепочка	Ключевая опция
SNAT (маскарадинг)	POSTROUTING	-j MASQUERADE
SNAT (фикс. IP)	POSTROUTING	-j SNAT --to-source IP
DNAT	PREROUTING	-j DNAT --to-destination IP
Перенаправление (локальное)	OUTPUT	-j REDIRECT --to-port 8080
