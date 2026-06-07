# Анализ трафика с tcpdump

## Что такое tcpdump

**tcpdump** — консольный анализатор сетевого трафика для Linux и Unix-систем.

Позволяет:

* перехватывать пакеты в реальном времени;
* анализировать сетевые соединения;
* искать проблемы сети;
* сохранять трафик для последующего анализа в Wireshark;
* расследовать сетевые инциденты и атаки.

---

# Установка

## Debian / Ubuntu

```bash
sudo apt install tcpdump -y
```

## CentOS / RHEL

```bash
sudo yum install tcpdump -y
```

Проверка установки:

```bash
tcpdump --version
```

---

# Базовый захват пакетов

## Захват на интерфейсе

```bash
sudo tcpdump -i eth0
```

---

## Захватить только 10 пакетов

```bash
sudo tcpdump -i eth0 -c 10
```

---

## Не выполнять DNS-резолвинг

```bash
sudo tcpdump -i eth0 -n
```

Работает быстрее и показывает IP-адреса вместо доменных имён.

---

## Не резолвить DNS и порты

```bash
sudo tcpdump -i eth0 -nn
```

Пример:

```text
192.168.1.100.443 > 192.168.1.50.51342
```

вместо:

```text
google.com.https > client.random_port
```

---

# Фильтрация по протоколам

## Только TCP

```bash
sudo tcpdump -i eth0 tcp
```

---

## Только UDP

```bash
sudo tcpdump -i eth0 udp
```

---

## Только ICMP (ping)

```bash
sudo tcpdump -i eth0 icmp
```

---

# Фильтрация по портам

## Любой трафик на порт 80

```bash
sudo tcpdump -i eth0 port 80
```

---

## Исходящие пакеты с порта 443

```bash
sudo tcpdump -i eth0 src port 443
```

---

## Входящие пакеты на порт 22

```bash
sudo tcpdump -i eth0 dst port 22
```

---

# Фильтрация по IP-адресам

## Весь трафик для узла

```bash
sudo tcpdump -i eth0 host 192.168.1.100
```

---

## Только источник

```bash
sudo tcpdump -i eth0 src 192.168.1.100
```

---

## Только получатель

```bash
sudo tcpdump -i eth0 dst 8.8.8.8
```

---

## Целая подсеть

```bash
sudo tcpdump -i eth0 net 192.168.1.0/24
```

---

# Комбинирование фильтров

## SSH от конкретного IP

```bash
sudo tcpdump -i eth0 \
host 192.168.1.100 and port 22
```

---

## HTTP или HTTPS

```bash
sudo tcpdump -i eth0 \
port 80 or port 443
```

---

## Исключить SSH и HTTP

```bash
sudo tcpdump -i eth0 \
not port 22 and not port 80
```

---

## Всё кроме SSH от конкретного хоста

```bash
sudo tcpdump -i eth0 \
host 192.168.1.100 and not port 22
```

---

# Сохранение и чтение дампов

## Запись в файл

```bash
sudo tcpdump -i eth0 -w capture.pcap
```

Формат `.pcap` используется Wireshark и другими анализаторами.

---

## Чтение дампа

```bash
sudo tcpdump -r capture.pcap
```

---

## Первые строки дампа

```bash
sudo tcpdump -r capture.pcap -n | head -20
```

---

## Фильтрация при чтении

```bash
sudo tcpdump -r capture.pcap port 443
```

---

# Детальный вывод

## Подробный режим

```bash
sudo tcpdump -i eth0 -v
```

Показывает:

* TTL
* TOS
* длину пакета
* дополнительные заголовки

---

## Ещё больше информации

```bash
sudo tcpdump -i eth0 -vv
```

---

## Максимальная детализация

```bash
sudo tcpdump -i eth0 -vvv
```

---

## MAC-адреса

```bash
sudo tcpdump -i eth0 -e
```

---

## Вывод полезной нагрузки как текста

```bash
sudo tcpdump -i eth0 -A
```

Полезно для:

* HTTP
* SMTP
* FTP
* других незашифрованных протоколов

---

## Hex + ASCII

```bash
sudo tcpdump -i eth0 -X
```

Пример вывода:

```text
0x0000: 4500 003c ...
GET /index.html HTTP/1.1
```

---

# Ограничение размера захватываемого пакета

## Первые 96 байт

```bash
sudo tcpdump -i eth0 -s 96
```

---

## Полный пакет

```bash
sudo tcpdump -i eth0 -s 0
```

---

# Управление временными метками

## Без времени

```bash
sudo tcpdump -i eth0 -t
```

---

## Unix Timestamp

```bash
sudo tcpdump -i eth0 -tt
```

---

## Полная дата и время

```bash
sudo tcpdump -i eth0 -tttt
```

Пример:

```text
2025-04-14 18:15:21.123456
```

---

# Практические примеры

## Захват TCP Handshake

```bash
sudo tcpdump -i lo \
port 443 \
-c 3 \
-S
```

Параметр:

```text
-S
```

показывает абсолютные номера последовательностей TCP.

Во втором терминале:

```bash
curl -k https://localhost
```

---

## Захват HTTP-запроса

```bash
sudo tcpdump -i lo \
port 80 \
-A \
-c 10
```

Во втором терминале:

```bash
curl http://localhost
```

---

## Мониторинг SSH-подключений

```bash
sudo tcpdump -i eth0 \
port 22 \
-n
```

---

## Поиск SYN Flood / DDoS

Показать только SYN-пакеты:

```bash
sudo tcpdump -i eth0 \
'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack == 0' \
-c 100
```

Если SYN-пакетов слишком много — возможна атака типа **SYN Flood**.

---

## Мониторинг DNS-запросов

```bash
sudo tcpdump -i eth0 \
port 53 \
-n
```

---

# TCP-флаги

| Флаг   | Назначение                         |
| ------ | ---------------------------------- |
| `[S]`  | SYN — начало соединения            |
| `[S.]` | SYN-ACK                            |
| `[.]`  | ACK — подтверждение                |
| `[P]`  | PUSH — немедленная передача данных |
| `[F]`  | FIN — завершение соединения        |
| `[F.]` | FIN-ACK                            |
| `[R]`  | RST — сброс соединения             |

---

# Разбор строки tcpdump

Пример:

```text
08:57:22.438301 IP 192.168.1.92.54280 > ubuntuserver.http: Flags [S], seq 4002233546
```

| Поле                 | Значение                 |
| -------------------- | ------------------------ |
| `08:57:22.438301`    | Время                    |
| `IP`                 | Протокол                 |
| `192.168.1.92.54280` | Источник (IP:порт)       |
| `>`                  | Направление передачи     |
| `ubuntuserver.http`  | Получатель               |
| `Flags [S]`          | SYN-флаг                 |
| `seq 4002233546`     | Номер последовательности |

---

# Полезные опции tcpdump

| Опция     | Назначение                      |
| --------- | ------------------------------- |
| `-i eth0` | Интерфейс для захвата           |
| `-c N`    | Остановиться после N пакетов    |
| `-n`      | Не выполнять DNS-резолвинг      |
| `-nn`     | Не резолвить DNS и порты        |
| `-s 0`    | Захватывать пакет полностью     |
| `-w file` | Сохранить дамп                  |
| `-r file` | Читать дамп                     |
| `-A`      | ASCII-представление             |
| `-X`      | Hex + ASCII                     |
| `-v`      | Подробный вывод                 |
| `-vv`     | Очень подробный вывод           |
| `-vvv`    | Максимальная детализация        |
| `-e`      | Показывать MAC-адреса           |
| `-t`      | Без времени                     |
| `-tt`     | Unix Timestamp                  |
| `-tttt`   | Полная дата и время             |
| `-q`      | Краткий вывод                   |
| `-S`      | Абсолютные TCP sequence numbers |

---

# Часто используемые команды

## Смотреть весь HTTP-трафик

```bash
sudo tcpdump -i eth0 port 80 -A
```

---

## Смотреть DNS-запросы

```bash
sudo tcpdump -i eth0 port 53 -n
```

---

## Смотреть SSH-соединения

```bash
sudo tcpdump -i eth0 port 22 -n
```

---

## Сохранить трафик для Wireshark

```bash
sudo tcpdump -i eth0 -w traffic.pcap
```

---

## Читать дамп

```bash
sudo tcpdump -r traffic.pcap
```

---

## Полный пакет без DNS

```bash
sudo tcpdump -i eth0 -nn -s 0
```

---

## Показать только SYN-пакеты

```bash
sudo tcpdump -i eth0 \
'tcp[tcpflags] & tcp-syn != 0'
```
