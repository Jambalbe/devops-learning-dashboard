# iptables — продвинутый фаервол

## Основные цепочки
- `INPUT`  — входящие пакеты
- `OUTPUT` — исходящие пакеты
- `FORWARD` — маршрутизируемые пакеты

## Таблицы
- `filter` (по умолчанию) — разрешение/запрет
- `nat` — преобразование адресов (SNAT/DNAT)
- `mangle` — изменение TTL, TOS и т.п.

## Просмотр правил
```bash
sudo iptables -L -v -n           # список правил со счетчиками
sudo iptables -S                 # правила в формате команд
sudo iptables -t nat -L -n       # правила NAT
```bash
#Добавление/удаление правил
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT   # добавить правило
sudo iptables -I INPUT 1 -s 10.0.0.5 -j DROP         # вставить в начало
sudo iptables -D INPUT 2                              # удалить по номеру

#Политики по умолчанию
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD DROP

#Разрешить установленные соединения (важно!)
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

#Пример: разрешить только SSH и HTTP
sudo iptables -F
sudo iptables -P INPUT DROP
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

#Логирование
sudo iptables -I INPUT 1 -m limit --limit 5/min -j LOG --log-prefix "DROP: "
Смотреть логи: sudo journalctl -kf | grep DROP

#Сохранение правил
sudo apt install iptables-persistent
sudo netfilter-persistent save

#NAT (Source NAT)
 Включить IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1
 Маскарадинг
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

#DNAT (проброс портов)
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.100:80

#Удалить все правила
sudo iptables -F
sudo iptables -X
sudo iptables -t nat -F

#Полезные флаги
Флаг	Назначение
-A	Append (добавить в конец)
-I	Insert (вставить в начало)
-D	Delete (удалить)
-L	List (список)
-F	Flush (очистить)
-p	Protocol (tcp/udp/icmp)
--dport	Destination port
-s	Source IP
-d	Destination IP
-i	Input interface
-o	Output interface
-j	Jump (цель: ACCEPT/DROP/LOG/REJECT)
-m state --state	Состояние соединения











