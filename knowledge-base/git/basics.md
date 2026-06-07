# Git — основы работы

## Что такое Git

**Git** — распределённая система контроля версий.

Позволяет:

- отслеживать изменения в файлах;
- работать в команде;
- хранить историю проекта;
- возвращаться к предыдущим версиям кода.

---

# Первоначальная настройка

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global core.editor nano
git config --global --list
```

### Пояснение

| Команда | Назначение |
|----------|------------|
| `user.name` | Имя автора коммитов |
| `user.email` | Email автора коммитов |
| `core.editor` | Редактор по умолчанию |
| `--list` | Показать все настройки |

---

# Создание репозитория

```bash
# Создать новый репозиторий
git init

# Клонировать репозиторий по HTTPS
git clone https://github.com/user/repo.git

# Клонировать по SSH
git clone git@github.com:user/repo.git
```

---

# Основные команды (жизненный цикл файла)

```bash
git status
git add file.txt
git add .
git add --all

git commit -m "сообщение"
git commit -am "сообщение"
```

### Что происходит

```text
Рабочая директория
        ↓
     git add
        ↓
      Staging
        ↓
   git commit
        ↓
   Репозиторий Git
```

---

# Просмотр истории

```bash
git log
git log --oneline
git log --oneline --graph
git log -n 5
git log --since="2 days ago"
git show commit_hash
```

### Полезный вариант

```bash
git log --oneline --graph --decorate --all
```

Показывает:

- ветки;
- теги;
- граф истории.

---

# Просмотр изменений (diff)

```bash
git diff
git diff --staged
git diff commit1 commit2
```

| Команда | Что показывает |
|----------|----------------|
| `git diff` | Изменения до `git add` |
| `git diff --staged` | Изменения после `git add` |
| `git diff A B` | Разницу между коммитами |

---

# Работа с удалёнными репозиториями

```bash
git remote -v

git remote add origin URL

git push -u origin main
git push

git pull
git fetch

git clone URL
```

---

## Разница между pull и fetch

### git fetch

```bash
git fetch
```

Скачивает изменения, но не объединяет их.

### git pull

```bash
git pull
```

Выполняет:

```text
git fetch + git merge
```

---

# .gitignore — игнорирование файлов

Создайте файл:

```text
.gitignore
```

Пример содержимого:

```gitignore
*.log
temp/
secret.txt
build/
.DS_Store
*.pyc
node_modules/
.env
```

---

# Отмена изменений

| Что нужно сделать | Команда |
|------------------|----------|
| Отменить изменения в файле | `git restore file.txt` |
| Убрать файл из staging | `git restore --staged file.txt` |
| Отменить последний коммит, сохранить изменения | `git reset --soft HEAD~1` |
| Отменить коммит и удалить изменения | `git reset --hard HEAD~1` |
| Вернуться к конкретному коммиту | `git reset --hard commit_hash` |

---

# Ветки (Branches)

## Просмотр веток

```bash
git branch
git branch -a
```

---

## Создание ветки

```bash
git branch feature/auth
```

или сразу перейти:

```bash
git checkout -b feature/auth
```

Современный вариант:

```bash
git switch -c feature/auth
```

---

## Переключение между ветками

```bash
git checkout main
```

или

```bash
git switch main
```

---

## Удаление ветки

```bash
git branch -d feature/auth
```

Принудительно:

```bash
git branch -D feature/auth
```

---

# Слияние веток (Merge)

Перейти в целевую ветку:

```bash
git checkout main
```

Слить изменения:

```bash
git merge feature/auth
```

---

# Конфликты слияния

Если Git не может автоматически объединить изменения:

```text
<<<<<<< HEAD
Код из текущей ветки
=======
Код из другой ветки
>>>>>>> feature/auth
```

Необходимо вручную:

1. выбрать правильный вариант;
2. удалить служебные строки;
3. выполнить:

```bash
git add .
git commit
```

---

# Теги

## Создать тег

```bash
git tag v1.0
```

---

## Создать аннотированный тег

```bash
git tag -a v1.0 -m "Release 1.0"
```

---

## Отправить теги

```bash
git push --tags
```

---

## Посмотреть теги

```bash
git tag
```

---

# Алиасы Git

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.lg "log --oneline --graph"
```

Использование:

```bash
git st
git co main
git br
git lg
```

---

# Типичный рабочий процесс

```bash
git pull origin main

git checkout -b feature/new-thing

# вносим изменения

git status

git add .

git commit -m "feat: add new thing"

git push -u origin feature/new-thing
```

Далее создаётся Pull Request на GitHub.

---

# Полезные команды

## Кто изменил строку

```bash
git blame file.txt
```

---

## Поиск по истории

```bash
git log --grep="fix"
```

---

## Посмотреть содержимое коммита

```bash
git show HEAD
```

---

## Узнать автора последнего коммита

```bash
git log -1
```

---

## Посмотреть содержимое файла из другого коммита

```bash
git show commit_hash:file.txt
```

---

# Часто используемые файлы Git

| Файл | Назначение |
|--------|------------|
| `.git/config` | Настройки репозитория |
| `.gitignore` | Игнорируемые файлы |
| `.git/HEAD` | Текущая ветка |
| `.git/index` | Staging Area |
| `.git/logs/` | История изменений ссылок |

---

# Краткая шпаргалка

| Команда | Действие |
|----------|----------|
| `git init` | Создать репозиторий |
| `git clone URL` | Клонировать репозиторий |
| `git status` | Проверить состояние |
| `git add file` | Добавить в staging |
| `git commit -m "msg"` | Создать коммит |
| `git push` | Отправить изменения |
| `git pull` | Получить изменения |
| `git fetch` | Скачать изменения без слияния |
| `git log --oneline` | Краткая история |
| `git diff` | Показать изменения |
| `git restore file` | Отменить изменения |
| `git branch` | Работа с ветками |
| `git merge` | Слияние веток |
| `git tag` | Работа с тегами |

---

# Запомнить

```bash
git status
git add .
git commit -m "message"
git push
```

Это самые часто используемые команды в ежедневной работе.
