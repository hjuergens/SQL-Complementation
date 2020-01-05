SQL Complementation
========================

This article is about the complementation of tables in a SQL database. The aim is to add rows which are not already in a table. The procedure is explainted step-by-step but the result will be a single SQL-script which guarantee the existens of unique rows.


# 1 Build a new statement
First step is to describe the rows you like to add to your existing table.

## Abstract
The rows to be added have to be presented in terms of SQL. Any SELECT-Statement on a UNION or temporal TABLE will do.
 

## Formal

```
SELECT <rawdata1> AS COMPOSITE_PRIMARYKEY_PART1 [FROM TMP_TABLE]
UNION
select 'Paul' as name
union
select 'Mary' as name;
```

```
select 'Peter' as name
union
select 'Paul' as name
union
select 'Mary' as name
```

## Example
In this example we use a table of students for our destination of new entries.


TODO Let's take an example of a Student table with columns student_id, name, reg_no(registration number), branch and address(student's home address).

```
CREATE TABLE IF NOT EXISTS student (
  student_id int(4) unsigned NOT NULL AUTO_INCREMENT,
  first_name varchar(32) NOT NULL,
  last_name varchar(32) NOT NULL,
) DEFAULT CHARSET=utf8;
```


-- First step is to describe the rows that sould be in existing table.
-- We keep it quite simple here:
select 'Peter' as name
union
select 'Paul' as name
union
select 'Mary' as name;

We have to ensure that a subset of columns identify any row identically.
The composition of this columns is a primary key and identify the primary key in the destination table.

You may have a look of the concept of Second Normal Form (2NF).

-- select (new statement) as NEWROWS



# 2 right join old with new

select
NEWROWS.*
from TABLE
right join (new statement) as NEWROWS
on TABLE.id = NEWROWS.id

# 3 insert where null

INSERT INTO 





---------------------------------------------


# Prepare for MySQL

Prepare a MySQL database to comprehend the steps a to test the complementing script
## database
Create a new database for testing purpose only.
```
SHOW DATABASES;
CREATE DATABASE Test_Records;
```

## user
Create a user for testing.
```
SELECT User, Host FROM mysql.user;
SHOW GRANTS FOR 'testuser'@'%';
SET PASSWORD FOR testuser = PASSWORD('password')
CREATE USER 'testuser'@'192.168.178.23' IDENTIFIED BY 'password';

GRANT ALL PRIVILEGES ON Test_Records.* TO 'testuser'@'%' identified by 'password';

FLUSH PRIVILEGES;

SHOW GRANTS FOR 'testuser'@'192.168.178.23';
```
## table

In the testing databse create a table of persons records.
```
USE Test_Records;

CREATE TABLE Persons (
    PersonID int,
    LastName varchar(255),
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255)
);
```

This is a How-To add new rows to an existing table, considering if this rows already exists in the table. 


--drop TABLE Persons;

-- MySQL: Check you table for unique constraints
select distinct CONSTRAINT_NAME
from information_schema.TABLE_CONSTRAINTS
where table_name = 'Persons' and constraint_type = 'UNIQUE';

-- This is a How-To add new rows to an existing table, considering if this rows already exists in the table. 


# right join old with new

-- We use a right join on the destination table to associate the
-- rows already exists with the rows we potentially want to add.
-- Because the data to add have no keys you have to choose which
-- identifies the new rows. Obvisouly the unique constraints of
-- the destination table may be a good choice to start with unless
-- the only unique column is an autofilled ID.


  
-- Selection of columns for which all combinated values are different. SC

## formaly
-- select
-- NEWROWS.*
-- from ATABLE
-- right join (new statement) as NEWROWS
-- on ATABLE.SELECTEDCOLUMNS = NEWROWS.SELECTEDCOLUMNS;

-- ### Example
-- Select all rows that does not exists in the targeted table so far.
SELECT
	NEWROWS.*
from student
right join (
	select 'Peter' as name
	union
	select 'Paul' as name
	union
	select 'Mary' as name) as NEWROWS
ON student.first_name = NEWROWS.name

-- ### OUTPUT:
-- Peter
-- Paul
-- Mary


# select only new ones

-- After connecting the rows to add with the rows in the targeted table a WHERE statement
-- restricted all rows of the source to the new ones.
-- formaly

-- SELECT
-- 	NEWROWS.*
-- FROM ATABLE
-- RIGHT JOIN (new statement) as NEWROWS
-- on ATABLE.SELECTEDCOLUMNS = NEWROWS.C;
-- WHERE ATABLE.SELECTEDCOLUMNS are NULL;

-- ### Example
-- Select all rows that does not exists in the targeted table so far.


SELECT
	NEWROWS.*
FROM student
RIGHT JOIN (
	SELECT 'Peter' AS name
	UNION
	SELECT 'Paul' AS name
	UNION
	SELECT 'Mary' AS name) as NEWROWS
ON student.first_name = NEWROWS.name
WHERE student.first_name IS NULL;

-- ### OUTPUT:
-- Peter
-- Paul
-- Mary

INSERT INTO Persons (FirstName, LastName) VALUES ('Mary','Travers');

-- ### OUTPUT:
-- id | name 
-- =====================
-- 1  | Peter
-- 2  | Paul 

| id | name |
----
| 1 | Peter |
| 2 | Paul |

DELETE FROM Persons WHERE FIRSTname = 'Mary';

select * from Persons

# insert where null


## formaly
-- INSERT INTO ATABLE (COLUMNS_IN_ATABLE)
-- VALUES (
-- select
-- NEWROWS.SOMECOLUMNS as COLUMNS_IN_ATABLE
-- from ATABLE
-- right join (new statement) as NEWROWS
-- on ATABLE.SELECTEDCOLUMNS = NEWROWS.SELECTEDCOLUMNS
-- WHERE ATABLE.ANYROW IS NULL;


```
INSERT INTO student (first_name,last_name) (
SELECT
	NEWROWS.name as first_name,
	'' as last_name
FROM student
right join (
	SELECT 'Peter' AS name
	UNION
	SELECT 'Paul' AS name
	UNION
	SELECT 'Mary' AS name) as NEWROWS
	ON student.first_name = NEWROWS.name
WHERE student.first_name is null
);
```
# Relations

-- Things get a little bit more complicated when it comes to relations.

-- There are three types of relations: One-to-one, One-to-many and Many-to-many (aka junction table)

-- Reference: https://stackoverflow.com/questions/7296846/how-to-implement-one-to-one-one-to-many-and-many-to-many-relationships-while-de

# One-to-one: Use a foreign key to the referenced table:


-- You must also put a unique constraint on the foreign key column (addess.student_id) to prevent multiple rows in the child table (address) from relating to the same row in the referenced table (student).
-- student: student_id, first_name, last_name, address_id
-- address: address_id, address, city, zipcode, student_id # you can have a
                                                        # "link back" if you need
-- MySQL                                                      

use Test_Records;

CREATE TABLE IF NOT EXISTS student (
  student_id int(4) unsigned NOT NULL AUTO_INCREMENT,
  first_name varchar(32) NOT NULL,
  last_name varchar(32) NOT NULL,
  address_id int(4) unsigned NOT NULL,
  PRIMARY KEY (`student_id`)
) DEFAULT CHARSET=utf8;
    
--ALTER TABLE student
--DROP COLUMN address_id; 

CREATE TABLE IF NOT EXISTS `address` (
  `address_id` int(4) unsigned NOT NULL AUTO_INCREMENT,
  `address` varchar(64) NOT NULL,
  `city` varchar(16) NOT NULL,
  `zipcode` int(5) unsigned NOT NULL,
  -- `student_id` int(4) unsigned NOT NULL,
  PRIMARY KEY (`address_id`) -- ,
  -- FOREIGN KEY (`student_id`) REFERENCES `student`(`student_id`),
  -- CONSTRAINT UC_Student UNIQUE (student_id)
) DEFAULT CHARSET=utf8;

ALTER TABLE `address` ADD CONSTRAINT UC_Student UNIQUE (student_id); 

ALTER TABLE address DROP COLUMN student_id; 

DROP TABLE address;

-- 1.) add addresses

INSERT INTO address (address, city, zipcode) ( 
-- set @row:=0;
SELECT
	-- @row := @row + 1 as row,
	NEWROWS.address,
	NEWROWS.city,
	NEWROWS.zipcode
FROM address
RIGHT JOIN (SELECT 
	'Antonio' as first_name,
	'Moreno' as last_name,
	'Mataderos 2312' as address,
	'Marseille' as city,
	68306  as zipcode) as NEWROWS
ON address.address = NEWROWS.address AND  address.city = NEWROWS.city AND address.zipcode = NEWROWS.zipcode 
WHERE address.address_id is null
);

-- 2.) add persons
-- 2. link the rows to thoses already exists in the the table
```
INSERT INTO student (first_name, last_name, address_id) (
SELECT
	NEWROWS.first_name,
	NEWROWS.last_name,
	NEWROWS.address_id
FROM student
RIGHT JOIN (
	SELECT
		ADDROWS.first_name,
		ADDROWS.last_name,
		address.address_id
	FROM address
	JOIN (SELECT 
		'Antonio' as first_name,
		'Moreno' as last_name,
		'Mataderos 2312' as address,
		'Marseille' as city,
		68306  as zipcode) as ADDROWS
	ON address.address = ADDROWS.address AND  address.city = ADDROWS.city AND address.zipcode = ADDROWS.zipcode
	) AS NEWROWS
ON student.first_name = NEWROWS.first_name AND student.last_name = NEWROWS.last_name
WHERE student.first_name is null
);
```

# One-to-many: Use a foreign key on the many side of the relationship linking back to the "one" side:

teachers: teacher_id, first_name, last_name # the "one" side
classes:  class_id, class_name, teacher_id  # the "many" side

# Many-to-many Use a junction table (example):

student: student_id, first_name, last_name
classes: class_id, name, teacher_id
student_classes: class_id, student_id     # the junction table
