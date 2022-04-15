Examples MySQL
==============

# Create student table

```
-- In this example we use a table of students for ourer destination of new entries.
CREATE TABLE IF NOT EXISTS student (
  student_id int(4) unsigned NOT NULL AUTO_INCREMENT,
  first_name varchar(32) NOT NULL,
  last_name varchar(32) NOT NULL,
) DEFAULT CHARSET=utf8;
```

# 1 Build a new statement

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

# 2 right join old with new

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
    SELECT 'Mary Travers' AS name
    ) AS newrows
ON student.first_name = newrows.first_name
 AND student.last_name = newrows.last_name
WHERE student.first_name IS NULL AND student.last_name IS NULL;
```

# 3 insert where null
```
INSERT INTO student (first_name,last_name) (
SELECT
	newrows.first_name,newrows.last_name
FROM student
RIGHT JOIN (SELECT 
	TRIM(LEFT( N.NAME, 5)) AS first_name,
	TRIM(RIGHT(N.NAME, 7)) AS last_name
    FROM (
    SELECT 'Peter Yarrow' AS name
    UNION
    SELECT 'Paul Stookey' AS name
    UNION
    SELECT 'Mary Travers' AS name
    ) AS newrows
ON student.first_name = newrows.first_name
 AND student.last_name = newrows.last_name
WHERE student.first_name IS NULL AND student.last_name IS NULL);
```

# One-to-one

```sql
CREATE TABLE IF NOT EXISTS student (
  student_id int(4) unsigned NOT NULL AUTO_INCREMENT,
  first_name varchar(32) NOT NULL,
  last_name varchar(32) NOT NULL,
  address_id int(4) unsigned NOT NULL,
  PRIMARY KEY (`student_id`)
) DEFAULT CHARSET=utf8;
```

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

add new address
```
INSERT INTO address (address, city, zipcode, student_id) (
SELECT
	newrows.address,
	newrows.city,
	newrows.zipcode,
	newrows.student_id
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
	) AS newrows
ON address.address = newrows.address AND address.city = newrows.city AND address.zipcode = newrows.zipcode
WHERE address.address_id IS NULL;
);
```
