# otus-pgsql-hw-lesson-8

##    Домашнее задание
Настройка autovacuum с учетом особеностей производительности

##    Цель:
запустить нагрузочный тест pgbench
настроить параметры autovacuum
проверить работу autovacuum

##    Описание/Пошаговая инструкция выполнения домашнего задания:

1.    Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
2.    Установить на него PostgreSQL 15 с дефолтными настройками

    [root@mck-network-test ~]# psql -V
    psql (PostgreSQL) 15.5
    [root@mck-network-test ~]#

    [root@mck-network-test ~]# sudo -u postgres psql
    could not change directory to "/root": Permission denied
    psql (15.5)
    Type "help" for help.
    
    postgres=#


3.    Создать БД для тестов: выполнить pgbench -i postgres

        bash-4.2$ pgbench -i postgres
        NOTICE:  table "pgbench_branches" does not exist, skipping
        NOTICE:  table "pgbench_tellers" does not exist, skipping
        NOTICE:  table "pgbench_accounts" does not exist, skipping
        NOTICE:  table "pgbench_history" does not exist, skipping
        creating tables...
        10000 tuples done.
        20000 tuples done.
        30000 tuples done.
        40000 tuples done.
        50000 tuples done.
        60000 tuples done.
        70000 tuples done.
        80000 tuples done.
        90000 tuples done.
        100000 tuples done.
        set primary key...
        vacuum...done.
        bash-4.2$



4.    Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres

bash-4.2$ pgbench -c8 -P 6 -T 60 -U postgres postgres
Password:
pgbench (15.5)
starting vacuum...end.
progress: 6.0 s, 1744.0 tps, lat 4.560 ms stddev 1.959, 0 failed
progress: 12.0 s, 1682.7 tps, lat 4.748 ms stddev 1.982, 0 failed
progress: 18.0 s, 1682.5 tps, lat 4.748 ms stddev 2.016, 0 failed
progress: 24.0 s, 1681.3 tps, lat 4.751 ms stddev 2.248, 0 failed
progress: 30.0 s, 1658.8 tps, lat 4.816 ms stddev 2.277, 0 failed
progress: 36.0 s, 1671.8 tps, lat 4.778 ms stddev 1.972, 0 failed
progress: 42.0 s, 1659.8 tps, lat 4.814 ms stddev 2.213, 0 failed
progress: 48.0 s, 1652.2 tps, lat 4.835 ms stddev 2.134, 0 failed
progress: 54.0 s, 1678.7 tps, lat 4.759 ms stddev 1.938, 0 failed
progress: 60.0 s, 1676.2 tps, lat 4.766 ms stddev 2.090, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 100736
number of failed transactions: 0 (0.000%)
latency average = 4.757 ms
latency stddev = 2.087 ms
initial connection time = 23.525 ms
tps = 1679.314678 (without initial connection time)
bash-4.2$


5.    Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
Протестировать заново
Что изменилось и почему?
Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
Посмотреть размер файла с таблицей
5 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
Подождать некоторое время, проверяя, пришел ли автовакуум
5 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть размер файла с таблицей
Отключить Автовакуум на конкретной таблице
10 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть размер файла с таблицей
Объясните полученный результат
Не забудьте включить автовакуум)
Задание со *:
Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
Не забыть вывести номер шага цикла.
