# LOGIN to mysql

```sh
mysql -u root -p
```

# Change password policy

## Find the password variables

```sh
SHOW VARIABLES LIKE '%password%';
```

## Set a variable

```sh
SET GLOBAL validate_password.length=10;
```

# User management

## Create a user

```sh
CREATE USER 'someusername'@'localhost' IDENTIFIED BY 'password';
```

## Show current users

All users are inside the mysql database under the table User.
To list the users we can either `USE mysql` then `SELECT User, Host FROM user;`
or simply do `SELECT User, Host FROM mysql.user;` from anywhere in the cli.

## Change user password

```sh
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY 'YourNewStrongPassword';
```

## Grant privileges

```sh
GRANT ALL PRIVILEGES ON dbname.* TO 'username'@'localhost';
```

## Show privileges

```sh
SHOW GRANTS FOR 'username'@'localhost';
```

# Database management

## Create a database

```sh
CREATE DATABASE dbname;
```

## Select a database;

```sh
USE dbname;
```

## Create a table

- `table_name`: The unique name for your new table.
- `column_name`: The name for each column in the table.
- `datatype`: The type of data the column will hold (e.g., INT, VARCHAR(255), DATE).
- `constraints`: Optional rules to enforce data integrity (e.g., NOT NULL, UNIQUE, PRIMARY KEY, FOREIGN KEY, AUTO_INCREMENT).

```sh
CREATE tablename (
    column1 datatype constraints,
    column2 datatype constraints,
    column3 datatype constraints,
);
```

## Example

```sh
CREATE TABLE Employees (
    EmployeeID INT AUTO_INCREMENT PRIMARY KEY,
    FirstName VARCHAR(100) NOT NULL,
    LastName VARCHAR(100) NOT NULL,
    HourlyPay DECIMAL(5, 2),
    HireDate DATE
);
```

# mysqldump

- Dump a single database: `mysqldump -u [username] -p [database_name] > backup.sql`
- Dump multiple databases: `mysqldump -u [username] -p -B [database1] [database2] > backup.sql`
- Dump only the schema: `mysqldump -u [username] -p -d [database_name] > schema.sql`
- Dump only the data: `mysqldump -u [username] -p -t [database_name] > data.sql `

Privileges: The user running mysqldump needs appropriate privileges, such as SELECT, for the tables being dumped.
