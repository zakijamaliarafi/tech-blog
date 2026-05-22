---
heroImage: '/vps-database-management.svg'
title: 'Installing and Securing Databases on Your VPS: MySQL and PostgreSQL'
description: 'A comprehensive guide to setting up, optimizing, and securing the world''s most popular open-source relational databases—MySQL, MariaDB, and PostgreSQL—on your virtual server.'
pubDate: 'Apr 16 2026'
---

While it is entirely possible to run a simple, static website using nothing but HTML and an Nginx web server, almost every meaningful web application requires a database. Whether you are deploying a standard WordPress blog, a complex custom Laravel application, a Node.js API, or a Django backend, you need a highly structured, queryable storage system to manage your users, content, and transactions.

The open-source relational database landscape is completely dominated by two towering giants: **MySQL** (alongside its identical fork, **MariaDB**) and **PostgreSQL**. 

Choosing between them often comes down to developer preference or the strict requirements of your software stack. MySQL is renowned for its speed, ubiquity, and ease of use, making it the default choice for the vast majority of PHP-based applications (like WordPress). PostgreSQL is legendary for its strict SQL compliance, bulletproof data integrity, and advanced features (like JSONB support and complex geographical queries), making it the darling of enterprise-grade Python and Ruby applications.

Regardless of which engine you choose, installing a database on a public VPS requires careful attention to security. A misconfigured database is a goldmine for attackers. This guide will walk you through installing, configuring, and hardening both systems on an Ubuntu/Debian-based VPS.

## 1. MySQL and MariaDB

For decades, MySQL was the undisputed king of web databases. After MySQL was acquired by Oracle, the original creator of MySQL (Michael Widenius) grew concerned about its open-source future. He "forked" the codebase and created **MariaDB**. 

For all practical intents and purposes, MariaDB is a drop-in replacement for MySQL. It uses the exact same commands, the exact same ports, and the exact same drivers. However, MariaDB generally receives performance optimizations and security patches faster than Oracle's MySQL. Therefore, on modern Linux distributions, installing MariaDB is highly recommended over standard MySQL.

### Installation

Connect to your VPS via SSH and update your package lists. Then, install the MariaDB server package.

```bash
sudo apt update
sudo apt install mariadb-server -y
```
*(If you strictly require Oracle MySQL, you would use `sudo apt install mysql-server` instead).*

Once the installation finishes, the database service will start automatically in the background.

### The Critical Security Step: `mysql_secure_installation`

Out of the box, MariaDB/MySQL is fundamentally insecure. It includes anonymous test users, an unsecured test database, and often allows the `root` database user to log in without a password.

You must immediately run the interactive security script included with the installation:

```bash
sudo mysql_secure_installation
```

You will be prompted with a series of critical security questions:
1.  **Enter current password for root:** If you just installed it, there is no password. Just press Enter.
2.  **Switch to unix_socket authentication?** Usually 'Y'. This allows the Linux `root` user to log into the database without a password, but blocks everyone else.
3.  **Change the root password?** If you didn't use unix_socket, set a massively complex, randomly generated password here.
4.  **Remove anonymous users?** **Yes (Y).** This prevents anyone from logging into the database without an account.
5.  **Disallow root login remotely?** **Yes (Y).** This is vital. The `root` database user should NEVER be allowed to connect from the public internet.
6.  **Remove test database and access to it?** **Yes (Y).**
7.  **Reload privilege tables now?** **Yes (Y).** This applies the changes immediately.

### Creating a Dedicated Database and User

You should **never** use the `root` database user in your application's configuration files (e.g., your `wp-config.php`). If your application has a vulnerability and an attacker extracts your database credentials, they will own every single database on the server.

You must practice the **Principle of Least Privilege**. Create a dedicated database for your app, and a dedicated user that only has permission to touch that specific database.

Log into the database shell as root:
```bash
sudo mysql -u root
```

Execute the following SQL commands:
```sql
-- 1. Create the database
CREATE DATABASE my_cool_app_db;

-- 2. Create the user and set a strong password
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'SuperSecureP@ssw0rd123!';

-- 3. Grant the user absolute control, but ONLY over their specific database
GRANT ALL PRIVILEGES ON my_cool_app_db.* TO 'app_user'@'localhost';

-- 4. Tell the engine to flush the cache and apply the new rules
FLUSH PRIVILEGES;

-- 5. Exit the shell
EXIT;
```

Your application can now safely connect using the `app_user` credentials.

## 2. PostgreSQL

PostgreSQL (affectionately called Postgres) is a massively powerful, object-relational database. It is slightly more complex to manage than MySQL, but the tradeoff in data integrity and advanced analytical capabilities is immense.

### Installation

The installation is straightforward via the standard `apt` repositories:

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
```
*(The `postgresql-contrib` package installs highly useful extensions and utilities).*

### Understanding Postgres Authentication (Peer vs. md5)

Postgres handles security differently than MySQL. By default, it relies heavily on Linux system users. 

When you install Postgres, it creates a Linux user account named `postgres`. It also creates a Postgres database superuser also named `postgres`. 
By default, Postgres uses **"peer" authentication**. This means that if you are logged into the Linux terminal as the user `postgres`, the database trusts you implicitly and lets you log into the database superuser account without asking for a password.

To manage the database, you must switch to the Linux `postgres` user:

```bash
# Switch to the postgres system user
sudo -i -u postgres

# Launch the Postgres interactive terminal (psql)
psql
```

### Creating a Database and Role (User)

In Postgres terminology, users are called "Roles". Let's create a dedicated database and a heavily restricted role for our application, mirroring the security best practices we used for MySQL.

Inside the `psql` prompt, execute:

```sql
-- 1. Create a new Role (User) with login capabilities and an encrypted password
CREATE ROLE my_app_role WITH LOGIN ENCRYPTED PASSWORD 'AnotherSecureP@ssw0rd!';

-- 2. Create the database, and assign ownership directly to the new role
CREATE DATABASE my_app_db OWNER my_app_role;

-- 3. Exit the shell
\q
```
*(To return to your normal Linux user account, type `exit`).*

Now, your web application can connect to the `my_app_db` database using the `my_app_role` credentials.

## Universal Database Security Hardening

Whether you deployed MariaDB or PostgreSQL, your job is not finished. You must harden the network layer.

### 1. Bind to Localhost Only

The single greatest security risk for a database is exposing its listening port (3306 for MySQL, 5432 for Postgres) to the public internet. If your web application (Nginx/PHP/Node) is running on the exact same VPS as your database, the database should **only** listen for connections originating from inside the server itself (`127.0.0.1` or `localhost`).

**For MariaDB/MySQL:**
Open `/etc/mysql/mariadb.conf.d/50-server.cnf` (or `/etc/mysql/mysql.conf.d/mysqld.cnf`).
Ensure the `bind-address` directive is set to localhost:
```ini
bind-address = 127.0.0.1
```
Restart the service: `sudo systemctl restart mariadb`.

**For PostgreSQL:**
Open `/etc/postgresql/<version>/main/postgresql.conf`.
Ensure the `listen_addresses` directive is set to localhost:
```ini
listen_addresses = 'localhost'
```
Restart the service: `sudo systemctl restart postgresql`.

### 2. Block Ports at the Firewall

Defense in depth dictates that even if your database is bound to localhost, you should still explicitly block the ports at your VPS firewall.

Using `ufw`:
```bash
sudo ufw deny 3306/tcp  # Block MySQL
sudo ufw deny 5432/tcp  # Block Postgres
```

If you use a GUI database management tool on your laptop (like DBeaver or DataGrip) and need to connect to the database, **do not open the port.** Instead, configure your GUI tool to use an **SSH Tunnel**. The GUI will securely connect to the server via SSH (Port 22), and then route its database traffic through that encrypted tunnel locally, entirely bypassing the need to expose the database to the internet.

### 3. Automate Logical Backups

Finally, implement a cron job to perform daily logical dumps of your databases. Raw database files cannot simply be copied while the database is running.

For MySQL:
```bash
mysqldump -u root my_cool_app_db > /backups/mysql_backup.sql
```
For Postgres (running as the postgres user):
```bash
pg_dump my_app_db > /backups/postgres_backup.sql
```

Compress these SQL files and use a tool like `rclone` to ship them to an offsite S3 bucket nightly. By combining isolated database users, strict localhost binding, and automated logical backups, you ensure your application's data remains both incredibly secure and highly resilient against disaster.
