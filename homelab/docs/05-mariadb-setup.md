# MariaDB Setup

## Goal
Install a database server and create a database with a limited-privilege user.

## Concept
**MariaDB** is a database server. Applications shouldn't connect as the admin account. Instead, I created a **least-privilege user** that can only access one database, so a mistake or security problem in the app can't damage everything else.

## Steps
1. Installed MariaDB:
   ```bash
   sudo apt install mariadb-server
   sudo systemctl status mariadb
   ```
2. Logged in as admin and created the database and user:
   ```sql
   sudo mariadb

   CREATE DATABASE practice;
   CREATE USER 'appuser'@'localhost' IDENTIFIED BY '<password>';
   GRANT ALL PRIVILEGES ON practice.* TO 'appuser'@'localhost';
   FLUSH PRIVILEGES;
   ```
3. Created a `students` table and added sample rows (TODO: columns used).
   ```sql
   USE practice;
   -- TODO: paste your CREATE TABLE and INSERT statements
   ```
4. Tested logging in as the new user:
   ```bash
   mariadb -u appuser -p practice
   ```

> Never put real passwords in a public repo.

![MariaDB query output](../screenshots/05-mariadb-students.png)

## Problems and fixes
- TODO: describe any problems. If none, write "None encountered."

## What I learned
- TODO: one or two sentences.
