---
layout: post
title: mySQL SYNTAX [ALTER TABLE]
date: 2020-01-21
excerpt: "DB SQL SYNTAX with 'ALTER TABLE'"
category: [Database]
feature: ../../assets/img/db1.png
comment: true
---

## ALTER TABLE
-------------------------------------
#### 1. ADD column
- SYNTAX & example

```sql
ALTER TABLE `table_name` ADD `column_name` `data_type`;

ALTER TABLE students ADD address varchar(100);
```

#### 2. DROP column
- SYNTAX & example

```sql
ALTER TABLE `table_name` DROP `column_name`;

ALTER TABLE students DROP address;
```

#### 3. MODIFY column
- SYNTAX & example

```sql
ALTER TABLE `table_name` 
MODIFY (`column1_name` `data_type` [DEFAULT] [NOT NULL],
        `column2_name` `data_type` [DEFAULT] [NOT NULL]);

ALTER TABLE students 
MODIFY (address varchar(100) NOT NULL,
        age int 17);
```

#### 4. RENAME COLUMN
- SYNTAX & example

```sql
ALTER TABLE `table_name`
RENAME COLUMN `old_name` TO `new_name`;

ALTER TABLE students 
RENAME COLUMN sex TO gender;
```

#### 5. CHANGE COLUMN
- SYNTAX & example

```sql
ALTER TABLE `table_name`
CHANGE COLUMN `old_name` `new_name` `data_type` [NOT NULL];

ALTER TABLE students
CHANGE COLUMN sex gender boolean NOT NULL;
```

#### 6. ADD CONSTRAINT
- SYNTAX & example

```sql
ALTER TABLE `table_name`
ADD CONSTRAINT `constraint_name` `constraint` (`column_name`);

ALTER TABLE students
ADD CONSTRAINT age_not_null NOT NULL (age);
```

#### 7. DROP CONSTRAINT
- SYNTAX & example

```sql
ALTER TABLE `table_name`
DROP CONSTRAINT `constraint_name`;

ALTER TABLE students
DROP CONSTRAINT age_not_null;
```
