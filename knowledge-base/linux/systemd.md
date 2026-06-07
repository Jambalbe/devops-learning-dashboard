# Systemd — управление сервисами и системой

## Основные команды управления сервисами
```bash
sudo systemctl start service_name     # запустить сервис
sudo systemctl stop service_name      # остановить сервис
sudo systemctl restart service_name   # перезапустить сервис
sudo systemctl reload service_name    # перечитать конфигурацию без остановки
sudo systemctl status service_name    # показать статус (активен, логи ошибок)
#Включение/отключение автозапуска
bash
sudo systemctl enable service_name    # добавить в автозагрузку
sudo systemctl disable service_name   # убрать из автозагрузки
sudo systemctl is-enabled service_name # проверить, включен ли автозапуск
#Просмотр сервисов
bash
systemctl list-units --type=service           # только активные сервисы
systemctl list-units --type=service --all     # все сервисы (включая остановленные)
systemctl list-unit-files --type=service      # состояние автозапуска у всех сервисов
systemctl list-dependencies service_name      # зависимости сервиса
#Анализ логов (journalctl)
bash
journalctl -u service_name               # логи конкретного сервиса
journalctl -u service_name -f            # следить за логами в реальном времени
journalctl -u service_name -n 50         # последние 50 строк
journalctl --since "10 minutes ago"      # логи за последние 10 минут
journalctl --since today                 # логи с сегодняшнего дня
journalctl --until "2024-01-01"          # логи до указанной даты
#Работа с systemd unit-файлами
bash
/etc/systemd/system/                     # пользовательские unit-файлы (приоритет)
/lib/systemd/system/                     # системные unit-файлы (от пакетов)
sudo systemctl daemon-reload             # перечитать конфигурацию после изменений
sudo systemctl edit service_name         # создать оверлей (доп. конфиг)
sudo systemctl cat service_name          # показать полный unit-файл
#Пример простого unit-файла (/etc/systemd/system/myapp.service)
ini
[Unit]
Description=My custom application
After=network.target

[Service]
Type=simple
User=devops
WorkingDirectory=/home/devops/myapp
ExecStart=/usr/bin/python3 app.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
После создания файла:

bash
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
#Просмотр системной информации
bash
systemctl reboot                     # перезагрузка
systemctl poweroff                   # выключение
systemctl suspend                    # спящий режим
systemctl hibernate                  # гибернация
hostnamectl                          # информация о хосте (имя, ОС)
timedatectl                          # настройки времени и даты
Работа с таймерами (аналог cron)
bash
systemctl list-timers                # список таймеров
# Пример таймера: /etc/systemd/system/backup.timer
[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
Маскирование сервиса (запрет на запуск)
bash
sudo systemctl mask service_name     # полностью заблокировать сервис
sudo systemctl unmask service_name   # разблокировать
Переменные окружения для сервиса
bash
sudo mkdir -p /etc/systemd/system/service_name.service.d/
sudo nano /etc/systemd/system/service_name.service.d/override.conf
Содержимое:

ini
[Service]
Environment="MY_VAR=value"
EnvironmentFile=/etc/myapp/env.conf
#Часто используемые типы сервисов
Type	Описание
simple	Процесс не форкается, работает в foreground (по умолчанию)
forking	Процесс форкается и завершает родителя (традиционные демоны)
oneshot	Запускается один раз и завершается (без Restart)
notify	Сообщает о готовности через sd_notify()
dbus	Запускается по запросу через D-Bus
Полезные команды
bash
systemctl is-active service_name      # работает ли сервис?
systemctl is-failed service_name      # был ли сбой?
systemctl show service_name | grep LoadState   # показать параметры сервиса
#Журнал загрузки (boot log)
bash
journalctl -b                         # логи текущей загрузки
journalctl -b -1                      # логи предыдущей загрузки
journalctl -b -p err                  # только ошибки текущей загрузки
#Краткая шпаргалка по systemctl
Команда	Действие
systemctl start	Запустить сейчас
systemctl stop	Остановить
systemctl restart	Перезапустить
systemctl reload	Перечитать конфиг
systemctl enable	Включить автозагрузку
systemctl disable	Отключить автозагрузку
systemctl status	Проверить статус + последние логи
systemctl cat	Показать unit-файл
systemctl edit	Создать оверлей
systemctl daemon-reload	Применить изменения unit-файлов
