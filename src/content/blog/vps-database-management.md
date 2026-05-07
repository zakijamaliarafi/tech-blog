---
title: 'Installing and Securing Databases on Your VPS'
description: 'How to set up, optimize, and secure MySQL and PostgreSQL databases on your virtual server.'
pubDate: 'Apr 16 2026'
---

Almost every dynamic web application—from a simple WordPress blog to a complex custom SaaS—requires a relational database to store and retrieve data. The two open-source giants in this arena are **MySQL** (and its fork, MariaDB) and **PostgreSQL**.

Here is a guide to installing and securing them on an Ubuntu/Debian-based VPS.

## 1. MySQL / MariaDB

MySQL is the most widely used open-source relational database management system. MariaDB is a drop-in replacement fork created by the original developers of MySQL.

### Installation

```bash
sudo apt update
# Install MySQL
sudo apt install mysql-server -y

# OR Install MariaDB (Recommended for most Linux distributions)
sudo apt install mariadb-server -y
```

### Initial Security Configuration

Immediately after installation, you must run the included security script. This script removes test databases, disables remote root logins, and allows you to set a strong root password.

```bash
sudo mysql_secure_installation
```

Answer the prompts:
*   Set a strong root password (or use the auth_socket plugin).
*   Remove anonymous users? **Yes**
*   Disallow root login remotely? **Yes**
*   Remove test database and access to it? **Yes**
*   Reload privilege tables now? **Yes**

### Creating a Database and User

Log in to the MySQL prompt:
```bash
sudo mysql -u root -p
```

Create a database and a dedicated user with privileges only for that database:
```sql
CREATE DATABASE myapp_db;
CREATE USER 'myapp_user'@'localhost' IDENTIFIED BY 'StrongPassword123!';
GRANT ALL PRIVILEGES ON myapp_db.* TO 'myapp_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 2. PostgreSQL

PostgreSQL (often called Postgres) is known for its advanced features, strict SQL compliance, and robustness. It is generally preferred for highly complex applications requiring data integrity and advanced data types.

### Installation

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
```

### Basic Setup and Security

Postgres handles security differently. By default, it uses "ident" or "peer" authentication, meaning the Linux system user `postgres` can log into the database without a password.

Switch to the `postgres` system user:
```bash
sudo -i -u postgres
```

Access the PostgreSQL prompt:
```bash
psql
```

Create a new database and a user with a password:
```sql
CREATE DATABASE myapp_db;
CREATE USER myapp_user WITH ENCRYPTED PASSWORD 'StrongPassword123!';
GRANT ALL PRIVILEGES ON DATABASE myapp_db TO myapp_user;
\q
```

To exit the `postgres` user shell, type `exit`.

## General Database Security Best Practices

Regardless of which system you choose, adhere to these rules:

1.  **Never Expose the Port to the Internet:** Unless absolutely necessary, your database port (3306 for MySQL, 5432 for Postgres) should be blocked by your firewall (`ufw` or `firewalld`) from external access. The web application running on the same server should connect via `localhost` (127.0.0.1).
2.  **Principle of Least Privilege:** Never use the `root` or `postgres` superuser accounts in your web application's configuration file. Always create a dedicated user for each application with access restricted solely to its own database.
3.  **Regular Backups:** Use tools like `mysqldump` or `pg_dump` to create regular logical backups of your data.
