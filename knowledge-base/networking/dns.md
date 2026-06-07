# DNS — система доменных имён (и настройка в Linux)

## Что такое DNS

DNS (Domain Name System) преобразует доменные имена (google.com) в IP-адреса (142.250.109.138).

## Просмотр DNS-настроек системы

```bash
cat /etc/resolv.conf           # текущие DNS-серверы (часто ссылка на systemd-resolved)
resolvectl status              # детальная информация (systemd-resolved)
systemd-resolve --status       # альтернативная команда (старые версии)
DNS-запросы вручную
bash
dig google.com                 # полный DNS-запрос (A-запись)
dig -x 8.8.8.8                 # обратный lookup (IP → имя)
dig @8.8.8.8 google.com        # использовать конкретный DNS-сервер
dig google.com NS              # NS-записи (NS-серверы домена)
dig google.com MX              # MX-записи (почтовые серверы)

nslookup google.com            # простой запрос
host google.com                # короткий запрос
Файл /etc/hosts (локальные записи)
bash
cat /etc/hosts                 # формат: IP домен1 домен2
# Пример:
# 127.0.0.1 localhost
# 192.168.1.100 myserver.local

# Добавить локальную запись
echo "192.168.1.147 test.lab.local" | sudo tee -a /etc/hosts

# Проверить
ping test.lab.local
Systemd-resolved (современный DNS-резолвер)
bash
systemctl status systemd-resolved      # статус сервиса
resolvectl status                      # DNS-серверы, домены поиска
resolvectl statistics                  # статистика кэша
sudo resolvectl flush-caches           # очистить кэш DNS
resolvectl query google.com            # показать IP и интерфейс, через который был запрос
#Настройка DNS через Netplan (Ubuntu)
yaml
# /etc/netplan/00-installer-config.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: yes
      dhcp4-overrides:
        use-dns: no               # игнорировать DNS от DHCP
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
        search:
          - lab.local
Применить: sudo netplan apply

#Поднятие локального DNS-сервера (dnsmasq)
bash
# Установка
sudo apt install dnsmasq -y

# Конфиг /etc/dnsmasq.d/lab.conf
# Слушать локально
interface=lo
bind-interfaces
# Не использовать /etc/resolv.conf
no-resolv
# Внешние DNS
server=8.8.8.8
server=8.8.4.4
# Локальная зона
domain=lab.local
# Статические A-записи
address=/test.lab.local/192.168.1.147
address=/api.lab.local/192.168.1.147
# CNAME (алиас)
cname=www.lab.local,test.lab.local

# Запуск
sudo systemctl restart dnsmasq
sudo systemctl enable dnsmasq

# Проверка
dig @127.0.0.1 test.lab.local
Порядок резолвинга (приоритет)
/etc/hosts (высший приоритет)

systemd-resolved (если активен) или указанные DNS-серверы

файл /etc/resolv.conf (если не symlink)

#Диагностика DNS-проблем
bash
# Проверить, отвечает ли DNS-сервер
dig @8.8.8.8 google.com +tcp

# Трассировка DNS-запроса (показать полный путь)
dig +trace google.com

# Проверить, что система резолвит
getent hosts google.com

# Посмотреть, какие DNS-запросы делает система (требует sudo)
sudo tcpdump -i eth0 port 53 -n
Очистка DNS-кэша в разных системах
bash
# systemd-resolved
sudo resolvectl flush-caches

# dnsmasq
sudo systemctl restart dnsmasq

# nscd (если используется)
sudo systemctl restart nscd
#Часто используемые публичные DNS-серверы
Провайдер	DNS-серверы
Google	8.8.8.8, 8.8.4.4
Cloudflare	1.1.1.1, 1.0.0.1
OpenDNS	208.67.222.222, 208.67.220.220
Quad9	9.9.9.9, 149.112.112.112
#Краткая шпаргалка по DNS
Команда	Назначение
cat /etc/resolv.conf	DNS-серверы системы
resolvectl status	Статус systemd-resolved
dig domain	DNS-запрос (детально)
nslookup domain	DNS-запрос (просто)
dig -x IP	Обратный lookup
dig @server domain	Запрос к конкретному DNS
sudo resolvectl flush-caches	Очистить кэш
sudo systemctl restart systemd-resolved	Перезапустить резолвер
