# Systemd и управление сервисами

## Основные команды для сервисов
```bash
sudo systemctl start service_name     # запустить сервис
sudo systemctl stop service_name      # остановить сервис
sudo systemctl restart service_name   # перезапустить сервис
sudo systemctl reload service_name    # перечитать конфиг (без остановки)
sudo systemctl status service_name    # показать статус и последние логи
#Включение/отключение автозапуска
bash
sudo systemctl enable service_name    # включить автозапуск при загрузке
sudo systemctl disable service_name   # отключить автозапуск
sudo systemctl is-enabled service_name # проверить, включен ли автозапуск
#Просмотр сервисов
bash
systemctl list-units --type=service           # все активные сервисы
systemctl list-units --type=service --all     # все сервисы (включая неактивные)
systemctl list-unit-files --type=service      # все файлы сервисов с их статусом
#Управление systemd (общее состояние)
bash
systemctl daemon-reload              # перечитать все unit-файлы (после создания/изменения)
systemctl reboot                     # перезагрузка
systemctl poweroff                   # выключение
systemctl suspend                    # спящий режим
#Просмотр логов (journalctl)
bash
journalctl                          # все логи
journalctl -u service_name          # логи конкретного сервиса
journalctl -f                       # следовать за логами (tail -f)
journalctl -n 50                    # последние 50 строк
journalctl --since "1 hour ago"     # логи за последний час
journalctl --since "2026-06-07 10:00:00" --until "2026-06-07 11:00:00"
journalctl -k                       # только логи ядра
journalctl -p err                   # только ошибки (emerg, alert, crit, err)
#Анализ логов (фильтрация)
bash
journalctl -u nginx -f                           # следить за логами nginx
journalctl -u nginx --since today                # логи nginx за сегодня
journalctl -u nginx -o json-pretty               # вывод в JSON (для скриптов)
journalctl -u nginx | grep "error"               # через grep
#Юниты (не только сервисы)
bash
systemctl list-units --type=target               # цели (targets)
systemctl list-units --type=socket               # сокеты
systemctl list-units --type=path                 # пути (активация по файлу)
systemctl list-units --type=timer                # таймеры (аналог cron)
#Создание простого сервиса (пример)
bash
# 1. Создать файл /etc/systemd/system/myapp.service
sudo nano /etc/systemd/system/myapp.service
Содержимое:

ini
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
bash
# 2. Перечитать конфигурацию
sudo systemctl daemon-reload

# 3. Включить и запустить
sudo systemctl enable myapp.service
sudo systemctl start myapp.service
Таймеры (замена cron)
bash
# Пример: выполнять скрипт каждый день в 3:00
# /etc/systemd/system/myapp.timer
[Unit]
Description=Daily timer for myapp

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
bash
# Команды для таймеров
systemctl list-timers                         # показать активные таймеры
sudo systemctl enable myapp.timer
sudo systemctl start myapp.timer
#Маскирование сервисов (запрет запуска)
bash
sudo systemctl mask service_name     # полностью заблокировать сервис (создаёт симлинк на /dev/null)
sudo systemctl unmask service_name   # разблокировать
#Переменные окружения для сервиса
bash
# Добавить в файл сервиса:
Environment="VAR1=value1" "VAR2=value2"
EnvironmentFile=/etc/myapp/env.conf
Краткая шпаргалка
Команда	Назначение
systemctl start	запустить сейчас
systemctl stop	остановить
systemctl restart	перезапустить
systemctl reload	перечитать конфиг (без остановки)
systemctl enable	добавить в автозагрузку
systemctl disable	убрать из автозагрузки
systemctl status	проверить статус и последние логи
systemctl daemon-reload	перечитать все unit файлы
journalctl -u name	посмотреть логи сервиса
journalctl -f	следить за всеми логами
#Полезные алиасы
bash
alias sc='sudo systemctl'
alias jc='journalctl -u'
alias jcf='journalctl -u -f'
