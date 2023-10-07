# Replication1
Репликация и масштабирование. Часть 1. Sergey Mironov SYS-20.

# Задание 1  
На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.  

Ответить в свободной форме.  

Ответ.

В режиме Master-Slave новые данные пишутся только на Master, а со Slave они только читаются; в конфигурации Master-Master CRUD операции выполняются на обоих нодах.  
Целостность БД.   
Master-Slave: после потери и восстановления связи данные с Master будут дописаны в Slave до актуального состояния.   
Master-Master: после потери и восстановления связи между серверами возникает конфликт и нарушение целостности данных.  

Master-master репликация это тот же Master-Slave, только настроенный в обе стороны.  
Как было сказано на лекции, мастер-мастер репликацию в MySQL применяют крайне редко, т.к. её очень легко сломать, и сложно поддерживать в рабочем состоянии. В случае обрыва связи между двумя мастерами MySQL получаются рассинхронизированные базы данных и репликация ломается.  

# Задание 2  
Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.  

Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.  

Ответ.  

Запускаю на одной виртуалке с помощью docker compose, оба mysql в одной сети докер: docker-compose.yml  
Файлы конфигураций master.cnf slave.cnf монтируются в контейнер.  

Проверяем server id (и соответственно нормально ли прочитаны cnf файлы):  
SHOW VARIABLES LIKE 'server_id';  

На мастере:  
CREATE USER 'replication'@'%' IDENTIFIED WITH mysql_native_password BY 'password';  
GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';  
SHOW MASTER STATUS\G  
-- *************************** 1. row ***************************  
--              File: mysql-bin.000003  
--          Position: 666  
--      Binlog_Do_DB:   
--  Binlog_Ignore_DB:   
-- Executed_Gtid_Set:   
-- 1 row in set (0.0013 sec)  

На слейве:  

CHANGE REPLICATION SOURCE TO  
SOURCE_HOST='mysql-master',  
SOURCE_USER='replication',  
SOURCE_PASSWORD='password',  
SOURCE_LOG_FILE='mysql-bin.000003',  
SOURCE_LOG_POS=666;  

START REPLICA;  

SHOW REPLICA STATUS\G;  

Проверка наличия ошибок:

![image](https://github.com/SergeyM90/Replication1/assets/84016375/ab40bae4-6e30-4b84-b4eb-6bb13b50cafa)

Пробуем создать таблицу на мастере (порт 3306):  

![image](https://github.com/SergeyM90/Replication1/assets/84016375/4face383-b953-484b-a585-1435e88e0c87)

Проверяем на слейве (порт 3307):  

![image](https://github.com/SergeyM90/Replication1/assets/84016375/18f5546d-7ca9-4883-b9c5-5db0350a436f)






