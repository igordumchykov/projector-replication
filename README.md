# projector-replication

Sets up MySQL DB in master-slave mode

# Run
1. Start up master, slave1, slave2 containers:
```shell
docker-compose up -d
```
2. Run [script file](./build.sh)

You should see the status:
```
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: database_master
                  Master_User: mydb_slave_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: 1.000003
          Read_Master_Log_Pos: 849
               Relay_Log_File: 20e6ee29fc35-relay-bin.000002
                Relay_Log_Pos: 318
        Relay_Master_Log_File: 1.000003
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
          Exec_Master_Log_Pos: 849
              Relay_Log_Space: 535
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
                  Master_UUID: 92d20201-5268-11ee-818a-0242ac160002
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
mysql: [Warning] Using a password on the command line interface can be insecure.
mysql: [Warning] Using a password on the command line interface can be insecure.
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: database_master
                  Master_User: mydb_slave_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: 1.000003
          Read_Master_Log_Pos: 849
               Relay_Log_File: 899963b1d011-relay-bin.000002
                Relay_Log_Pos: 318
        Relay_Master_Log_File: 1.000003
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
          Exec_Master_Log_Pos: 849
              Relay_Log_Space: 535
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
                  Master_UUID: 92d20201-5268-11ee-818a-0242ac160002
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
```

3. Login to master container DB:

```shell
mysql root -u -p
```

4. Create a table:

```mysql
use my_db;
CREATE TABLE IF NOT EXISTS person (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255) NOT NULL, last_name TEXT)  ENGINE=INNODB;
```

5. Create a script that insert data to DB:
```mysql
CREATE PROCEDURE InsertData(num INT)
BEGIN
    DECLARE i INT DEFAULT 0;

    WHILE i < num DO
            insert into person (name, last_name) values ("Igor", "Dumchykov");
            SET i = i + 1;
        SELECT SLEEP(5);
        END WHILE;

END;
```

6. Login to slave containers and check `person` table:
```mysql
select * from person;
```

Data replicated to slaves.

7. Turn off slave 1 and check data on slave 2. Data replicated to slave 2.
8. Remove middle column in db on slave 2:
```mysql
ALTER TABLE person DROP COLUMN name;
```
9. Remove the last column in db on slave 2:
```mysql
ALTER TABLE person DROP COLUMN last_name;
```

9. Run select on slave2. You will find data without removed columns.

Also there is an error in slave2 logs:

`[ERROR] [MY-013146] [Repl] Slave SQL for channel '': Worker 1 failed executing transaction 'ANONYMOUS' at master log 1.000003, end_log_pos 4665; Column 1 of table 'my_db.person' cannot be converted from type 'varchar(1020(bytes))' to type 'text', Error_code: MY-013146`