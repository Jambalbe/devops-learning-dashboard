# Анализ трафика с tcpdump

## Установка
```bash
sudo apt install tcpdump -y   # Debian/Ubuntu
sudo yum install tcpdump -y   # CentOS/RHEL
#Базовый захват пакетов
bash
sudo tcpdump -i eth0                     # захват на интерфейсе eth0
sudo tcpdump -i eth0 -c 10               # захватить 10 пакетов и остановиться
sudo tcpdump -i eth0 -n                  # не преобразовывать IP в имена (быстрее)
sudo tcpdump -i eth0 -nn                 # не преобразовывать IP и порты
#Фильтры по протоколу и порту
bash
sudo tcpdump -i eth0 tcp                  # только TCP
sudo tcpdump -i eth0 udp                  # только UDP
sudo tcpdump -i eth0 icmp                 # только ICMP (ping)
sudo tcpdump -i eth0 port 80              # трафик на порт 80 (в любом направлении)
sudo tcpdump -i eth0 src port 443         # только исходящие пакеты с порта 443
sudo tcpdump -i eth0 dst port 22          # только входящие на порт 22
#Фильтры по IP-адресам
bash
sudo tcpdump -i eth0 host 192.168.1.100              # все пакеты с/на 192.168.1.100
sudo tcpdump -i eth0 src 192.168.1.100               # только от этого IP
sudo tcpdump -i eth0 dst 8.8.8.8                     # только на этот IP
sudo tcpdump -i eth0 net 192.168.1.0/24              # целая подсеть
#Комбинированные фильтры (and, or, not)
bash
sudo tcpdump -i eth0 host 192.168.1.100 and port 22          # SSH с конкретного IP
sudo tcpdump -i eth0 port 80 or port 443                      # HTTP или HTTPS
sudo tcpdump -i eth0 not port 22 and not port 80              # исключая SSH и HTTP
sudo tcpdump -i eth0 host 192.168.1.100 and not port 22       # всё, кроме SSH
#Сохранение и чтение дампов
bash
sudo tcpdump -i eth0 -w capture.pcap          # записать в файл (бинарный)
sudo tcpdump -r capture.pcap                  # прочитать файл
sudo tcpdump -r capture.pcap -n | head -20    # читать и показывать первые 20 строк
sudo tcpdump -r capture.pcap port 443         # фильтровать при чтении
#Детальный вывод
bash
sudo tcpdump -i eth0 -v                       # verbose (TTL, TOS, заголовки)
sudo tcpdump -i eth0 -vv                      # более детально
sudo tcpdump -i eth0 -e                       # показывать MAC-адреса (link-level)
sudo tcpdump -i eth0 -A                       # печатать payload как ASCII (для HTTP)
sudo tcpdump -i eth0 -X                       # печатать в hex и ASCII
#Лимитирование размера пакета
bash
sudo tcpdump -i eth0 -s 96                   # захватывать только первые 96 байт пакета
sudo tcpdump -i eth0 -s 0                    # захватывать весь пакет (по умолчанию 262144)
#Таймстампы
bash
sudo tcpdump -i eth0 -t                      # не показывать время
sudo tcpdump -i eth0 -tt                     # вывод времени в формате Unix timestamp
sudo tcpdump -i eth0 -tttt                   # читаемый формат (YYYY-MM-DD HH:MM:SS)
#Примеры для практики
Захват TCP handshake
bash
sudo tcpdump -i lo port 443 -c 3 -S          # флаг -S показывает абсолютные номера последовательностей
# В другом терминале: curl -k https://localhost
#Захват HTTP запроса
bash
sudo tcpdump -i lo port 80 -A -c 10
# В другом терминале: curl http://localhost
#Мониторинг всех подключений к SSH
bash
sudo tcpdump -i eth0 port 22 -n
#Поиск DDoS атаки (много SYN пакетов)
bash
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack == 0' -c 100
#Проверка DNS запросов
bash
sudo tcpdump -i eth0 port 53 -n
#Чтение TCP флагов в выводе
text
[S]  = SYN      (начало соединения)
[.]  = ACK      (подтверждение)
[P]  = PUSH     (данные отправлены сразу)
[F]  = FIN      (закрытие соединения)
[R]  = RST      (сброс соединения, ошибка)
[S.] = SYN-ACK  (ответ на SYN)
[F.] = FIN-ACK  (закрытие с подтверждением)
#Расшифровка строки tcpdump
text
08:57:22.438301 IP 192.168.1.92.54280 > ubuntuserver.http: Flags [S], seq 4002233546
Поле	Значение
08:57:22.438301	Время
IP	Протокол
192.168.1.92.54280	Источник (IP.порт)
>	Направление (от источника к получателю)
ubuntuserver.http	Получатель (хост.порт)
Flags [S]	SYN-флаг
seq 4002233546	Номер последовательности
#Полезные опции tcpdump
Опция	Назначение
-i eth0	Интерфейс
-c N	Остановиться после N пакетов
-n	Не резолвить DNS
-nn	Не резолвить DNS и порты
-s 0	Весь пакет (не обрезать)
-w file	Записать в файл
-r file	Прочитать из файла
-A	ASCII (текст)
-X	Hex + ASCII
-v, -vv, -vvv	Уровень детализации
-e	MAC-адреса
-t	Без таймстампов
-q	Краткий вывод
