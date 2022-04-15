SQL Complementation
========================

This article is about the complementation of tables in a relational database. The aim is to add rows which are not already in a table. The procedure is explained step-by-step, but the result will be a single SQL-script which guarantee the existent of unique rows.

This is a How-To add new rows to an existing table, considering if this rows already exists in the table.

## Example
In this example we use a table of students for our destination of new entries.


Let's take an example of a Student table with columns
student_id,
first_name, last_name,
reg_no (registration number),
branch and address (student's home address).

```
PRAGMA encoding="UTF-8";

-- In this example we use a table of students for ourer destination of new entries.
CREATE TABLE IF NOT EXISTS student (
  student_id INTEGER PRIMARY KEY AUTOINCREMENT,
  first_name varchar(32) NOT NULL,
  last_name varchar(32) NOT NULL
);
```


# 1 Build a new statement
First step is to describe the rows you like to add to your existing table.
Every of these rows is unique identified by a (not necessary proper) subset of the columns which contains
the overall payload. This subset build the **composite primary key**.

Example: Consider a series of measurements with only one observation per time. The
payload is the measurement observation and the time (date & daytime) of measurement. But only
the date and the daytime of measurement is the composite primary key.



## Abstract
The rows to be added have to be presented in terms of SQL. Any SELECT-Statement on a UNION or temporal TABLE will do.

At least every NOT NULL column has to be considered. The raw source data has to be mapped
to the columns of the destination table with AS.

## Formal

The `data_statement` is
```
SELECT <rawdata1> AS <destination_columns> [FROM TMP_TABLE]
UNION
SELECT <rawdata2> AS <destination_columns> [FROM TMP_TABLE]
...
```

or

```
SELECT N.* AS <destination_columns>
FROM (
    VALUES (<rawdata1>, <rawdata2>, ...) AS N
)
```

## Example

We keep it quite simple here and build a table with the first and last name of three students:

The `students_statement` is

```
SELECT 
	TRIM(substr(N.name, 1, 5)) AS first_name,
    TRIM(substr(N.name, 6))    AS last_name
FROM (
	SELECT 'Peter Yarrow' AS name UNION
	SELECT 'Paul Stookey' AS name UNION
	SELECT 'Mary Travers' AS name
) AS N;
```
or
```
SELECT 
    TRIM(substr(N.Column1, 1, 5)) AS first_name,
    TRIM(substr(N.Column1, 6))    AS last_name
FROM (
Values 
    ('Peter Yarrow'),
    ('Paul Stookey'),
    ('Mary Travers')
) AS N;
```

The table looks like this:

|first_name | last_name |
|---|---|
|Peter |	Yarrow|
|Paul |	Stookey|
|Mary |	Travers|

We have to ensure that a subset of columns identify any row --says any student-- identically.
The composition of this columns is a primary key which has to identify the primary key in the destination table.

Simplified: Here is no other student with the name Peter Yarrow.

You may have a look of the concept of Second Normal Form (2NF).



# 2 right join old with new

We use a right join on the destination table to associate the
rows already exists with the rows we potentially want to add.
Because the data to add have no keys you have to choose which
identifies the new rows. Obviously the unique constraints of
the destination table may be a good choice to start with unless
the only unique column is an auto-filled ID.


## Abstract

Two steps:

* For all potentially new data records combine them with the corresponding records
  in the destination table.
* Restrict this combination to those which do not have a corresponding entry in the
  destination table.

## Formal

A slightly  formal expression of  `new_data_statement` may be
```
SELECT
    newrows.*
FROM table
RIGHT JOIN (data_statement) AS newrows
    ON table.<composite_primary_key> = newrows.<composite_primary_key>
WHERE table.* IS NULL;
```

## Example

The `new_students_statement` becomes
```
SELECT
	newrows.*
FROM (
	SELECT 
		TRIM(substr(newrows.name, 1, 5)) AS first_name,
	    TRIM(substr(newrows.name, 6))    AS last_name
    FROM (
	    SELECT 'Peter Yarrow' AS name UNION
	    SELECT 'Paul Stookey' AS name UNION
	    SELECT 'Mary Travers' AS name   
    ) AS newrows) AS newrows
LEFT JOIN  student
ON student.first_name = newrows.first_name
 AND student.last_name = newrows.last_name
WHERE student.first_name IS NULL AND student.last_name IS NULL;
```

### OUTPUT

The table student looks like this:

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
add a unique constraint on (first_name, last_name) because, even when it's unlikely,
we want to maintain students with equal names. Or identify the same person, after
she has changed her name.

But in this example the choice of (first_name, last_name) as temporal primary key
will do, as long all added full names are different.


### OUTPUT:

After insert Mary in the destination table student the result of the `new_students_statement` becomes:

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
INSERT INTO ATABLE (payload_columns)
VALUES (new_data_statement);
```

## Example
```
INSERT INTO student (first_name,last_name)
	SELECT
		newrows.first_name,newrows.last_name
	FROM (
		SELECT 
			TRIM(substr(newrows.name, 1, 5)) AS first_name,
		    TRIM(substr(newrows.name, 6)) AS last_name
	    FROM (
		    SELECT 'Peter Yarrow' AS name UNION
		    SELECT 'Paul Stookey' AS name UNION
		    SELECT 'Mary Travers' AS name   
	    ) AS newrows) AS newrows
	LEFT JOIN  student
	ON student.first_name = newrows.first_name
	 AND student.last_name = newrows.last_name
	WHERE student.first_name IS NULL AND student.last_name IS NULL;
```

### Output

Let's look at the result with the help of
`
SELECT * FROM student;
`
.

|student_id|first_name|last_name|
| ---------|----------|---------|
|11	|Peter	|Yarrow|
|12	|Paul	|Stookey|
|13	|Mary	|Travers|

# Relations

Things get a bit more complicated when it comes to relations.

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

Assign the foreign key to the record

The `supplemented_data_statement` is
```
        SELECT
            newrows.*,
            btable.<foreign_key>
        FROM btable
        JOIN (data_statement) AS newrows
	    ON btable.<composite_primary_key> = newrows.<composite_primary_key>	
```

statement `new_data_statement`
```
SELECT
	newrows.* AS <columns_in_atable_including_foreign_key>
FROM atable
RIGHT JOIN (
        supplemented_data_statement	
	) AS newrows
ON atable.<composite_primary_key> = newrows.<composite_primary_key>
WHERE atable.* IS NULL;
```

And the `insert_data_statement` as we already knew it

```
INSERT INTO ATABLE (<columns_in_atable_including_foreign_key>)
VALUES (new_data_statement);
```


## One-to-one: Use a foreign key to the referenced table:

_You must also put a unique constraint on the foreign key column
(addess.student_id) to prevent multiple rows in the child table (address)
from relating to the same row in the referenced table (student)._

### Example

```sql
CREATE TABLE IF NOT EXISTS student (
  student_id INTEGER PRIMARY KEY AUTOINCREMENT,
  first_name varchar(32) NOT NULL,
  last_name varchar(32) NOT NULL,
  address_id INTEGER NOT NULL
);
```

or

```
ALTER TABLE student ADD COLUMN address_id INTEGER;
```

We add another table for the addresses in the database.

```sql
CREATE TABLE IF NOT EXISTS address (
  address_id INTEGER PRIMARY KEY AUTOINCREMENT,
  address varchar(64) NOT NULL,
  city varchar(16) NOT NULL,
  zipcode INTEGER NOT NULL,
  student_id INTEGER NOT NULL,
  FOREIGN KEY(student_id) REFERENCES student(student_id)
);
```

Add a new address:

```
INSERT INTO address (address, city, zipcode, student_id)
	SELECT
		newrows.address,
		newrows.city,
		newrows.zipcode,
		newrows.student_id
	FROM  (
	SELECT
			ADDROWS.address,
			ADDROWS.city,
			ADDROWS.zipcode,
			student.student_id
		FROM student
		JOIN (SELECT 
			'Paul' as first_name,
			'Stookey' as last_name,
			'Mataderos 2312' as address,
			'Marseille' as city,
			68306  as zipcode) as ADDROWS
		ON student.first_name = ADDROWS.first_name AND  student.last_name = ADDROWS.last_name	
		) AS newrows
	LEFT OUTER JOIN address
	ON address.address = newrows.address AND address.city = newrows.city AND address.zipcode = newrows.zipcode
	WHERE address.address_id IS NULL;
```



## One-to-many: Use a foreign key on the many side of the relationship linking back to the "one" side:

To figure out how this case is handed is left to the reader. You may try
it with the following tables:
* teachers(teacher_id, first_name, last_name) # the "one" side
* classes(class_id, class_name, teacher_id)  # the "many" side

## Many-to-many Use a junction table (example):

And this last example is also empty so far.

* student(student_id, first_name, last_name)
* classes(class_id, name, teacher_id)
* student_classes(class_id, student_id)     # the junction table

