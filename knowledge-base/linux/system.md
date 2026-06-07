# Systemd и управление сервисами

## Что такое systemd

**systemd** — это система инициализации и управления сервисами в современных Linux-дистрибутивах.

Основные задачи:

* запуск системы при загрузке;
* управление сервисами и демонами;
* сбор логов через `journald`;
* управление таймерами;
* управление сокетами и зависимостями между сервисами.

Проверить версию:

```bash id="w5t6g1"
systemctl --version
```

Проверить, используется ли systemd:

```bash id="d8x4c2"
ps -p 1
```

Обычно PID 1 принадлежит процессу:

```text id="n2j9q7"
systemd
```

---

# Управление сервисами

## Запуск сервиса

```bash id="q7h4m8"
sudo systemctl start service_name
```

---

## Остановка сервиса

```bash id="y4v8k3"
sudo systemctl stop service_name
```

---

## Перезапуск сервиса

```bash id="r9m1t6"
sudo systemctl restart service_name
```

Полезно после изменения конфигурации.

---

## Перечитать конфигурацию без остановки

```bash id="a3f7w9"
sudo systemctl reload service_name
```

Поддерживается не всеми сервисами.

Примеры:

* nginx
* apache2
* sshd

---

## Проверить статус

```bash id="u8p2n5"
sudo systemctl status service_name
```

Пример:

```text id="g5r2c1"
● nginx.service - A high performance web server
   Loaded: loaded
   Active: active (running)
```

---

# Автозапуск сервисов

## Включить автозапуск

```bash id="j6x1k4"
sudo systemctl enable service_name
```

Сервис будет запускаться после загрузки системы.

---

## Отключить автозапуск

```bash id="m9c3v7"
sudo systemctl disable service_name
```

---

## Проверить автозапуск

```bash id="p4z8r2"
systemctl is-enabled service_name
```

Возможные результаты:

```text id="l1q6s5"
enabled
disabled
static
masked
```

---

# Просмотр сервисов

## Активные сервисы

```bash id="e7n5u3"
systemctl list-units --type=service
```

---

## Все сервисы

```bash id="h2k8w6"
systemctl list-units --type=service --all
```

---

## Все unit-файлы

```bash id="b5r4t9"
systemctl list-unit-files --type=service
```

Пример:

```text id="v8m3n1"
nginx.service     enabled
ssh.service       enabled
docker.service    disabled
```

---

# Состояние systemd

## Перечитать unit-файлы

После изменения или создания сервиса:

```bash id="s7d4f2"
sudo systemctl daemon-reload
```

---

## Перезагрузка системы

```bash id="k3p8v5"
sudo systemctl reboot
```

---

## Выключение

```bash id="n6c2x7"
sudo systemctl poweroff
```

---

## Спящий режим

```bash id="w9m5r4"
sudo systemctl suspend
```

---

# Логи через journalctl

## Все логи

```bash id="q2f7t8"
journalctl
```

---

## Логи конкретного сервиса

```bash id="c4j9u1"
journalctl -u service_name
```

---

## Последние 50 строк

```bash id="r1x6m3"
journalctl -n 50
```

---

## Следить за логами в реальном времени

```bash id="v8p4s7"
journalctl -f
```

Аналог:

```bash id="m3k7q2"
tail -f
```

---

## Логи за последний час

```bash id="h5w1t9"
journalctl --since "1 hour ago"
```

---

## Логи за период времени

```bash id="x7c2r4"
journalctl \
--since "2026-06-07 10:00:00" \
--until "2026-06-07 11:00:00"
```

---

## Только логи ядра

```bash id="d4n8y6"
journalctl -k
```

---

## Только ошибки

```bash id="u1v5j3"
journalctl -p err
```

Уровни логирования:

| Уровень | Значение           |
| ------- | ------------------ |
| emerg   | Авария             |
| alert   | Срочное сообщение  |
| crit    | Критическая ошибка |
| err     | Ошибка             |
| warning | Предупреждение     |
| notice  | Важное уведомление |
| info    | Информация         |
| debug   | Отладка            |

---

# Анализ логов сервиса

## Следить за логами nginx

```bash id="g8f2m5"
journalctl -u nginx -f
```

---

## Логи за сегодня

```bash id="z4k7r8"
journalctl -u nginx --since today
```

---

## Вывод в формате JSON

```bash id="p6n3w1"
journalctl -u nginx -o json-pretty
```

Полезно для автоматизации и скриптов.

---

## Поиск ошибок

```bash id="t9v5c2"
journalctl -u nginx | grep error
```

---

# Юниты systemd

Сервисы — лишь один тип unit-файлов.

---

## Targets

Аналог уровней запуска (runlevels).

```bash id="a7r4m6"
systemctl list-units --type=target
```

---

## Socket Units

```bash id="n2v9k4"
systemctl list-units --type=socket
```

Используются для socket activation.

---

## Path Units

```bash id="c8w3t7"
systemctl list-units --type=path
```

Позволяют запускать сервис при изменении файла или каталога.

---

## Timers

Аналог cron.

```bash id="y5p1j8"
systemctl list-units --type=timer
```

---

# Создание собственного сервиса

## Создать unit-файл

```bash id="m4t8n2"
sudo nano /etc/systemd/system/myapp.service
```

---

## Содержимое сервиса

```ini id="k7f5x9"
[Unit]
Description=My custom app
After=network.target

[Service]
ExecStart=/usr/bin/python3 /opt/myapp/app.py
Restart=always
User=myuser
Group=mygroup

[Install]
WantedBy=multi-user.target
```

---

## Перечитать конфигурацию

```bash id="r3w7c6"
sudo systemctl daemon-reload
```

---

## Включить автозапуск

```bash id="q8n2v5"
sudo systemctl enable myapp.service
```

---

## Запустить сервис

```bash id="f6m4t1"
sudo systemctl start myapp.service
```

---

## Проверить работу

```bash id="u3k9r7"
systemctl status myapp.service
```

---

# Полезные параметры Service

## Автоматический перезапуск

```ini id="b2x8n4"
Restart=always
```

Варианты:

| Значение    | Поведение                |
| ----------- | ------------------------ |
| no          | Не перезапускать         |
| on-failure  | Только при ошибке        |
| always      | Всегда                   |
| on-abnormal | При аварийном завершении |

---

## Рабочий каталог

```ini id="g5v3r8"
WorkingDirectory=/opt/myapp
```

---

## Переменные окружения

```ini id="j1t7k6"
Environment="VAR1=value1"
Environment="VAR2=value2"
```

---

## Файл с переменными

```ini id="s9n4w2"
EnvironmentFile=/etc/myapp/env.conf
```

---

# Таймеры systemd

Таймеры являются современной заменой cron.

---

## Создать таймер

```bash id="v7p5m3"
sudo nano /etc/systemd/system/myapp.timer
```

---

## Содержимое

```ini id="c4k8r1"
[Unit]
Description=Daily timer for myapp

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

---

## Активировать таймер

```bash id="w1n6t4"
sudo systemctl enable myapp.timer
sudo systemctl start myapp.timer
```

---

## Показать таймеры

```bash id="m8v2r7"
systemctl list-timers
```

---

## Примеры расписаний

| Значение             | Описание                   |
| -------------------- | -------------------------- |
| `daily`              | Каждый день                |
| `weekly`             | Каждую неделю              |
| `hourly`             | Каждый час                 |
| `*:0/15`             | Каждые 15 минут            |
| `Mon *-*-* 03:00:00` | Каждый понедельник в 03:00 |

---

# Маскирование сервисов

Полностью запрещает запуск сервиса.

---

## Заблокировать сервис

```bash id="h6p3k8"
sudo systemctl mask service_name
```

Создаётся ссылка:

```text id="n5x1v4"
/dev/null
```

---

## Разблокировать

```bash id="q4t7r9"
sudo systemctl unmask service_name
```

---

# Проверка зависимостей

## Что требуется сервису

```bash id="u7m2w5"
systemctl list-dependencies service_name
```

---

## Обратные зависимости

```bash id="r1n8k6"
systemctl list-dependencies --reverse service_name
```

---

# Полезные команды диагностики

## Проверить наличие ошибок в unit-файле

```bash id="x8v4r2"
systemd-analyze verify myapp.service
```

---

## Время загрузки системы

```bash id="j3n7t5"
systemd-analyze
```

---

## Какие сервисы тормозят загрузку

```bash id="p6w1k9"
systemd-analyze blame
```

---

# Полезные алиасы

Добавить в:

```bash id="v9m5c1"
~/.bashrc
```

---

## Быстрый доступ к systemctl

```bash id="g4n8r6"
alias sc='sudo systemctl'
```

---

## Просмотр логов сервиса

```bash id="k2t7v3"
alias jc='journalctl -u'
```

---

## Следить за логами сервиса

```bash id="r5m9w8"
alias jcf='journalctl -u -f'
```

---

Применить изменения:

```bash id="d7p4k2"
source ~/.bashrc
```

---

# Шпаргалка

| Команда                   | Назначение            |
| ------------------------- | --------------------- |
| `systemctl start`         | Запустить сервис      |
| `systemctl stop`          | Остановить сервис     |
| `systemctl restart`       | Перезапустить         |
| `systemctl reload`        | Перечитать конфиг     |
| `systemctl status`        | Проверить состояние   |
| `systemctl enable`        | Включить автозапуск   |
| `systemctl disable`       | Отключить автозапуск  |
| `systemctl daemon-reload` | Перечитать unit-файлы |
| `journalctl -u name`      | Логи сервиса          |
| `journalctl -f`           | Следить за логами     |
| `systemctl list-timers`   | Список таймеров       |
| `systemctl mask`          | Запретить запуск      |
| `systemctl unmask`        | Разрешить запуск      |

---

# Запомнить

### Проверить сервис

```bash id="t2v6m8"
systemctl status nginx
```

### Перезапустить сервис

```bash id="n7w3r5"
sudo systemctl restart nginx
```

### Посмотреть логи

```bash id="f4k8p1"
journalctl -u nginx -f
```

### Включить автозапуск

```bash id="m1r9v6"
sudo systemctl enable nginx
```

### Перечитать unit-файлы

```bash id="q5t2n7"
sudo systemctl daemon-reload
```
