# Git — ветки и слияние (Merge)

## Что такое ветка

**Ветка (branch)** — это указатель на определённый коммит.

Git хранит историю в виде графа коммитов, а ветка просто указывает на текущую вершину этой истории.

Преимущества веток:

* разработка новых функций независимо от основной версии;
* исправление ошибок без влияния на продакшн;
* параллельная работа нескольких разработчиков;
* безопасное тестирование изменений.

---

# Как Git хранит ветки

Пример:

```text
A → B → C ← main
```

Ветка `main` указывает на коммит `C`.

После создания новой ветки:

```text
A → B → C ← main
          ↑
       feature
```

Обе ветки указывают на один коммит.

После нового коммита в `feature`:

```text
A → B → C ← main
          \
           D ← feature
```

Ветки начинают расходиться.

---

# Просмотр веток

## Локальные ветки

```bash
git branch
```

Пример:

```text
* main
  feature/auth
  feature/api
```

Символ `*` показывает текущую ветку.

---

## Удалённые ветки

```bash
git branch -r
```

---

## Все ветки

```bash
git branch -a
```

---

## Последние коммиты веток

```bash
git branch -v
```

---

# Создание веток

## Создать ветку

```bash
git branch feature/new
```

Ветка создаётся, но переключения не происходит.

---

## Переключиться на ветку

```bash
git checkout feature/new
```

или современный вариант:

```bash
git switch feature/new
```

---

## Создать и переключиться одной командой

Старый синтаксис:

```bash
git checkout -b feature/new
```

Современный синтаксис:

```bash
git switch -c feature/new
```

---

# Переключение между ветками

```bash
git switch main
```

или

```bash
git checkout main
```

При переключении Git изменяет содержимое рабочей директории под выбранную ветку.

---

# Слияние веток (Merge)

Предположим:

```text
main
 └── A → B

feature
 └── A → B → C → D
```

Переходим в целевую ветку:

```bash
git switch main
```

Выполняем слияние:

```bash
git merge feature
```

---

# Fast-Forward Merge

Если ветка `main` не изменялась после создания `feature`, Git просто передвинет указатель.

До:

```text
main
  ↓
A → B

      feature
         ↓
A → B → C → D
```

После:

```text
A → B → C → D
             ↑
         main, feature
```

Коммит слияния не создаётся.

---

## Принудительно создать Merge Commit

Даже если возможен Fast-Forward:

```bash
git merge --no-ff feature
```

Полезно для сохранения истории фич.

---

# Three-Way Merge

Если ветки изменялись независимо:

До:

```text
        C ← main
       /
A → B
       \
        D ← feature
```

После:

```text
        C
       / \
A → B     M ← main
       \ /
        D
```

`M` — специальный merge-коммит.

---

# Просмотр истории веток

Самый полезный вариант:

```bash
git log --oneline --graph --decorate --all
```

Пример:

```text
*   a1b2c3 Merge branch 'feature'
|\
| * d4e5f6 Add login page
| * g7h8i9 Add auth service
* | j1k2l3 Update README
|/
* m4n5o6 Initial commit
```

---

# Разрешение конфликтов

## Причина конфликта

Один и тот же участок файла был изменён в разных ветках.

Пример:

### Ветка main

```text
Version 1
```

### Ветка feature

```text
Version 2
```

Git не знает, какую версию оставить.

---

## Запуск merge

```bash
git merge feature
```

Ошибка:

```text
CONFLICT (content): Merge conflict in file.txt
Automatic merge failed.
```

---

## Проверить конфликтные файлы

```bash
git status
```

---

## Вид конфликта

```text
<<<<<<< HEAD
Version 1
=======
Version 2
>>>>>>> feature
```

Где:

| Маркер            | Значение        |
| ----------------- | --------------- |
| `<<<<<<< HEAD`    | Текущая ветка   |
| `=======`         | Разделитель     |
| `>>>>>>> feature` | Вливаемая ветка |

---

## Разрешить конфликт

Отредактировать файл вручную:

```text
Version 2
```

или

```text
Version 1
Version 2
```

Затем:

```bash
git add file.txt
git commit
```

или

```bash
git merge --continue
```

---

# Отмена merge

## До создания коммита

```bash
git merge --abort
```

Вернёт репозиторий в состояние до начала слияния.

---

## После создания merge-коммита

Создать обратный коммит:

```bash
git revert -m 1 <merge_commit_hash>
```

---

# Удаление веток

## Безопасное удаление

```bash
git branch -d feature/new
```

Git проверит, что ветка уже слита.

---

## Принудительное удаление

```bash
git branch -D feature/new
```

---

## Удаление удалённой ветки

```bash
git push origin --delete feature/new
```

---

# Сравнение веток

## Разница между ветками

```bash
git diff main..feature
```

---

## Коммиты, которых нет в main

```bash
git log --oneline main..feature
```

---

## Показать обе ветки на графе

```bash
git log --oneline --graph main feature
```

---

# Временное сохранение изменений (stash)

Если нужно переключиться на другую ветку:

```bash
git stash
```

Проверить список:

```bash
git stash list
```

Переключиться:

```bash
git switch feature
```

Вернуть изменения:

```bash
git stash pop
```

---

# Полезные алиасы

## Красивое дерево коммитов

```bash
git config --global alias.tree \
"log --graph --pretty=format:'%C(yellow)%h%C(cyan)%d%Creset %s %C(green)(%cr)%Creset' --all"
```

Использование:

```bash
git tree
```

---

## Список всех веток

```bash
git config --global alias.branches "branch -a -v"
```

Использование:

```bash
git branches
```

---

# Типичный рабочий процесс

Создать новую ветку:

```bash
git switch -c feature/auth
```

Сделать изменения:

```bash
git add .
git commit -m "feat: add authentication"
```

Отправить на сервер:

```bash
git push -u origin feature/auth
```

После Pull Request:

```bash
git switch main
git pull
git merge feature/auth
```

Удалить ветку:

```bash
git branch -d feature/auth
```

---

# Шпаргалка

| Команда                           | Действие                     |
| --------------------------------- | ---------------------------- |
| `git branch`                      | Список веток                 |
| `git branch name`                 | Создать ветку                |
| `git switch name`                 | Переключиться                |
| `git switch -c name`              | Создать и переключиться      |
| `git merge name`                  | Слить ветку в текущую        |
| `git merge --abort`               | Отменить merge               |
| `git branch -d name`              | Удалить ветку                |
| `git branch -D name`              | Удалить принудительно        |
| `git push origin --delete name`   | Удалить удалённую ветку      |
| `git stash`                       | Временно сохранить изменения |
| `git stash pop`                   | Вернуть изменения            |
| `git log --graph --oneline --all` | Граф истории                 |

---

# Запомнить

```bash
# Создать ветку
git switch -c feature/new

# Сделать коммит
git add .
git commit -m "feat: new feature"

# Отправить ветку
git push -u origin feature/new

# Слить в main
git switch main
git merge feature/new

# Удалить ветку
git branch -d feature/new
```

Это базовый цикл работы с ветками в Git.
