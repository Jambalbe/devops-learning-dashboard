# Пользователи и sudo в Linux

## Просмотр информации о пользователях
```bash
whoami                    # текущий пользователь
id                        # UID, GID, группы текущего пользователя
id username               # информация о конкретном пользователе
users                     # список вошедших пользователей
who                       # подробно о вошедших (терминал, время)
w                         # активные сессии + выполняемые команды
last                      # история входов
lastlog                   # последний вход каждого пользователя
#Создание пользователей
bash
sudo useradd username               # просто создать (без домашней директории)
sudo useradd -m username            # создать с домашней директорией /home/username
sudo useradd -m -s /bin/bash username   # указать оболочку
sudo adduser username               # интерактивное создание (Debian/Ubuntu, удобнее)
#Удаление пользователей
bash
sudo userdel username               # удалить пользователя (домашняя директория остаётся)
sudo userdel -r username            # удалить пользователя и его домашнюю директорию
#Изменение пароля
bash
passwd                        # сменить свой пароль
sudo passwd username          # сменить пароль другого пользователя (только root)
sudo passwd -l username       # заблокировать пароль (запретить вход)
sudo passwd -u username       # разблокировать
#Группы
bash
groups                        # группы текущего пользователя
groups username               # группы другого пользователя
sudo groupadd groupname       # создать группу
sudo groupdel groupname       # удалить группу
sudo usermod -aG groupname username   # добавить пользователя в группу (-aG важно!)
sudo gpasswd -d username groupname    # удалить пользователя из группы
#Изменение UID/GID и домашней директории
bash
sudo usermod -u 1501 username         # изменить UID
sudo usermod -g groupname username    # изменить основную группу
sudo usermod -d /new/home -m username # сменить домашнюю директорию (-m переносит файлы)
#Смена пользователя (su)
bash
su                    # переключиться на root (запросит пароль root)
su -                  # переключиться на root с загрузкой его окружения
su username           # переключиться на другого пользователя
su - username         # переключиться с загрузкой окружения
exit                  # вернуться обратно
sudo — выполнение команд от root
bash
sudo command          # выполнить команду с правами root
sudo -i               # получить интерактивную root-сессию
sudo -u username command   # выполнить команду от имени указанного пользователя
sudo -l               # посмотреть, какие команды разрешены текущему пользователю
#Настройка sudo (/etc/sudoers)
#Редактировать ТОЛЬКО командой visudo (она проверяет синтаксис):

bash
sudo visudo           # правильный способ редактирования sudoers
Примеры строк в sudoers:
bash
# Дать пользователю право на все команды
username ALL=(ALL) ALL

# Дать группе admin права на все команды
%admin ALL=(ALL) ALL

# Дать пользователю право на конкретные команды
username ALL=(ALL) /bin/systemctl restart nginx, /bin/systemctl status nginx

# Разрешить выполнение без пароля
username ALL=(ALL) NOPASSWD: ALL

# Разрешить только команды от определённого пользователя (например, www-data)
username ALL=(www-data) /usr/bin/systemctl restart php-fpm

# Отключить sudo для пользователя
username ALL=(ALL) !ALL
#Файлы и директории пользователей
Путь	Назначение
/etc/passwd	Список пользователей (username:x:UID:GID:info:home:shell)
/etc/shadow	Хеши паролей (только для root)
/etc/group	Список групп
/home/username/	Домашняя директория
/etc/skel/	Шаблон файлов для новых пользователей
#Специальные пользователи и группы
Пользователь/группа	Назначение
root	Суперпользователь (UID 0)
daemon	Фоновые сервисы
www-data	Веб-сервер (nginx/apache)
systemd-journal	Доступ к логам
sudo / wheel	Пользователи с sudo
#Ограничения для пользователей
bash
# Установить срок действия пароля
sudo chage -M 90 username        # пароль истечёт через 90 дней
sudo chage -l username           # показать настройки

# Заблокировать пользователя
sudo usermod -L username         # lock
sudo usermod -U username         # unlock

# Задать оболочку (shell) запрещающую вход
sudo usermod -s /sbin/nologin username
sudo usermod -s /bin/false username
Полезные команды
bash
# Поиск файлов по владельцу
find /home -user username

# Изменить владельца всех файлов пользователя
sudo chown -R newuser:newuser /home/olduser/

# Посмотреть, кто использует ресурсы (для каждого пользователя)
systemctl --user          # юзер-сервисы (systemd)
#Краткая шпаргалка по переходу между пользователями
Команда	Результат
su	стать root (нужен пароль root)
su -	стать root + окружение root
su username	стать другим пользователем
sudo command	выполнить command от root
sudo -i	стать root через sudo
sudo -u user command	выполнить от user
#Безопасность
#Не используйте su root без необходимости, лучше sudo

#Никогда не ставьте NOPASSWD: ALL для обычных пользователей в продакшне

#Регулярно проверяйте last и journalctl -u ssh на подозрительные входы
