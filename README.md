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

