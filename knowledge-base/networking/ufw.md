# UFW — Uncomplicated Firewall (простой фаервол)

## Что такое UFW

**UFW (Uncomplicated Firewall)** — удобная обёртка над `iptables`, которая упрощает настройку фаервола в Linux.

По умолчанию установлен в большинстве версий Ubuntu.

---

# Основные команды

## Просмотр состояния

```bash
sudo ufw status             # статус и список правил
sudo ufw status verbose     # подробная информация
sudo ufw status numbered    # правила с номерами
sudo ufw show added         # только пользовательские правила
```

---

## Включение и выключение

```bash
sudo ufw enable             # включить фаервол
sudo ufw disable            # выключить фаервол
sudo ufw reset              # удалить все правила и отключить UFW
```

---

# Разрешение портов

## По номеру порта

```bash
sudo ufw allow 22
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

### Пояснение

| Команда | Назначение |
|----------|------------|
| `allow 22` | Разрешить TCP и UDP на порт 22 |
| `allow 80/tcp` | Разрешить HTTP только по TCP |
| `allow 443/tcp` | Разрешить HTTPS |

---

## По имени сервиса

```bash
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
```

UFW использует список сервисов из файла:

```bash
/etc/services
```

---

# Разрешение доступа по IP

## Разрешить всё с конкретного IP

```bash
sudo ufw allow from 192.168.1.50
```

## Разрешить подсеть

```bash
sudo ufw allow from 192.168.1.0/24
```

## Разрешить только SSH с конкретного IP

```bash
sudo ufw allow from 192.168.1.50 to any port 22
```

---

# Работа с интерфейсами

```bash
sudo ufw allow in on eth0 to any port 80
sudo ufw allow out on eth1 to any port 443
```

| Команда | Назначение |
|----------|------------|
| `in on eth0` | Входящий трафик через интерфейс |
| `out on eth1` | Исходящий трафик через интерфейс |

---

# Запрет правил

```bash
sudo ufw deny 23
sudo ufw deny 3306/tcp
sudo ufw deny from 10.0.0.100
```

---

# Удаление правил

## По номеру

```bash
sudo ufw status numbered
sudo ufw delete 2
```

## По содержимому

```bash
sudo ufw delete allow 80/tcp
sudo ufw delete deny from 10.0.0.100
```

---

# Политики по умолчанию

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw default deny routed
```

### Рекомендуемая конфигурация сервера

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

---

# Логирование

## Включение

```bash
sudo ufw logging on
```

## Выключение

```bash
sudo ufw logging off
```

## Уровни логирования

```bash
sudo ufw logging low
sudo ufw logging medium
sudo ufw logging high
```

| Уровень | Описание |
|----------|------------|
| low | Минимальный объём логов |
| medium | Средний уровень |
| high | Подробное логирование |

---

# Просмотр логов

```bash
sudo tail -f /var/log/ufw.log
```

Показать последние блокировки:

```bash
sudo grep "UFW BLOCK" /var/log/ufw.log | tail -20
```

Через systemd journal:

```bash
sudo journalctl | grep UFW | tail -20
```

---

# Расширенные правила

## Разрешить TCP только из подсети

```bash
sudo ufw allow proto tcp from 192.168.1.0/24 to any port 22
```

## Разрешить диапазон портов

```bash
sudo ufw allow 8000:9000/tcp
```

## Добавить комментарий

```bash
sudo ufw allow 8080/tcp comment "Tomcat server"
```

## Использовать REJECT вместо DROP

```bash
sudo ufw reject 8080/tcp
```

### Разница между DROP и REJECT

| Действие | Что происходит |
|-----------|----------------|
| DROP | Пакет просто игнорируется |
| REJECT | Отправляется сообщение об отказе |

---

# Защита SSH от брутфорса

```bash
sudo ufw limit ssh
```

Ограничивает количество попыток подключения.

Пример:

```bash
sudo ufw limit ssh
sudo ufw allow from 192.168.1.100 to any port 22
```

---

# Типовые конфигурации

## Веб-сервер (HTTP + HTTPS + SSH)

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

sudo ufw enable
```

---

## Сервер с ограниченным доступом

```bash
sudo ufw default deny incoming

sudo ufw allow from 192.168.1.0/24 to any port 22
sudo ufw allow from 10.0.0.5 to any port 3306

sudo ufw enable
```

---

## Защищённый SSH

```bash
sudo ufw limit ssh
sudo ufw allow from 192.168.1.100 to any port 22
```

---

# Сброс всей конфигурации

```bash
sudo ufw reset
```

После подтверждения:

```text
yes
```

- все правила будут удалены;
- UFW будет отключён.

---

# Как UFW связан с iptables

UFW не является отдельным фаерволом.

Он автоматически создаёт правила в `iptables`.

Посмотреть правила:

```bash
sudo iptables -L -n
```

Посмотреть пользовательские правила UFW:

```bash
sudo iptables -L ufw-user-input -n
```

---

# Полезные команды диагностики

## Проверить открытые порты

```bash
ss -tulnp
```

или

```bash
sudo netstat -tulnp
```

## Проверить доступность порта с другого хоста

```bash
nc -zv SERVER_IP 22
```

или

```bash
telnet SERVER_IP 22
```

---

# Краткая шпаргалка

| Действие | Команда |
|-----------|----------|
| Включить UFW | `sudo ufw enable` |
| Выключить UFW | `sudo ufw disable` |
| Статус | `sudo ufw status` |
| Разрешить HTTP | `sudo ufw allow 80/tcp` |
| Разрешить HTTPS | `sudo ufw allow 443/tcp` |
| Разрешить SSH | `sudo ufw allow ssh` |
| Запретить порт | `sudo ufw deny 8080` |
| Разрешить IP | `sudo ufw allow from 10.0.0.1` |
| Удалить правило №3 | `sudo ufw delete 3` |
| Защита SSH | `sudo ufw limit ssh` |
| Сбросить настройки | `sudo ufw reset` |
| Включить логирование | `sudo ufw logging on` |
| Смотреть логи | `sudo tail -f /var/log/ufw.log` |

---

# Полезные замечания

⚠️ Перед включением UFW на удалённом сервере обязательно разрешите SSH:

```bash
sudo ufw allow ssh
sudo ufw enable
```

Иначе можно потерять доступ к серверу.

Проверить правила перед включением:

```bash
sudo ufw status verbose
```

Проверить конфигурацию после включения:

```bash
sudo ufw status numbered
```
