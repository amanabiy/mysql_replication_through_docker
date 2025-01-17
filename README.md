# MySQL Replication Setup with Docker Compose

## Prerequisites
- Docker and Docker Compose installed on your machine.
- Basic knowledge of MySQL.

## Steps to Set Up MySQL Replication

### 1. **Start the Containers**

Run the following command to start the MySQL Master and Replica containers:

```bash
docker-compose up -d
```

This will start the containers in detached mode. It may take a minute or two for MySQL to initialize.

### 2. **Access the MySQL Master**

Access the MySQL master container:
```bash
docker exec -it mysql-master mysql -u root -p
```

Enter the root password (`root_password`).

### 3. **Create Replication User on Master**

Create a replication user with replication privileges:
```sql
CREATE USER 'replica_user'@'%' IDENTIFIED BY 'replica_password';
GRANT REPLICATION SLAVE ON *.* TO 'replica_user'@'%';
FLUSH PRIVILEGES;
```

### 4. **Get Master Status**

Get the current binary log file and position, which is needed for the replica to start replicating:
```sql
SHOW MASTER STATUS;
```

Note the **File** and **Position** values. You will need these for the replica configuration.

### 5. **Access the MySQL Replica**

Access the MySQL replica container:
```bash
docker exec -it mysql-replica mysql -u root -p
```

Enter the root password (`root_password`).

### 6. **Configure Replica with Master Information**

On the replica, run the following commands to configure it for replication, replacing `mysql-bin.000004` and `492` with the values you got from the master status:

```sql
STOP SLAVE;

CHANGE REPLICATION SOURCE TO
    SOURCE_HOST='mysql-master',
    SOURCE_USER='replica_user',
    SOURCE_PASSWORD='replica_password',
    SOURCE_LOG_FILE='mysql-bin.000004',
    SOURCE_LOG_POS=492;

START SLAVE;
```

### 7. **Check Replication Status**

On the replica, check the replication status:
```sql
SHOW SLAVE STATUS\G
```

Look for:
- **Slave_IO_Running**: Should be `Yes`.
- **Slave_SQL_Running**: Should be `Yes`.

If both are `Yes`, replication is working correctly.

### 8. **Troubleshooting**

If you encounter errors, you may need to:
- Check the `Last_IO_Error` and `Last_SQL_Error` in `SHOW SLAVE STATUS\G`.
- Ensure that the `replica_user` has the correct privileges and is using the appropriate authentication plugin.
- If necessary, use `SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1;` to skip errors and continue replication.

### 9. **Verify Data Replication**

To verify that the replication is working:
1. On the master, create a test database and table:
   ```sql
   CREATE DATABASE test_db;
   USE test_db;
   CREATE TABLE test_table (id INT PRIMARY KEY, name VARCHAR(100));
   INSERT INTO test_table (id, name) VALUES (1, 'Replication Test');
   ```

2. On the replica, check that the database and data are replicated:
   ```sql
   SHOW DATABASES;
   USE test_db;
   SELECT * FROM test_table;
   ```

If the data appears on the replica, replication is working as expected.