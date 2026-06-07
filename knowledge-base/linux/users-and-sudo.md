# Пользователи и sudo в Linux

## Что такое пользователь в Linux

Каждый пользователь в Linux имеет:

* уникальный UID (User ID);
* основную группу (GID);
* домашнюю директорию;
* оболочку (shell);
* набор прав доступа.

Пользователи могут быть:

* обычными (human users);
* системными (для сервисов и демонов);
* суперпользователем (`root`).

---

# Просмотр информации о пользователях

## Текущий пользователь

```bash
whoami
```

---

## UID, GID и группы

```bash
id
```

Пример:

```text
uid=1000(user) gid=1000(user) groups=1000(user),27(sudo)
```

---

## Информация о конкретном пользователе

```bash
id username
```

---

## Пользователи, вошедшие в систему

```bash
users
```

---

## Подробная информация о сессиях

```bash
who
```

Пример:

```text
user pts/0 2026-06-07 10:15
```

---

## Активные сессии и выполняемые команды

```bash
w
```

Показывает:

* пользователя;
* время входа;
* терминал;
* загрузку системы;
* выполняемую команду.

---

## История входов

```bash
last
```

---

## Последний вход каждого пользователя

```bash
lastlog
```

---

# Создание пользователей

## Создать пользователя без домашней директории

```bash
sudo useradd username
```

---

## Создать пользователя с домашней директорией

```bash
sudo useradd -m username
```

Будет создан каталог:

```text
/home/username
```

---

## Указать оболочку

```bash
sudo useradd -m -s /bin/bash username
```

---

## Интерактивное создание пользователя

Debian/Ubuntu:

```bash
sudo adduser username
```

Команда запросит:

* пароль;
* имя;
* дополнительные сведения.

Обычно это наиболее удобный способ.

---

# Удаление пользователей

## Удалить пользователя

```bash
sudo userdel username
```

Домашняя директория останется.

---

## Удалить пользователя вместе с домашней директорией

```bash
sudo userdel -r username
```

---

# Управление паролями

## Сменить собственный пароль

```bash
passwd
```

---

## Сменить пароль другого пользователя

```bash
sudo passwd username
```

---

## Заблокировать пароль

```bash
sudo passwd -l username
```

Пользователь не сможет войти по паролю.

---

## Разблокировать пароль

```bash
sudo passwd -u username
```

---

# Работа с группами

## Посмотреть свои группы

```bash
groups
```

---

## Посмотреть группы другого пользователя

```bash
groups username
```

---

## Создать группу

```bash
sudo groupadd groupname
```

---

## Удалить группу

```bash
sudo groupdel groupname
```

---

## Добавить пользователя в группу

```bash
sudo usermod -aG groupname username
```

⚠️ Ключ `-aG` очень важен.

Без `-a` пользователь потеряет остальные группы.

---

## Удалить пользователя из группы

```bash
sudo gpasswd -d username groupname
```

---

# Изменение параметров пользователя

## Изменить UID

```bash
sudo usermod -u 1501 username
```

---

## Изменить основную группу

```bash
sudo usermod -g groupname username
```

---

## Изменить домашнюю директорию

```bash
sudo usermod -d /new/home -m username
```

Ключ `-m` переносит файлы автоматически.

---

# Переключение между пользователями (su)

## Переключиться на root

```bash
su
```

Потребуется пароль root.

---

## Переключиться на root с окружением

```bash
su -
```

Загружаются:

* переменные окружения;
* PATH;
* профиль пользователя root.

---

## Переключиться на другого пользователя

```bash
su username
```

---

## Переключиться с загрузкой окружения

```bash
su - username
```

---

## Вернуться обратно

```bash
exit
```

---

# Использование sudo

## Выполнить команду от root

```bash
sudo command
```

Пример:

```bash
sudo systemctl restart nginx
```

---

## Получить root-сессию

```bash
sudo -i
```

---

## Выполнить команду от имени другого пользователя

```bash
sudo -u username command
```

Пример:

```bash
sudo -u www-data php artisan cache:clear
```

---

## Проверить доступные команды sudo

```bash
sudo -l
```

---

# Настройка sudo

## Редактирование sudoers

Использовать только:

```bash
sudo visudo
```

Команда проверяет синтаксис перед сохранением.

---

## Полный доступ пользователю

```text
username ALL=(ALL) ALL
```

---

## Полный доступ группе

```text
%admin ALL=(ALL) ALL
```

---

## Разрешить конкретные команды

```text
username ALL=(ALL) /bin/systemctl restart nginx, /bin/systemctl status nginx
```

---

## Выполнение без пароля

```text
username ALL=(ALL) NOPASSWD: ALL
```

---

## Выполнение от конкретного пользователя

Например, от имени `www-data`:

```text
username ALL=(www-data) /usr/bin/systemctl restart php-fpm
```

---

## Запретить sudo

```text
username ALL=(ALL) !ALL
```

---

# Основные файлы пользователей

## Список пользователей

```text
/etc/passwd
```

Формат записи:

```text
username:x:UID:GID:comment:home:shell
```

Пример:

```text
user:x:1000:1000:User:/home/user:/bin/bash
```

---

## Хеши паролей

```text
/etc/shadow
```

Доступен только root.

---

## Группы

```text
/etc/group
```

---

## Домашние каталоги

```text
/home/username/
```

---

## Шаблон новых пользователей

```text
/etc/skel/
```

Содержимое копируется при создании нового пользователя.

---

# Специальные пользователи

| Пользователь    | Назначение               |
| --------------- | ------------------------ |
| root            | Суперпользователь        |
| daemon          | Системные демоны         |
| www-data        | Веб-серверы nginx/apache |
| nobody          | Минимальные привилегии   |
| systemd-network | Сетевые службы           |
| systemd-journal | Работа с журналом        |

---

# Специальные группы

| Группа          | Назначение                 |
| --------------- | -------------------------- |
| sudo            | Права sudo (Debian/Ubuntu) |
| wheel           | Права sudo (RHEL/CentOS)   |
| docker          | Управление Docker без sudo |
| adm             | Чтение системных логов     |
| systemd-journal | Доступ к journalctl        |

---

# Ограничения для пользователей

## Срок действия пароля

Установить 90 дней:

```bash
sudo chage -M 90 username
```

---

## Посмотреть настройки пароля

```bash
sudo chage -l username
```

---

## Заблокировать пользователя

```bash
sudo usermod -L username
```

---

## Разблокировать пользователя

```bash
sudo usermod -U username
```

---

## Запретить вход через shell

### Вариант 1

```bash
sudo usermod -s /sbin/nologin username
```

---

### Вариант 2

```bash
sudo usermod -s /bin/false username
```

Часто используется для сервисных аккаунтов.

---

# Полезные команды

## Найти файлы пользователя

```bash
find /home -user username
```

---

## Сменить владельца файлов

```bash
sudo chown -R newuser:newuser /home/olduser/
```

---

## Пользовательские сервисы systemd

```bash
systemctl --user
```

---

# Проверка безопасности

## Кто недавно входил

```bash
last
```

---

## Проверить SSH-входы

```bash
journalctl -u ssh
```

или

```bash
journalctl -u sshd
```

(зависит от дистрибутива)

---

## Неудачные попытки входа

```bash
sudo grep "Failed password" /var/log/auth.log
```

---

# Разница между root, su и sudo

| Команда        | Что делает                           |
| -------------- | ------------------------------------ |
| `root`         | Суперпользователь (UID 0)            |
| `su`           | Переключение на другого пользователя |
| `su -`         | Переключение с загрузкой окружения   |
| `sudo command` | Выполнение одной команды от root     |
| `sudo -i`      | Получение root-сессии                |

---

# Практические примеры

## Создать пользователя

```bash
sudo adduser developer
```

---

## Добавить в sudo

```bash
sudo usermod -aG sudo developer
```

---

## Проверить группы

```bash
groups developer
```

---

## Заблокировать пользователя

```bash
sudo usermod -L developer
```

---

## Разблокировать

```bash
sudo usermod -U developer
```

---

## Выполнить команду от имени www-data

```bash
sudo -u www-data whoami
```

---

## Посмотреть разрешения sudo

```bash
sudo -l
```

---

# Шпаргалка

| Команда        | Назначение                    |
| -------------- | ----------------------------- |
| `whoami`       | Текущий пользователь          |
| `id`           | UID/GID и группы              |
| `adduser`      | Создать пользователя          |
| `userdel -r`   | Удалить пользователя          |
| `passwd`       | Сменить пароль                |
| `groupadd`     | Создать группу                |
| `usermod -aG`  | Добавить в группу             |
| `su -`         | Переключиться на пользователя |
| `sudo command` | Выполнить команду от root     |
| `sudo -i`      | Root-сессия                   |
| `sudo visudo`  | Редактирование sudoers        |
| `chage`        | Политика паролей              |
| `last`         | История входов                |

---

# Запомнить

### Создать пользователя

```bash
sudo adduser username
```

### Добавить sudo

```bash
sudo usermod -aG sudo username
```

### Проверить группы

```bash
groups username
```

### Получить root-сессию

```bash
sudo -i
```

### Проверить разрешения sudo

```bash
sudo -l
```

### Безопасно редактировать sudoers

```bash
sudo visudo
```
