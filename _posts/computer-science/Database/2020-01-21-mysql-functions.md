---
layout: post
title: mySQL FUNCTIONs
date: 2020-01-21
excerpt: "Collections of mySQL functions"
category: [Database]
feature: ../../assets/img/db1.png
comment: true
---

## MySQL Functions
------------------
- LEFT(`field_name`, `n`)
  - 레코드의 `field_name` value를 왼쪽부터 n개 return
  - EXAMPLE
  
  ```sql
  SELECT LEFT(name, 3) as left_three 
  FROM students;
  ```

- WEEKDAY(`date_field`)
  - 레코드의 `date_field` value의 요일 return (`0 : monday ~ 6 : sunday`)
  - `date_field`의 value가 `YYYY-MM-DD` 이어야 한다.
  - EXAMPLE
  
  ```sql
  SELECT WEEKDAY(date_written) as weekday 
  FROM books
  WHERE author in ("Shakespeare", "Werber");
  ```
  
- FIELD(`field_name`, `val_1`, `val_2`, ...)
  - ORDER BY 뒤에 나와서 해당 필드 값의 순서를 직접 지정한다.
  - EXAMPLE
  
  ```sql
  SELECT name
  FROM students
  ORDER BY FIELD(LEFT(phone, 3), "010", "016", "011");
  ```
