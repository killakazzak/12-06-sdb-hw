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

Создаем учетную запись master для сервера репликации и проверям права:
```sql
CREATE USER 'replication'@'%';
GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
SHOW GRANTS FOR replication@'%';
```

![image](https://github.com/killakazzak/12-06-sdb-hw/assets/32342205/f69f8e5d-9f85-4016-8591-b32582b5f09c)

Настройка Master

```sql
printf "[mysqld]\nserver_id = 1\nlog_bin = mysql-bin\n" >> /etc/mysql/my.cnf
sudo systemctl restart mysql.service && sudo  systemctl status mysql.service
```


![image](https://github.com/killakazzak/12-06-sdb-hw/assets/32342205/e400bcae-dfbd-447c-a7c1-e2c4fa122993)

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

