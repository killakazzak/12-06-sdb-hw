# Домашнее задание к занятию "`Git`" - `Тен Денис`

### Задание 1

Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*

### Решение Задание 1

Установка MySQL 8 на Ubuntu 22.04 LTS Linux

```bash
sudo apt update
sudo apt list --upgradable # get a list of upgrades
sudo apt upgrade
sudo apt install mysql-server-8.0
```
Первоначальная настройка SQL сервера

```bash
sudo mysql
```
```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'My7Pass@Word_9_8A_zE';
exit
```
```bash
sudo systemctl enable --now  mysql.service && sudo systemctl restart mysql.service && sudo  systemctl status mysql.service
```
Проверка установки

![image](https://github.com/killakazzak/12-06-sdb-hw/assets/32342205/6670bd39-f75b-4b0a-9261-544a69e1e4ac)

На сервере MASTER

Создаем учетную запись для репликации и проверям права

```sql
CREATE USER 'slaveuser'@'ubuntu22-client' IDENTIFIED WITH mysql_native_password BY 'My7Pass@Word_9_8A_zE';
GRANT REPLICATION SLAVE ON *.* TO 'slaveuser'@'ubuntu22-client';
SHOW GRANTS FOR 'slaveuser'@'ubuntu22-client';
```
![image](https://github.com/killakazzak/12-06-sdb-hw/assets/32342205/bd0964ba-a409-412e-aeea-98dded9a2154)

Настраиваем MySQL

```bash
printf "[mysqld]\nserver_id = 1\nlog_bin = mysql-bin\n" >> /etc/mysql/my.cnf
sudo systemctl restart mysql.service && sudo  systemctl status mysql.service
mysql -p
```
Получаем информацию о текущем состоянии мастера 

```sql
SHOW MASTER STATUS;
```

![image](https://github.com/killakazzak/12-06-sdb-hw/assets/32342205/f72e0d24-1e94-4cb4-8203-30c9f2ea931d)

На сервере MASTER

Настраиваем MySQL

```sql
printf "[mysqld]\nserver_id = 2\nlog_bin = mysql-bin\nrelay-log = /var/lib/mysql/mysql-relay-bin\nrelay-log-index = /var/lib/mysql/mysql-relay-bin.index\nread_only = 1\n" >> /etc/mysql/my.cnf
sudo systemctl restart mysql.service && sudo  systemctl status mysql.service
mysql -p
```

```sql
CHANGE MASTER TO MASTER_HOST='ubuntu22-server', MASTER_USER='slaveuser', MASTER_PASSWORD='My7Pass@Word_9_8A_zE', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=1214;
START SLAVE;
```

Получаем информацию о текущем состоянии репликации

```sql
SHOW SLAVE STATUS\G
```
![image](https://github.com/killakazzak/12-06-sdb-hw/assets/32342205/7df47013-f090-4f33-bef0-2fbcaf80a737)

Получаем информацию о статусе потоков репликации (replication worker), и позиции репликации, последнем прочитанным бинарный журнал (binary log), текущей задачи репликации и другие связанные данные.

Используя эту команду, можно получить детальную информацию о каждом потоке репликации на сервере базы данных MySQL

```sql
mysql> SELECT * FROM performance_schema.replication_applier_status_by_worker\G;
*************************** 1. row ***************************
                                           CHANNEL_NAME:
                                              WORKER_ID: 1
                                              THREAD_ID: 52
                                          SERVICE_STATE: ON
                                      LAST_ERROR_NUMBER: 0
                                     LAST_ERROR_MESSAGE:
                                   LAST_ERROR_TIMESTAMP: 0000-00-00 00:00:00.000000
                               LAST_APPLIED_TRANSACTION: ANONYMOUS
     LAST_APPLIED_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP: 2024-05-26 17:49:43.777606
    LAST_APPLIED_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP: 2024-05-26 17:49:43.777606
         LAST_APPLIED_TRANSACTION_START_APPLY_TIMESTAMP: 2024-05-26 17:49:43.775068
           LAST_APPLIED_TRANSACTION_END_APPLY_TIMESTAMP: 2024-05-26 17:49:43.777417
                                   APPLYING_TRANSACTION:
         APPLYING_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
        APPLYING_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
             APPLYING_TRANSACTION_START_APPLY_TIMESTAMP: 0000-00-00 00:00:00.000000
                 LAST_APPLIED_TRANSACTION_RETRIES_COUNT: 0
   LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_NUMBER: 0
  LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_MESSAGE:
LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_TIMESTAMP: 0000-00-00 00:00:00.000000
                     APPLYING_TRANSACTION_RETRIES_COUNT: 0
       APPLYING_TRANSACTION_LAST_TRANSIENT_ERROR_NUMBER: 0
      APPLYING_TRANSACTION_LAST_TRANSIENT_ERROR_MESSAGE:
    APPLYING_TRANSACTION_LAST_TRANSIENT_ERROR_TIMESTAMP: 0000-00-00 00:00:00.000000
*************************** 2. row ***************************
                                           CHANNEL_NAME:
                                              WORKER_ID: 2
                                              THREAD_ID: 53
                                          SERVICE_STATE: ON
                                      LAST_ERROR_NUMBER: 0
                                     LAST_ERROR_MESSAGE:
                                   LAST_ERROR_TIMESTAMP: 0000-00-00 00:00:00.000000
                               LAST_APPLIED_TRANSACTION:
     LAST_APPLIED_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
    LAST_APPLIED_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
         LAST_APPLIED_TRANSACTION_START_APPLY_TIMESTAMP: 0000-00-00 00:00:00.000000
           LAST_APPLIED_TRANSACTION_END_APPLY_TIMESTAMP: 0000-00-00 00:00:00.000000
                                   APPLYING_TRANSACTION:
         APPLYING_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
        APPLYING_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
             APPLYING_TRANSACTION_START_APPLY_TIMESTAMP: 0000-00-00 00:00:00.000000
                 LAST_APPLIED_TRANSACTION_RETRIES_COUNT: 0
   LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_NUMBER: 0
  LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_MESSAGE:
LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_TIMESTAMP: 0000-00-00 00:00:00.000000
                     APPLYING_TRANSACTION_RETRIES_COUNT: 0
       APPLYING_TRANSACTION_LAST_TRANSIENT_ERROR_NUMBER: 0
      APPLYING_TRANSACTION_LAST_TRANSIENT_ERROR_MESSAGE:
    APPLYING_TRANSACTION_LAST_TRANSIENT_ERROR_TIMESTAMP: 0000-00-00 00:00:00.000000
*************************** 3. row ***************************
                                           CHANNEL_NAME:
                                              WORKER_ID: 3
                                              THREAD_ID: 54
                                          SERVICE_STATE: ON
                                      LAST_ERROR_NUMBER: 0
                                     LAST_ERROR_MESSAGE:
                                   LAST_ERROR_TIMESTAMP: 0000-00-00 00:00:00.000000
                               LAST_APPLIED_TRANSACTION:
     LAST_APPLIED_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
    LAST_APPLIED_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
         LAST_APPLIED_TRANSACTION_START_APPLY_TIMESTAMP: 0000-00-00 00:00:00.000000
           LAST_APPLIED_TRANSACTION_END_APPLY_TIMESTAMP: 0000-00-00 00:00:00.000000
                                   APPLYING_TRANSACTION:
         APPLYING_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
        APPLYING_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
             APPLYING_TRANSACTION_START_APPLY_TIMESTAMP: 0000-00-00 00:00:00.000000
                 LAST_APPLIED_TRANSACTION_RETRIES_COUNT: 0
   LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_NUMBER: 0
  LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_MESSAGE:
LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_TIMESTAMP: 0000-00-00 00:00:00.000000
                     APPLYING_TRANSACTION_RETRIES_COUNT: 0
       APPLYING_TRANSACTION_LAST_TRANSIENT_ERROR_NUMBER: 0
      APPLYING_TRANSACTION_LAST_TRANSIENT_ERROR_MESSAGE:
    APPLYING_TRANSACTION_LAST_TRANSIENT_ERROR_TIMESTAMP: 0000-00-00 00:00:00.000000
*************************** 4. row ***************************
                                           CHANNEL_NAME:
                                              WORKER_ID: 4
                                              THREAD_ID: 55
                                          SERVICE_STATE: ON
                                      LAST_ERROR_NUMBER: 0
                                     LAST_ERROR_MESSAGE:
                                   LAST_ERROR_TIMESTAMP: 0000-00-00 00:00:00.000000
                               LAST_APPLIED_TRANSACTION:
     LAST_APPLIED_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
    LAST_APPLIED_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
         LAST_APPLIED_TRANSACTION_START_APPLY_TIMESTAMP: 0000-00-00 00:00:00.000000
           LAST_APPLIED_TRANSACTION_END_APPLY_TIMESTAMP: 0000-00-00 00:00:00.000000
                                   APPLYING_TRANSACTION:
         APPLYING_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
        APPLYING_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
             APPLYING_TRANSACTION_START_APPLY_TIMESTAMP: 0000-00-00 00:00:00.000000
                 LAST_APPLIED_TRANSACTION_RETRIES_COUNT: 0
   LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_NUMBER: 0
  LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_MESSAGE:
LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_TIMESTAMP: 0000-00-00 00:00:00.000000
                     APPLYING_TRANSACTION_RETRIES_COUNT: 0
       APPLYING_TRANSACTION_LAST_TRANSIENT_ERROR_NUMBER: 0
      APPLYING_TRANSACTION_LAST_TRANSIENT_ERROR_MESSAGE:
    APPLYING_TRANSACTION_LAST_TRANSIENT_ERROR_TIMESTAMP: 0000-00-00 00:00:00.000000
4 rows in set (0.00 sec)

ERROR:
No query specified
```

Проверяем репликацию:

На сервере MASTER

```sql
CREATE DATABASE testdb;
USE testdb;

CREATE TABLE test1 (
  id INT PRIMARY KEY,
  name VARCHAR(255)
);

CREATE TABLE test2 (
  id INT PRIMARY KEY,
  age INT
);

CREATE TABLE test3 (
  id INT PRIMARY KEY,
  description TEXT
);
```

На сервере SLAVE

```sql
SHOW DATABASES;
```
![image](https://github.com/killakazzak/12-06-sdb-hw/assets/32342205/baaf4888-8a8d-43cd-afc0-5f5b250cdcd7)

```sql
USE testdb;
SHOW TABLES;
```
![image](https://github.com/killakazzak/12-06-sdb-hw/assets/32342205/99eecd06-8a1f-43a4-a03f-16a4c0868d3f)



Дополнительно:
Обнаружил, что если база до создания реплики уже была создана на MASTER сервере, то при изменениях в базе на реплике возникает ошибка. Для исключения данной ошибки использую export / import баз данных.

На сервере MASTER

```sql
SHOW DATABASES;
```
![image](https://github.com/killakazzak/12-06-sdb-hw/assets/32342205/a3c34c8b-083c-4efa-aba2-09fe8a73ffca)


```bash
mysqldump -uroot -p --skip-lock-tables --single-transaction --flush-logs --hex-blob -A | gzip -c > dump.sql.gz
scp dump.sql.gz denis@ubuntu22-client:/home/denis/
```
На сервере SLAVE

```sql
STOP SLAVE;
```

```bash
gunzip dump.sql.gz
mysql -uroot -p -f < dump.sql
```
```sql
STOP SLAVE;
RESET SLAVE;
SHOW DATABASES;
```
![image](https://github.com/killakazzak/12-06-sdb-hw/assets/32342205/cd7f7397-c1ca-45ce-8982-f27ea7fba154)


На сервере MASTER

Получаем информацию о текущем состоянии мастера 

```sql
SHOW MASTER STATUS\G;
```
![image](https://github.com/killakazzak/12-06-sdb-hw/assets/32342205/db619e59-3031-4fb7-bbb9-18e664c32c4d)

На сервере SLAVE

```sql
CHANGE MASTER TO MASTER_HOST='ubuntu22-server', MASTER_USER='slaveuser', MASTER_PASSWORD='My7Pass@Word_9_8A_zE', MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=157;
START SLAVE;
SHOW SLAVE STATUS\G;
```

Проверка репликации

Для теста на сервере MASTER удаляем базу test4

До удаления MASTER севрер
![image](https://github.com/killakazzak/12-06-sdb-hw/assets/32342205/4669682e-be8f-45f3-9f12-97f633a89699)

До удаления SLAVE севрер
![image](https://github.com/killakazzak/12-06-sdb-hw/assets/32342205/47d777c5-a510-4586-80b4-fcd495feae2d)

После удаления MASTER севрер
![image](https://github.com/killakazzak/12-06-sdb-hw/assets/32342205/fdf8b18a-70e3-4d24-bbb4-fb21cef88a72)

После удаления SLAVE севрер
![image](https://github.com/killakazzak/12-06-sdb-hw/assets/32342205/7bb6532b-7e7f-4ab4-b4fd-3a46452b19ff)


```text
mysql> SHOW SLAVE STATUS\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: ubuntu22-server
                  Master_User: slaveuser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 903
               Relay_Log_File: mysql-relay-bin.000002
                Relay_Log_Pos: 1072
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 903
              Relay_Log_Space: 1282
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: 4faa5a5e-1aa8-11ef-9ec2-005056a1b8c6
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
       Master_public_key_path:
        Get_master_public_key: 0
            Network_Namespace:
1 row in set, 1 warning (0.00 sec)

ERROR:
No query specified
```




---

### Задание 2

Разработайте план для выполнения горизонтального и вертикального шаринга базы данных. База данных состоит из трёх таблиц: 

- пользователи, 
- книги, 
- магазины (столбцы произвольно). 

Опишите принципы построения системы и их разграничение или разбивку между базами данных.

*Пришлите блоксхему, где и что будет располагаться. Опишите, в каких режимах будут работать сервера.* 

### Решение Задание 2

---

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

---

### Задание 3* 

Опишите основные преимущества использования масштабирования методами:

- активный master-сервер и пассивный репликационный slave-сервер; 
- master-сервер и несколько slave-серверов;
- активный сервер со специальным механизмом репликации — distributed replicated block device (DRBD);
- SAN-кластер.

*Дайте ответ в свободной форме.*

### Решение Задание 3* 

---

### Задание 4*

Выполните настройку выбранных методов шардинга из задания 2.

*Пришлите конфиг Docker и SQL скрипт с командами для базы данных*.

### Решение Задание 4*

---

