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

Через флеш значения которые были

        max_connections = 40 / 100 
        shared_buffers = 1GB /128MB
        effective_cache_size = 3GB /4GB
        maintenance_work_mem = 512MB / 64MB
        checkpoint_completion_target = 0.9
        wal_buffers = 16MB / -1
        default_statistics_target = 500 / 100
        random_page_cost = 4 / 4
        effective_io_concurrency = 2 / 1
        work_mem = 6553kB / 4MB
        min_wal_size = 4GB / 80MB
        max_wal_size = 16GB / 1GB



6.    Протестировать заново
    
            bash-4.2$ pgbench -c8 -P 6 -T 60 -U postgres postgres
            Password:
            pgbench (15.5)
            starting vacuum...end.
            progress: 6.0 s, 1708.5 tps, lat 4.653 ms stddev 1.888, 0 failed
            progress: 12.0 s, 1681.7 tps, lat 4.751 ms stddev 2.003, 0 failed
            progress: 18.0 s, 1672.3 tps, lat 4.778 ms stddev 1.999, 0 failed
            progress: 24.0 s, 1686.8 tps, lat 4.735 ms stddev 2.017, 0 failed
            progress: 30.0 s, 1689.7 tps, lat 4.729 ms stddev 2.113, 0 failed
            progress: 36.0 s, 1692.1 tps, lat 4.721 ms stddev 1.928, 0 failed
            progress: 42.0 s, 1683.0 tps, lat 4.747 ms stddev 1.954, 0 failed
            progress: 48.0 s, 1695.0 tps, lat 4.713 ms stddev 1.916, 0 failed
            progress: 54.0 s, 1675.5 tps, lat 4.768 ms stddev 2.015, 0 failed
            progress: 60.0 s, 1684.0 tps, lat 4.744 ms stddev 2.094, 0 failed
            transaction type: <builtin: TPC-B (sort of)>
            scaling factor: 1
            query mode: simple
            number of clients: 8
            number of threads: 1
            maximum number of tries: 1
            duration: 60 s
            number of transactions actually processed: 101220
            number of failed transactions: 0 (0.000%)
            latency average = 4.734 ms
            latency stddev = 1.995 ms
            initial connection time = 25.816 ms
            tps = 1687.355076 (without initial connection time)
            bash-4.2$



7.    Что изменилось и почему?

Судя по параметрам, мы увеличили объём выделяемой памяти. Но при тестировании принципиально показатели не изменились, незначительно улучшился latency stddev = 1.995 ms, возможно данные 
параметры ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB избыточны для тестирования, и ВМ справляется даже без оптимизации параметров postgres
Как говорили на вебинаре, данные параметры конфигурации необходимо подбирать опытным путём, рекомендаций по "оптитмальным" параметрам не существует.

8.    Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк

CREATE TABLE student(id serial, fio char(100))

        postgres=# CREATE TABLE student(id serial, fio char(100))
        postgres-# ;
        CREATE TABLE
        postgres=# \dt
                      List of relations
         Schema |       Name       | Type  |  Owner
        --------+------------------+-------+----------
         public | pgbench_accounts | table | postgres
         public | pgbench_branches | table | postgres
         public | pgbench_history  | table | postgres
         public | pgbench_tellers  | table | postgres
         public | student          | table | postgres
        (5 rows)

        postgres=# INSERT INTO student(fio) SELECT 'noname' FROM generate_series(1,1000000);
        INSERT 0 1000000
        postgres=#



9.    Посмотреть размер файла с таблицей

            postgres=# SELECT pg_size_pretty(pg_TABLE_size('student'));
             pg_size_pretty
            ----------------
             135 MB
            (1 row)
            
            postgres=#

10.    5 раз обновить все строчки и добавить к каждой строчке любой символ

            postgres=# update student set fio = 'name1';
            UPDATE 1000000
            postgres=# update student set fio = 'name2';
            UPDATE 1000000
            postgres=# update student set fio = 'name3';
            UPDATE 1000000
            postgres=# update student set fio = 'name4';
            UPDATE 1000000
            postgres=# update student set fio = 'name5';
            UPDATE 1000000
            postgres=#
    
    
            postgres=# SELECT pg_size_pretty(pg_TABLE_size('student'));
             pg_size_pretty
            ----------------
             674 MB
            (1 row)


11.    Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
Подождать некоторое время, проверяя, пришел ли автовакуум

вывод уже после автовакуума

SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'student';

        postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'student';
         relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
        ---------+------------+------------+--------+-------------------------------
         student |    1003214 |          0 |      0 | 2023-12-30 10:21:18.187428-05
        (1 row)






12.    5 раз обновить все строчки и добавить к каждой строчке любой символ

обновил 3 раза, и получилось захватить процесс автовакуума

        postgres=# update student set fio = 'name6';
        UPDATE 1000000
        postgres=# update student set fio = 'name7';
        UPDATE 1000000
        postgres=# update student set fio = 'name8';
        UPDATE 1000000
        postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'student';
         relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
        ---------+------------+------------+--------+-------------------------------
         student |    1003214 |    2999844 |    299 | 2023-12-30 10:21:18.187428-05
        (1 row)
        
        postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'student';
         relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
        ---------+------------+------------+--------+-------------------------------
         student |     984079 |    1928791 |    195 | 2023-12-30 10:27:09.588305-05
        (1 row)
        
        postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'student';
         relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
        ---------+------------+------------+--------+-------------------------------
         student |     984079 |    1928791 |    195 | 2023-12-30 10:27:09.588305-05
        (1 row)
        
        postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'student';
         relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
        ---------+------------+------------+--------+-------------------------------
         student |     984079 |    1928791 |    195 | 2023-12-30 10:27:09.588305-05
        (1 row)


        postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'student';
         relname | n_live_tup | n_dead_tup | ratio% |       last_autovacuum
        ---------+------------+------------+--------+------------------------------
         student |    1411246 |          0 |      0 | 2023-12-30 10:28:09.00593-05
        (1 row)

   
13.    Посмотреть размер файла с таблицей

            postgres=# SELECT pg_size_pretty(pg_TABLE_size('student'));
             pg_size_pretty
            ----------------
             674 MB
            (1 row)

размер файла не уменьшился после вакуума, как и говорили на вебинаре, после обновления размер вырос, вакуум удалил строки, но оставил размер,
после нового обновления занимался данный объём

14.    Отключить Автовакуум на конкретной таблице

ALTER TABLE student SET (autovacuum_enabled = off);


        postgres=# ALTER TABLE student SET (autovacuum_enabled = off);
        ALTER TABLE
        postgres=#


15.    10 раз обновить все строчки и добавить к каждой строчке любой символ

            UPDATE 1000000
            postgres=# update student set fio = 'name002';
            UPDATE 1000000
            postgres=# update student set fio = 'name003';
            UPDATE 1000000
            postgres=# update student set fio = 'name004';
            UPDATE 1000000
            postgres=# update student set fio = 'name005';
            UPDATE 1000000
            postgres=# update student set fio = 'name006';
            UPDATE 1000000
            postgres=# update student set fio = 'name007';
            UPDATE 1000000
            postgres=# update student set fio = 'name008';
            UPDATE 1000000
            postgres=# update student set fio = 'name009';
            UPDATE 1000000
            postgres=# update student set fio = 'name010';
            UPDATE 1000000
            postgres=#


16.    Посмотреть размер файла с таблицей

            postgres=# SELECT pg_size_pretty(pg_TABLE_size('student'));
             pg_size_pretty
            ----------------
             1482 MB
            (1 row)

17.    Объясните полученный результат

за счёт мёртвых строк

        postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'student';
         relname | n_live_tup | n_dead_tup | ratio% |       last_autovacuum
        ---------+------------+------------+--------+------------------------------
         student |    1411246 |    9998695 |    708 | 2023-12-30 10:28:09.00593-05
        (1 row)



18.    Не забудьте включить автовакуум)

            ALTER TABLE
            postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'student';
             relname | n_live_tup | n_dead_tup | ratio% |       last_autovacuum
            ---------+------------+------------+--------+------------------------------
             student |    1411246 |    9998695 |    708 | 2023-12-30 10:28:09.00593-05
            (1 row)
            
            
            postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'student';
             relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
            ---------+------------+------------+--------+-------------------------------
             student |    1019325 |          0 |      0 | 2023-12-30 10:41:46.373623-05
            (1 row)
            
            postgres=# SELECT pg_size_pretty(pg_TABLE_size('student'));
             pg_size_pretty
            ----------------
             1482 MB
            (1 row)



Задание со *:
Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
Не забыть вывести номер шага цикла.


        DO $$
            BEGIN
            for i in 1..10 loop
                raise notice 'i:   %', i;
                UPDATE student SET fio = 'name-' || i;
            end loop;
            END;
        $$;

        postgres=#
        postgres=# DO $$
        postgres$#     BEGIN
        postgres$#     for i in 1..10 loop
        postgres$#         raise notice 'i:   %', i;
        postgres$#         UPDATE student SET fio = 'name-' || i;
        postgres$#     end loop;
        postgres$#     END;
        postgres$# $$;
        NOTICE:  i:   1
        NOTICE:  i:   2
        NOTICE:  i:   3
        NOTICE:  i:   4
        NOTICE:  i:   5
        NOTICE:  i:   6
        NOTICE:  i:   7
        NOTICE:  i:   8
        NOTICE:  i:   9
        NOTICE:  i:   10
        DO
        postgres=# select * from student where id = 100;
         id  |                                                 fio
        -----+------------------------------------------------------------------------------------------------------
         100 | name-10
        (1 row)
        
        postgres=#
        






