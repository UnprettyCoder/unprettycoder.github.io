---
layout: post
title: mySQL SYNTAX [SELECT]
date: 2020-01-21
excerpt: "DB SQL SYNTAX with 'SELECT'"
category: [Database]
feature: ../../assets/img/db1.png
comment: true
---

## SELECT
------------------
### SELECT basic skel
```sql
SELECT [DISTINCT] `column_1` [AS `col_1_nick`] [,`column_2` [AS `col_2_nick`]] ...
FROM `table_name`
WHERE `condition`
ORDER BY `column_1` [ASC | DESC] [,`column_2` [ASC | DESC]] ... [LIMIT `n` [,`m`]];
```

#### 1st Line
```sql
SELECT [DISTINCT] `column_1` [AS `col_1_nick`] [,`column_2` [AS `col_2_nick`]] ...
```
- `DISTINCT`
  - 출력하려는 필드의 값이 동일한 레코드가 존재하면 하나만 출력한다
  - 중복 불허 옵션
  - EXAMPLE
  
  ```sql
  SELECT DISTINCT author FROM books;
  ```
- `AS`
  - 출력하려는 필드의 열이름의 별칭을 지정할 수 있다.
  - 필드 값에 어떠한 연산을 수행한 후, 별칭으로 출력할 때 사용
  - EXAMPLE

  ```sql
  SELECT yearly_income / 12 AS monthly_income FROM projects where project_name = "myProject";
  ```

#### 2nd Line
```sql
FROM `table_name`
```
- `FROM`
  - 현재 사용중인 DB에서 어떤 TABLE로부터 정보를 검색할 것인지 지정

#### 3rd Line
```sql
WHERE `condition`
```
- `WHERE`
  - 특정 조건을 만족하는 레코드만을 출력하기 위한 조건 지정 옵션
  - EXAMPLE
    
  ```sql
  SELECT * FROM students WHERE age = 17;
  SELECT * FROM students WHERE age = 18 && gender = 1;
  SELECT * FROM students WHERE age = 19 || gender = 0;
  ```
  
#### last Line
```sql
ORDER BY `column_1` [ASC | DESC] [,`column_2` [ASC | DESC]] ... [LIMIT `n` [,`m`]];
```
- `ORDER BY`
  - 레코드가 출력되는 순서를 정하는 옵션
  - `ASC`(DEFAULT) : 오름차순 | `DESC` : 내림차순
  - EXAMPLE  
  
  ```sql
  SELECT * FROM students WHERE gender = 1 ORDER BY age DESC;
  ```
