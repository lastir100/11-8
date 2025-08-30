# Домашнее задание к занятию «Резервное копирование баз данных»


---

### Задание 1. Резервное копирование

### Кейс
Финансовая компания решила увеличить надёжность работы баз данных и их резервного копирования. 

Необходимо описать, какие варианты резервного копирования подходят в случаях: 

1.1. Необходимо восстанавливать данные в полном объёме за предыдущий день.

1.2. Необходимо восстанавливать данные за час до предполагаемой поломки.

1.3.* Возможен ли кейс, когда при поломке базы происходило моментальное переключение на работающую или починенную базу данных.

*Приведите ответ в свободной форме.*

### Решение 1

1.1 полный бэкап 1 раз в день
1.2 инкрементный бэкап каждый час
1.3 для мгновенного переключения требуется репликация master-master

---

### Задание 2. PostgreSQL

2.1. С помощью официальной документации приведите пример команды резервирования данных и восстановления БД (pgdump/pgrestore).

2.1.* Возможно ли автоматизировать этот процесс? Если да, то как?

*Приведите ответ в свободной форме.*

### Решение 2
2.1 To dump a database called mydb into an SQL-script file:
```
$ pg_dump mydb > db.sql
```
To dump a database into a custom-format archive file:
```
$ pg_dump -Fc mydb > db.dump
```
To drop the database and recreate it from the dump:
```
$ dropdb mydb
$ pg_restore -C -d postgres db.dump
```
2.2 создание бэкапов автоматизировать возможно
например, выполнение скрипта с командой резервирования и изменяемым файлом дампа, по расписанию с помощью cron

---

### Задание 3. MySQL

3.1. С помощью официальной документации приведите пример команды инкрементного резервного копирования базы данных MySQL. 

3.1.* В каких случаях использование реплики будет давать преимущество по сравнению с обычным резервным копированием?

*Приведите ответ в свободной форме.*

### Решение 3
3.1 перед инкрементным резервным копированием надо сделать полный бэкап
```
$> mysqldump --single-transaction --flush-logs --source-data=2 --all-databases > backup_sunday_1_PM.sql
```
далее ежедневно (или другой период инкремента) выполнить 
```
mysqladmin flush-log
```
binary log между бэкапами, нужно сохранить в качестве инкрементной резервной копии, он будет содержать необходимую информацию для восстановления.

в руководстве:
```
$> mysqldump --single-transaction --flush-logs --source-data=2 \
         --all-databases > backup_sunday_1_PM.sql

After executing this command, the data directory contains a new binary log file, gbichot2-bin.000007,
because the --flush-logs option causes the server to flush its logs. The --source-data option causes mysqldump to write binary
log information to its output, so the resulting .sql dump file includes these lines:

-- Position to start replication or point-in-time recovery from
-- CHANGE REPLICATION SOURCE TO SOURCE_LOG_FILE='gbichot2-bin.000007',SOURCE_LOG_POS=4;
Because the mysqldump command made a full backup, those lines mean two things:

The dump file contains all changes made before any changes written to the gbichot2-bin.000007 binary log file or higher.

All data changes logged after the backup are not present in the dump file, but are present in the gbichot2-bin.000007 binary log file or higher.

On Monday at 1 p.m., we can create an incremental backup by flushing the logs to begin a new binary log file.
For example, executing a mysqladmin flush-logs command creates gbichot2-bin.000008.
All changes between the Sunday 1 p.m. full backup and Monday 1 p.m. are written in gbichot2-bin.000007.
This incremental backup is important, so it is a good idea to copy it to a safe place.
(For example, back it up on tape or DVD, or copy it to another machine.)
On Tuesday at 1 p.m., execute another mysqladmin flush-logs command.
All changes between Monday 1 p.m. and Tuesday 1 p.m. are written in gbichot2-bin.000008 (which also should be copied somewhere safe).
```
---

Задания, помеченные звёздочкой, — дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.
