# Файлы и права доступа в Linux

## Просмотр прав
```bash
ls -l file.txt
# -rw-r--r-- 1 user group 1234 date file.txt
# Первый символ: - (файл), d (директория), l (ссылка)
# Далее: rwx (владелец), rwx (группа), rwx (остальные)

#Изменение прав (символьный и числовой методы)
chmod u+x script.sh        # добавить execute владельцу
chmod go-w file.txt        # убрать write у группы и остальных
chmod 755 script.sh        # rwxr-xr-x
chmod 600 secret.txt       # rw-------

#Изменение владельца
chown user:group file.txt
chown -R user:group dir/

#Права по умолчанию (umask)
umask 022   # новые файлы 644, директории 755

#Специальные биты
chmod +t dir/    # sticky bit (только владелец может удалять)
chmod g+s dir/   # setgid (новые файлы наследуют группу)
chmod u+s file   # setuid (выполняется от владельца)

#Краткая шпаргалка
Число	Права	Описание
7	rwx	чтение+запись+выполнение
6	rw-	чтение+запись
5	r-x	чтение+выполнение
4	r--	только чтение
0	---	нет прав
