# MySQL Mirror Server Setup

Below are the steps to configure a MySQL replica:

## 1. Prepare the Primary Server (Master)

 MySQL Configuration

* Edit the `my.cnf` file of the master server to enable binary logging and establish a unique server ID:
  
  ```plaintext
  [mysqld]
  log-bin=mysql-bin
  server-id=1
  ```
  
* Create a Replica User: You will need a specific user for the replica that has the appropriate permissions to read the binlogs. This can be done with the following SQL command

```sql
CREATE USER 'replica'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%';
FLUSH PRIVILEGES;
```

## 2. Prepare the Mirror Server (Slave)

* MySQL Configuration: As with the primary server, configure the my.cnf file on the mirror server with a unique ID, and without enabling binary logging

  ```plaintext
    [mysqld]
    server-id=2
  ```

* Initialize the Mirror Server: If the mirror server needs to have the current data from the primary server, perform a dump of the primary database and load it onto the mirror server

## 3. Create a Backup of the Primary Database

* Lock the Tables: This is necessary for a consistent read. On the primary server

```sql
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;
```

* Export the Database: Use mysqldump to export the database. Be sure to include the --master-data option so that the log position is automatically included in the dump

  ```plaintext
  mysqldump -u root -p --all-databases --master-data > dbdump.sql
  ```

* Unlock the Tables: Once the dump is complete, release the tables:

  ```sql
  UNLOCK TABLES;
  ```

## 4. Import the Dump to the Mirror Server

* Load the dbdump.sql file into the mirror server

  ```plaintext
  mysql -u root -p < dbdump.sql
  ```

## 5. Configure and Start the Replica

* On the mirror server, configure the replica to point to the primary server using the noted log file information:
  
  ```sql
    CHANGE MASTER TO
    MASTER_HOST='master_ip_address',
    MASTER_USER='replica',
    MASTER_PASSWORD='password',
    MASTER_LOG_FILE='log_file_name',
    MASTER_LOG_POS=log_position;
  ```

* Start the replication process:

```sql
START SLAVE;
```

* Check the replica status:

```sql
SHOW SLAVE STATUS\G
```

* The Slave_IO_Running and Slave_SQL_Running should both be set to Yes
