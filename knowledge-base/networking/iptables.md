# iptables — продвинутый фаервол

## Основные цепочки

| Цепочка   | Назначение              |
| --------- | ----------------------- |
| `INPUT`   | Входящие пакеты         |
| `OUTPUT`  | Исходящие пакеты        |
| `FORWARD` | Маршрутизируемые пакеты |

---

## Таблицы

| Таблица  | Назначение                                              |
| -------- | ------------------------------------------------------- |
| `filter` | Разрешение и запрет трафика (используется по умолчанию) |
| `nat`    | Преобразование сетевых адресов (SNAT/DNAT)              |
| `mangle` | Изменение параметров пакетов (TTL, TOS и др.)           |

---

## Просмотр правил

```bash
# Список правил со счетчиками
sudo iptables -L -v -n

# Правила в формате команд
sudo iptables -S

# Правила NAT
sudo iptables -t nat -L -n
```

---

## Добавление и удаление правил

```bash
# Добавить правило
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Вставить правило в начало цепочки
sudo iptables -I INPUT 1 -s 10.0.0.5 -j DROP

# Удалить правило по номеру
sudo iptables -D INPUT 2
```

---

## Политики по умолчанию

```bash
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD DROP
```

---

## Разрешение установленных соединений

> Важно: это правило рекомендуется добавлять одним из первых, иначе ответы на уже установленные соединения могут блокироваться.

```bash
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

---

## Пример: разрешить только SSH и HTTP

```bash
# Очистить существующие правила
sudo iptables -F

# Запретить весь входящий трафик по умолчанию
sudo iptables -P INPUT DROP

# Разрешить loopback-интерфейс
sudo iptables -A INPUT -i lo -j ACCEPT

# Разрешить установленные соединения
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Разрешить SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Разрешить HTTP
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

---

## Логирование

```bash
# Логировать отброшенные пакеты
sudo iptables -I INPUT 1 -m limit --limit 5/min -j LOG --log-prefix "DROP: "
```

Просмотр логов:

```bash
sudo journalctl -kf | grep DROP
```

---

## Сохранение правил

Установка пакета:

```bash
sudo apt install iptables-persistent
```

Сохранение текущей конфигурации:

```bash
sudo netfilter-persistent save
```

---

## NAT (Source NAT)

### Включение IP Forwarding

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

### Маскарадинг

```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

---

## DNAT (Проброс портов)

Перенаправление входящих подключений с порта `8080` на веб-сервер `192.168.1.100:80`:

```bash
sudo iptables -t nat -A PREROUTING \
-p tcp --dport 8080 \
-j DNAT --to-destination 192.168.1.100:80
```

---

## Удаление всех правил

```bash
# Очистить правила filter
sudo iptables -F

# Удалить пользовательские цепочки
sudo iptables -X

# Очистить правила NAT
sudo iptables -t nat -F
```

---

## Полезные флаги

| Флаг               | Назначение                                   |
| ------------------ | -------------------------------------------- |
| `-A`               | Append — добавить в конец                    |
| `-I`               | Insert — вставить в начало                   |
| `-D`               | Delete — удалить правило                     |
| `-L`               | List — показать список правил                |
| `-F`               | Flush — очистить правила                     |
| `-p`               | Протокол (`tcp`, `udp`, `icmp`)              |
| `--dport`          | Порт назначения                              |
| `-s`               | IP-адрес источника                           |
| `-d`               | IP-адрес назначения                          |
| `-i`               | Входящий интерфейс                           |
| `-o`               | Исходящий интерфейс                          |
| `-j`               | Действие (`ACCEPT`, `DROP`, `LOG`, `REJECT`) |
| `-m state --state` | Состояние соединения                         |

---

## Часто используемые действия

| Действие     | Описание                                            |
| ------------ | --------------------------------------------------- |
| `ACCEPT`     | Разрешить пакет                                     |
| `DROP`       | Отбросить пакет без ответа                          |
| `REJECT`     | Отклонить пакет и отправить уведомление отправителю |
| `LOG`        | Записать информацию о пакете в лог                  |
| `DNAT`       | Изменить адрес назначения                           |
| `SNAT`       | Изменить адрес источника                            |
| `MASQUERADE` | Автоматический SNAT для динамических IP             |
|              |                                                     |
