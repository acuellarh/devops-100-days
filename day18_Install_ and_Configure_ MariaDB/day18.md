# Day 18: Install and Configure MariaDB

## Details 
https://engineer.kodekloud.com/project-details

## Activitu

We need to setup a database server on Nautilus DB Server in Stratos Datacenter. Please perform the below given steps on DB Server:

a. Install/Configure MariaDB server.

b. Create a database named kodekloud_db9.

c. Create a user called kodekloud_aim and set its password to ksH85UJjhb.

d. Grant full permissions to user kodekloud_aim on database kodekloud_db9.


## Solution steps

**Installation and configuration steps**

```
# 1. Install MariaDB
sudo yum install mariadb-server -y

# 2. Start the service
sudo systemctl start mariadb
sudo systemctl enable mariadb

# 3. Connect to MariaDB
sudo mysql

# 4. Inside MariaDB, run these commands:
CREATE DATABASE kodekloud_db9;
CREATE USER 'kodekloud_aim'@'localhost' IDENTIFIED BY 'ksH85UJjhb';
GRANT ALL PRIVILEGES ON kodekloud_db9.* TO 'kodekloud_aim'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

**Verify configuration**

```
# Step 1: Make sure you're at the Linux prompt (not MariaDB prompt)
# Step 2: Run this:
mysql -u kodekloud_aim -p kodekloud_db9

# Step 3: Enter password when asked:
# ksH85UJjhb

# Step 4: You should see:
MariaDB [kodekloud_db9]>

# Step 5: Test it worked:
SELECT DATABASE();

# Step 6: ctlr + D
EXIT;
```
**Check full permissions**

```
--  Connect to MariaDB
sudo mysql

-- check grants
SHOW GRANTS FOR 'kodekloud_aim'@'localhost';

# Step 6: ctlr + D

```

**Resuls**

```
[peter@stdb01 ~]$ sudo mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 6
Server version: 10.5.29-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SHOW GRANTS FOR 'kodekloud_aim'@'localhost';
+----------------------------------------------------------------------------------------------------------------------+
| Grants for kodekloud_aim@localhost                                                                                   |
+----------------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `kodekloud_aim`@`localhost` IDENTIFIED BY PASSWORD '*EA9DB4E40E56A4D2684320C021DF1D3B6FFECEE7' |
| GRANT ALL PRIVILEGES ON `kodekloud_db9`.* TO `kodekloud_aim`@`localhost`                                             |
+----------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.000 sec)

MariaDB [(none)]> ^D^FBye
[peter@stdb01 ~]$ 

```
