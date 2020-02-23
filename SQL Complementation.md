SQL Complementation
========================

This article is about the complementation of tables in a relational database. The aim is to add rows which are not already in a table. The procedure is explainted step-by-step but the result will be a single SQL-script which guarantee the existens of unique rows.

This is a How-To add new rows to an existing table, considering if this rows already exists in the table. 

## Example
In this example we use a table of students for our destination of new entries.


Let's take an example of a Student table with columns 
student_id, 
first_name, last_name,
reg_no (registration number), 
branch and address (student's home address).

```
-- In this example we use a table of students for ourer destination of new entries.
CREATE TABLE IF NOT EXISTS student (
  student_id int(4) unsigned NOT NULL AUTO_INCREMENT,
  first_name varchar(32) NOT NULL,
  last_name varchar(32) NOT NULL,
) DEFAULT CHARSET=utf8;
```


# 1 Build a new statement
First step is to describe the rows you like to add to your existing table.

## Abstract
The rows to be added have to be presented in terms of SQL. Any SELECT-Statement on a UNION or temporal TABLE will do.
 

## Formal

The `data_statement` is
```
SELECT <rawdata1> AS COMPOSITE_PRIMARYKEY_PART1 [FROM TMP_TABLE]
UNION
SELECT <rawdata2> AS COMPOSITE_PRIMARYKEY_PART2 [FROM TMP_TABLE]
...
```


## Example

We keep it quite simple here and build a table with the first and last name of three students:

The `students_statement` is
```
SELECT 
	TRIM(LEFT( N.NAME, 5)) AS first_name,
	TRIM(RIGHT(N.NAME, 7)) AS last_name
FROM (
SELECT 'Peter Yarrow' AS name
UNION
SELECT 'Paul Stookey' AS name
UNION
SELECT 'Mary Travers' AS name) AS N;
```

The table looks like this:

|first_name | last_name |
|---|---|
|Peter |	Yarrow|
|Paul |	Stookey|
|Mary |	Travers|

We have to ensure that a subset of columns identify any row --says any student-- identically.
The composition of this columns is a primary key which has to identify the primary key in the destination table.

You may have a look of the concept of Second Normal Form (2NF).



# 2 right join old with new

-- We use a right join on the destination table to associate the
-- rows already exists with the rows we potentially want to add.
-- Because the data to add have no keys you have to choose which
-- identifies the new rows. Obviously the unique constraints of
-- the destination table may be a good choice to start with unless
-- the only unique column is an auto-filled ID.


  
-- Selection of columns for which all combined values are different. SC

## Abstract

Two steps:

For all potentially new data records combine them with the corresponding records
in the destination table.

Restrict this combination to those which do not have a corresponding entry in the
destination table.   

### Alternative
Connecting all potentially new data to the already existing data. 
After connecting the rows to add with the rows in the targeted table a WHERE statement
restricted all rows of the source to the new ones.
Select all rows that does not exists in the targeted table so far.

## Formal

A slitly  formal expression of  `new_data_statement` may be
```
SELECT
    newrows.*
FROM table
RIGHT JOIN (data_statement) AS newrows
    ON table.COMPOSITE_PRIMARYKEY = newrows.COMPOSITE_PRIMARYKEY
WHERE table.* IS NULL;
```

## Example

The `new_students_statement` becomes
```
SELECT
	newrows.*
FROM student
RIGHT JOIN (SELECT 
	TRIM(LEFT( N.NAME, 5)) AS first_name,
	TRIM(RIGHT(N.NAME, 7)) AS last_name
    FROM (
    SELECT 'Peter Yarrow' AS name
    UNION
    SELECT 'Paul Stookey' AS name
    UNION
    SELECT 'Mary Travers' AS name) AS N
    ) AS newrows
ON student.first_name = newrows.first_name
 AND student.last_name = newrows.last_name
WHERE student.first_name IS NULL AND student.last_name IS NULL;
```

### OUTPUT

The table looks like this:

| first_name | last_name |
| ---------- | --------- |
| Peter | Yarrow |
| Paul  | Stookey |
| Mary  | Travers |


# 3 insert where null

The next expression may be executed several times.
```
INSERT INTO student (first_name, last_name) VALUES ('Mary','Travers');
```
But we still hesitate to
add an unique constraint on (first_name, last_name) because, even when it's unlikely,
we want to maintain students with equal names. Or identify the same person, after
she has change her name.

But in this example the choice of (first_name, last_name) as temporal primary key 
will do, as long all added full names are different.
 

### OUTPUT:

After insert Mary in the destination table student the Result of the `new_students_statement` becomes: 

| first_name | last_name |
| ---------- | --------- |
| Peter | Yarrow |
| Paul  | Stookey |

Because there is already a row with Mary Travers the 
part`student.first_name IS NULL AND student.last_name IS NULL` in the  
statement `new_students_statement` becomes `false`.



## Formal


`insert_data_statement`

```
INSERT INTO ATABLE (COLUMNS_IN_ATABLE)
VALUES (new_data_statement);
```

## Example
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

### Output

Lets see the result with
```
SELECT * FROM student;
```

|student_id|first_name|last_name|
| ---------|----------|---------|
|11	|Peter	|Yarrow|
|12	|Paul	|Stookey|
|13	|Mary	|Travers|

# Relations

Things get a little bit more complicated when it comes to relations.

There are three types of relations: 
* One-to-one,
* One-to-many and
* Many-to-many (aka junction table)

Reference: https://stackoverflow.com/questions/7296846/how-to-implement-one-to-one-one-to-many-and-many-to-many-relationships-while-de

## Abstract

The relations between tables are expressed in terms of foreign keys.
Before saving a new data record, the value for the foreign key must be determined
and the raw data record must be supplemented with this value.

## Formal


Let as replace `COLUMNS_IN_ATABLE` from :::
with `COLUMNS_IN_ATABLE_INCLUDING_FOREIGN_KEY` 

Assign the foreign key to the record

The `supplemented_data_statement` is
```
        SELECT
            ADDROWS.COLUMNS_IN_ATABLE_EXCLUDING_FOREIGN_KEY,
            BTABLE.COMPOSITE_KEY
        FROM BTABLE
        JOIN (data_statement) as ADDROWS
	    ON BTABLE.COMPOSITE_PRIMARYKEY = ADDROWS.COMPOSITE_PRIMARYKEY	
```

statement `new_data_statement`
```
SELECT
	NEWROWS.*
FROM ATABLE
RIGHT JOIN (
        supplemented_data_statement	
	) AS NEWROWS
ON ATABLE.COMPOSITE_PRIMARYKEY = NEWROWS.COMPOSITE_PRIMARYKEY
WHERE ATABLE.* is null
```

And the `insert_data_statement` as we already knew it

```
INSERT INTO ATABLE (COLUMNS_IN_ATABLE_INCLUDING_FOREIGN_KEY)
VALUES (new_data_statement);
```


## One-to-one: Use a foreign key to the referenced table:

_You must also put a unique constraint on the foreign key column
(addess.student_id) to prevent multiple rows in the child table (address) 
from relating to the same row in the referenced table (student)._

### Example

```sql
CREATE TABLE IF NOT EXISTS student (
  student_id int(4) unsigned NOT NULL AUTO_INCREMENT,
  first_name varchar(32) NOT NULL,
  last_name varchar(32) NOT NULL,
  address_id int(4) unsigned NOT NULL,
  PRIMARY KEY (`student_id`)
) DEFAULT CHARSET=utf8;
```

or

```
ALTER TABLE student ADD COLUMN address_id int(4) unsigned NOT NULL;
```

We add another table for the addresses in the database.

```
CREATE TABLE IF NOT EXISTS address (
  address_id int(4) unsigned NOT NULL AUTO_INCREMENT,
  address varchar(64) NOT NULL,
  city varchar(16) NOT NULL,
  zipcode int(5) unsigned NOT NULL,
  student_id int(4) unsigned NOT NULL, -- FOREIGN KEY
  PRIMARY KEY (address_id),
  FOREIGN KEY (student_id) REFERENCES student(student_id),
  CONSTRAINT UC_Student UNIQUE (student_id)
) DEFAULT CHARSET=utf8;
```

Add a new address:

```
INSERT INTO address (address, city, zipcode, student_id) (
SELECT
	NEWROWS.address,
	NEWROWS.city,
	NEWROWS.zipcode,
	NEWROWS.student_id
FROM address
RIGHT JOIN (
SELECT
		ADDROWS.address,
		ADDROWS.city,
		ADDROWS.zipcode,
		student.student_id
	FROM student
	JOIN (SELECT 
		'Antonio' as first_name,
		'Moreno' as last_name,
		'Mataderos 2312' as address,
		'Marseille' as city,
		68306  as zipcode) as ADDROWS
	ON student.first_name = ADDROWS.first_name AND  student.last_name = ADDROWS.last_name	
	) AS NEWROWS
ON address.address = NEWROWS.address AND address.city = NEWROWS.city AND address.zipcode = NEWROWS.zipcode
WHERE address.address_id is null
);
```



## One-to-many: Use a foreign key on the many side of the relationship linking back to the "one" side:

teachers: teacher_id, first_name, last_name # the "one" side
classes:  class_id, class_name, teacher_id  # the "many" side

## Many-to-many Use a junction table (example):

student: student_id, first_name, last_name
classes: class_id, name, teacher_id
student_classes: class_id, student_id     # the junction table

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




--drop TABLE Persons;

-- MySQL: Check you table for unique constraints
select distinct CONSTRAINT_NAME
from information_schema.TABLE_CONSTRAINTS
where table_name = 'Persons' and constraint_type = 'UNIQUE';


------------------------------------------------------------






