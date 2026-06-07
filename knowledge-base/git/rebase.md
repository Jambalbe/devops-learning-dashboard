# Git Rebase — переписывание истории

## Что такое rebase?

**Rebase** — это процесс переноса коммитов из одной ветки на вершину другой, создавая линейную историю.

В отличие от merge, который создаёт merge-коммит, rebase **переписывает историю** — коммиты получают новые хеши.

## Простой rebase

```bash
# Переключиться на ветку, которую хотим перебазировать
git checkout feature

# Перенести feature на вершину main
git rebase main
Что происходит:

Git находит общего предка feature и main

Берёт коммиты из feature, которых нет в main

Создаёт новые коммиты поверх main

Перемещает указатель feature на новые коммиты

Визуализация
До rebase:

text
      D---E (feature)
     /
A---B---C (main)
После rebase:

text
A---B---C---D'---E' (feature)
            ↑
          (main)
Rebase с конфликтом
bash
git checkout feature
git rebase main
Если возникает конфликт:

bash
# Показать конфликтные файлы
git status

# Разрешить конфликт вручную (отредактировать файл)

# Отметить как разрешённый
git add conflicted-file.txt

# Продолжить rebase
git rebase --continue
Другие опции при конфликте:

bash
git rebase --skip      # пропустить текущий коммит
git rebase --abort     # отменить rebase, вернуться как было
Интерактивный rebase (изменение истории)
bash
# Переписать последние 3 коммита
git rebase -i HEAD~3

# Или от определённого коммита (не включая его)
git rebase -i 7401577
Откроется редактор со списком коммитов:

text
pick abc1234 Первый коммит
pick def5678 Второй коммит
pick ghi9012 Третий коммит
Команды интерактивного rebase
Команда	Сокращение	Что делает
pick	p	Оставить коммит
reword	r	Изменить сообщение коммита
edit	e	Остановиться, чтобы поправить коммит
squash	s	Объединить с предыдущим коммитом
fixup	f	Объединить, отбросив сообщение
drop	d	Удалить коммит
Примеры использования
Объединить последние 3 коммита в один:

text
pick abc1234 Первый коммит
squash def5678 Второй коммит
squash ghi9012 Третий коммит
Изменить сообщение второго коммита:

text
pick abc1234 Первый коммит
reword def5678 Второй коммит
pick ghi9012 Третий коммит
Удалить ненужный коммит:

text
pick abc1234 Первый коммит
drop def5678 Второй коммит
pick ghi9012 Третий коммит
Очистка истории (удаление экспериментальных коммитов)
bash
# 1. Найти коммит, после которого начались эксперименты
git log --oneline

# 2. Запустить интерактивный rebase
git rebase -i 7401577   # с этого коммита (не включая его)

# 3. В редакторе заменить pick на drop для ненужных коммитов

# 4. Сохранить и закрыть

# 5. Принудительно запушить (только для личных веток!)
git push origin main --force-with-lease
Отмена rebase
Если rebase пошёл не так:

bash
# Полная отмена (вернуться до начала rebase)
git rebase --abort

# Или найти предыдущее состояние через reflog
git reflog                     # показать историю HEAD
git reset --hard HEAD@{2}      # вернуться на 2 шага назад
Золотое правило rebase
Никогда не делайте rebase публичных веток (main, develop, master)!

Rebase изменяет хеши коммитов. Если кто-то уже скопировал старые коммиты (git pull), у него возникнут серьёзные конфликты.

Можно делать rebase:
✅ Личных веток (feature/*, fix/*, experiment/*)

Нельзя делать rebase:
❌ Общих веток (main, develop, release/*)

Сравнение merge и rebase
Характеристика	Merge	Rebase
История	Сохраняет реальную хронологию	Делает историю линейной
Хеши коммитов	Остаются теми же	Меняются (новые коммиты)
Merge-коммит	Создаётся	Не создаётся
Безопасность	Безопасен для общих веток	Опасен для общих веток
Когда использовать	main, develop	feature, fix
Полезные алиасы
bash
# Красивый граф для визуализации rebase
git config --global alias.tree "log --graph --pretty=format:'%C(yellow)%h%C(cyan)%d%Creset %s %C(green)(%cr)%Creset' --all"

# Показать коммиты, которые будут перебазированы
git config --global alias.rebase-test "log --oneline origin/main..HEAD"
Полный пример работы с rebase
bash
# 1. Создать feature-ветку от main
git checkout main
git pull origin main
git checkout -b feature/new

# 2. Сделать несколько коммитов
echo "first" > file.txt
git add file.txt
git commit -m "first commit"

echo "second" >> file.txt
git commit -am "second commit"

# 3. Вернуться на main и сделать новый коммит
git checkout main
echo "main work" > main.txt
git add main.txt
git commit -m "main work"

# 4. Перебазировать feature на main
git checkout feature/new
git rebase main

# 5. Теперь история линейна, можно делать PR
git push -u origin feature/new
Краткая шпаргалка
Команда	Действие
git rebase main	Перебазировать текущую ветку на main
git rebase -i HEAD~3	Интерактивный rebase последних 3 коммитов
git rebase --continue	Продолжить после разрешения конфликта
git rebase --skip	Пропустить текущий коммит
git rebase --abort	Отменить rebase
git push --force-with-lease	Запушить после rebase (осторожно!)
git reflog	Найти предыдущее состояние HEAD
