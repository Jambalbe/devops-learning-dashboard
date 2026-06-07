# Управление процессами в Linux

## Просмотр процессов
```bash
ps                    # процессы текущего терминала
ps aux                # все процессы (в формате BSD)
ps -ef                # все процессы (в формате System V)
ps aux --sort=-%cpu   # отсортированные по CPU (вверху самые активные)
ps -u user            # процессы конкретного пользователя

#Динамический просмотр
bash
top                   # интерактивный режим (q - выход)
htop                  # улучшенная версия top (установите: sudo apt install htop)
# В htop: F6 - сортировка, F9 - kill процесс, F10 - выход
#Древо процессов
bash
pstree                # дерево процессов
pstree -p             # с PID
pstree user           # дерево процессов пользователя
#Запуск процессов
bash
./script.sh           # запуск в foreground
./script.sh &         # запуск в background (возвращает управление)
nohup ./script.sh &   # защита от закрытия терминала (выходные в nohup.out)
#Управление фоновыми заданиями
bash
jobs                  # список фоновых заданий текущей сессии
fg %1                 # вернуть задание №1 в foreground
bg %1                 # перевести остановленное задание в background
Ctrl+Z                # остановить текущее foreground-задание (отправить в background)
#Остановка процессов (kill)
bash
kill PID              # отправить SIGTERM (15) — вежливое завершение
kill -15 PID          # то же самое
kill -9 PID           # SIGKILL — принудительное завершение (не может быть перехвачено)
kill -2 PID           # SIGINT — как Ctrl+C
kill -1 PID           # SIGHUP — перечитать конфигурацию (для демонов)
#Отправка сигналов по имени
bash
kill -TERM PID
kill -KILL PID
kill -INT PID
kill -HUP PID
#Отправка сигналов всем процессам
bash
pkill process_name    # убить все процессы с именем
pkill -f script.py    # убить по полному имени команды
killall process_name  # убить все процессы с точным именем
#Приоритеты процессов (nice/renice)
bash
nice -n 10 ./script.sh     # запустить с приоритетом 10 (низкий)
nice -n -5 ./script.sh     # запустить с приоритетом -5 (высокий — требует прав)
renice -n 5 -p PID         # изменить приоритет запущенного процесса
renice -n -10 -u user      # изменить приоритет всех процессов пользователя
#Ожидание завершения процесса
bash
wait PID              # дождаться завершения конкретного процесса
wait                  # дождаться всех дочерних процессов
#Информация о процессах
bash
pidof process_name    # PID процесса по имени
pgrep process_name    # список PID по имени
pgrep -u user process_name
lsof -p PID           # открытые файлы процесса
cat /proc/PID/status  # детальная информация о процессе
cat /proc/PID/cmdline # команда запуска (с аргументами)
#Сигналы (краткий справочник)
Сигнал	Номер	Значение
SIGHUP	1	Перечитать конфиг
SIGINT	2	Прерывание (Ctrl+C)
SIGQUIT	3	Выйти с дампом памяти (Ctrl+)
SIGKILL	9	Безусловное завершение
SIGTERM	15	Вежливое завершение (по умолчанию)
SIGSTOP	19	Приостановка (не может быть перехвачен)
SIGCONT	18	Продолжить остановленный процесс
#Примеры использования
bash
# Найти PID процесса nginx
pidof nginx

# Вежливо остановить nginx
kill -TERM $(pidof nginx)

# Принудительно остановить зависший процесс
kill -9 12345

# Запустить долгий скрипт в background
python long_task.py > output.log 2>&1 &

# Посмотреть, как меняется CPU у процесса
top -p 12345

# Приостановить и возобновить процесс
kill -STOP 12345
kill -CONT 12345
#Полезные алиасы (добавить в ~/.bashrc)
bash
alias pscpu='ps aux --sort=-%cpu | head -20'
alias psmem='ps aux --sort=-%mem | head -20'
alias myps='ps -u $USER'
