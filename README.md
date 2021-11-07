# Занятие №7: Нагрузочное тестирование и тюнинг PostgreSQL

1. сделать инстанс Google Cloud Engine типа e2-medium с ОС Ubuntu 20.04
2. поставить на него PostgreSQL 13 из пакетов собираемых postgres.org
3. настроить кластер PostgreSQL 13 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины
4. нагрузить кластер через утилиту https://github.com/Percona-Lab/sysbench-tpcc (требует установки https://github.com/akopytov/sysbench)  
**Выполнение:** Создал пользователя *sbtest* с пароле *password*. Далее создал БД *sbtest* и дал все права на неё польователю *sbtest*.  
Подготовил БД к тестированию командой:
```
./tpcc.lua --pgsql-host=127.0.0.1 --pgsql-port=5432 --pgsql-user=sbtest --pgsql-password=password --pgsql-db=sbtest --threads=4 --tables=1 --scale=10 --trx_level=RC --db-ps-mode=auto --db-driver=pgsql prepare
```
Запускал тестирование командой:
```
./tpcc.lua --pgsql-host=127.0.0.1 --pgsql-port=5432 --pgsql-user=sbtest --pgsql-password=password --pgsql-db=sbtest --threads=4 --tables=1 --scale=10 --trx_level=RC --db-ps-mode=auto --db-driver=pgsql --time=300 --report-interval=1 run
```
5. написать какого значения tps удалось достичь, показать какие параметры в
какие значения устанавливали и почему  
**Ответ:**
Максимальное значение tpc, которого удалось добиться в ходе тестирования - 205. Для этого изменял следующие параметры:
```
max_connections = 200
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 256MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 5242kB
min_wal_size = 1GB
max_wal_size = 4GB
max_worker_processes = 2
max_parallel_workers_per_gather = 1
max_parallel_workers = 2
max_parallel_maintenance_workers = 1
synchronous_commit = OFF
```
Все параметры кроме последнего установлены согласно рекоммендациям сайта *pgtune.leopard.in.ua* (при одном из тестов стандартные значения показали tpc выше, чем с этими рекоммендациями, но повторить такое не удалось). Синхронный коммит был отключен для существенного увеличения производительности жертвуя при этом надежностью.
