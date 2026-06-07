# GitHub — Pull Requests и работа с удалённым репозиторием

## Что такое Pull Request

Pull Request (PR) — запрос на включение изменений из одной ветки в другую (обычно в `main`) на GitHub.

Позволяет:

- проводить code review;
- обсуждать изменения;
- автоматически запускать тесты и проверки CI/CD.

---

# Полный цикл работы с Pull Request

## 1. Создать ветку локально

```bash
git checkout main
git pull origin main                    # обновить main до последней версии
git checkout -b feature/new-feature     # создать ветку для изменений
```

---

## 2. Внести изменения и запушить

```bash
# ... делаем изменения ...

git add .
git commit -m "feat: add new feature"
git push -u origin feature/new-feature
```

---

## 3. Создать Pull Request на GitHub

### Через веб-интерфейс

1. Открыть репозиторий на GitHub.
2. Нажать **Compare & pull request**.
3. Указать:
   - **base:** `main`
   - **compare:** `feature/new-feature`
4. Заполнить:
   - **Title** — краткое описание изменений.
   - **Description** — что сделано и как проверить.
5. Нажать **Create pull request**.

### Через GitHub CLI

```bash
gh pr create \
  --title "feat: add new feature" \
  --body "Описание изменений" \
  --base main
```

---

## 4. Провести Code Review

### Как ревьюер

1. Открыть вкладку **Files changed**.
2. Нажать `+` рядом со строкой.
3. Оставить комментарий.
4. Нажать **Finish review** и выбрать:

| Действие | Назначение |
|-----------|------------|
| Approve | Одобрить изменения |
| Comment | Просто оставить комментарий |
| Request changes | Потребовать доработки |

### Как автор

- отвечать на комментарии;
- вносить исправления;
- делать новые коммиты;
- выполнять push.

```bash
git push origin feature/new-feature
```

PR обновится автоматически.

---

## 5. Внести правки после ревью

### Обычное исправление

```bash
nano file.txt

git add file.txt
git commit -m "fix: address review feedback"
git push origin feature/new-feature
```

### Склеить несколько коммитов

```bash
git rebase -i HEAD~3
```

Заменить:

```text
pick
```

на:

```text
squash
```

или

```text
fixup
```

После завершения:

```bash
git push --force-with-lease origin feature/new-feature
```

---

## 6. Завершить Pull Request (Merge)

После одобрения нажать **Merge pull request**.

### Варианты Merge

| Опция | Что делает | Когда использовать |
|---------|------------|-------------------|
| Create a merge commit | Создаёт merge-коммит | Командная разработка |
| Squash and merge | Все коммиты превращаются в один | Много мелких коммитов |
| Rebase and merge | Линейная история без merge-коммита | Любителям чистой истории |

После merge рекомендуется удалить ветку:

```text
Delete branch
```

---

## 7. Обновить локальный репозиторий

```bash
git checkout main

git pull origin main

git branch -d feature/new-feature

git remote prune origin
```

---

# Управление ветками на GitHub

## Удалить ветку на GitHub

```bash
git push origin --delete feature/old-branch
```

## Посмотреть удалённые ветки

```bash
git branch -r
```

## Очистить ссылки на удалённые ветки

```bash
git remote prune origin
```

---

# Работа с чужим репозиторием (Fork)

## 1. Создать Fork на GitHub

Нажать кнопку **Fork**.

---

## 2. Клонировать свой Fork

```bash
git clone https://github.com/yourname/repo.git

cd repo
```

---

## 3. Добавить оригинальный репозиторий

```bash
git remote add upstream https://github.com/original/repo.git
```

Проверить:

```bash
git remote -v
```

---

## 4. Синхронизировать Fork с оригиналом

```bash
git checkout main

git pull upstream main

git push origin main
```

---

## 5. Создать рабочую ветку

```bash
git checkout -b feature/contribution
```

---

## 6. Внести изменения и отправить их

```bash
git push -u origin feature/contribution
```

---

## 7. Создать PR

На GitHub создать Pull Request из своей ветки в репозиторий-источник.

---

# GitHub CLI

## Установка

```bash
sudo apt install gh
```

## Авторизация

```bash
gh auth login
```

## Создать Pull Request

```bash
gh pr create \
  --title "feat: something" \
  --body "Description"
```

## Список Pull Requests

```bash
gh pr list
```

## Статус Pull Requests

```bash
gh pr status
```

## Получить PR локально

```bash
gh pr checkout 123
```

---

# Шаблон Pull Request

```markdown
## Что сделано

- Добавлен чеклист модуля 2
- Обновлён README.md

## Как проверить

1. Перейти на ветку feature/update
2. Запустить:

   make test

## Связанные задачи

- Closes #15
- Related to #12

## Скриншоты

![screenshot](link)

## Чеклист

- [ ] Код протестирован
- [ ] Документация обновлена
- [ ] CI/CD проходит успешно
```

---

# Краткая шпаргалка

| Команда / действие | Назначение |
|-------------------|------------|
| `git push -u origin feature/branch` | Опубликовать ветку |
| GitHub → New Pull Request | Создать PR |
| GitHub → Files changed | Посмотреть изменения |
| GitHub → Add review comment | Оставить комментарий |
| GitHub → Approve → Merge | Одобрить и влить изменения |
| `git pull origin main` | Обновить локальный main |
| `git branch -d feature/branch` | Удалить локальную ветку |
| `git push origin --delete feature/branch` | Удалить ветку на GitHub |

---

# Полезные рекомендации

### Перед созданием PR

```bash
git pull origin main
git rebase main
```

Это уменьшает вероятность конфликтов.

### Безопасный force push

```bash
git push --force-with-lease
```

Используйте вместо:

```bash
git push --force
```

Так вы случайно не затрёте чужие изменения.

### Проверить различия между ветками

```bash
git diff main...feature/new-feature
```

### Посмотреть все открытые PR через GitHub CLI

```bash
gh pr list --state open
```
