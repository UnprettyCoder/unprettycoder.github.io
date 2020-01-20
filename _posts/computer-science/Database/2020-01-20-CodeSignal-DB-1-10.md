---
layout: post
title: Code Signal DB 문제 [1 ~ 10]
date: 2020-01-20
excerpt: "DB 관련 문제 풀이"
category: [Database]
feature: ../../assets/img/db1.png
comment: true
---

## CODE SIGNAL Database algorithm solve [ 1 ~ 10 ]

#### 1. projectList
- Preconditions
    - Projects Table은 internal_id, project_name, team_size, team_lead, income 필드로 구성
- Requirements
    - Projects Table에서 internal_id, team_size 필드 없이 출력
- Solution
```sql
CREATE PROCEDURE projectList()
BEGIN
  alter table Projects drop internal_id;
  alter table Projects drop team_size;
    
  select * from Projects;
  /* select project_name, team_lead, income from Projects; 로 대체 가능 */
END
```
---------------------------------------------------

#### 2. countriesSelection
- Preconditions
    - countries Table은 name, continent, population 필드로 구성
- Requirements
    - countries Table에서 continent == "Africa" 인 레코드를 name 기준 오름차순 정렬
- Solution
```sql
CREATE PROCEDURE countriesSelection()
BEGIN
	select * from countries 
    where continent = "Africa" 
    order by name;
END
```
---------------------------------------------------

#### 3. monthlyScholarships
- Preconditions
    - scholarship Table은 id, scholarship 필드로 구성 && scholarship의 값은 연간 장학액수
- Requirements
    - scholarship Table에서 id, scholarship을 출력하되, scholarship은 연간이 아니라 월간으로 출력
- Solution
```sql
CREATE PROCEDURE monthlyScholarships()
BEGIN
	select id, scholarship/12 as scholarship from scholarships;
END
```
---------------------------------------------------

#### 4. projectsTeam
- Preconditions
    - projectLog Table은 id, name, description, timestamp 필드로 구성
- Requirements
    - projectLog Table에서 name을 오름차순으로 출력 [중복값은 출력하지 않는다.]
- Solution
```sql
CREATE PROCEDURE projectsTeam()
BEGIN
	select DISTINCT name from projectLog 
    order by name ASC;
END
```
---------------------------------------------------

#### 5. automaticNotifications
- Preconditions
    - users Table은 id, username, role, email 필드로 구성
- Requirements
    - users Table에서 role 값이 "admin", "premium" 인 것들을 제외하고 email 기준 오름차순 출력
- Solution
```sql
CREATE PROCEDURE automaticNotifications()
BEGIN
    SELECT email FROM users
    WHERE role not in ("admin", "premium")
    ORDER BY email;
END
```
---------------------------------------------------

#### 6. volleyballResults
- Preconditions
    - results Table은 name, country, scored, missed, wins 필드로 구성
- Requirements
    - results Table을 wins 기준 오름차순으로 정렬
- Solution
```sql
CREATE PROCEDURE volleyballResults()
BEGIN
	select * from results 
    order by wins ASC;
END
```
---------------------------------------------------

#### 7. mostExpensive
- Preconditions
    - Products Table은 id, name, price, quantity 필드로 구성
- Requirements
    - Products Table을 보고 (price * quantity) => TotalPrice 값이 가장 큰 record.name을 출력
    - TotalPrice의 최대값이 동일한 레코드가 다수 존재할 경우, name 기준 오름차순 출력
- Solution
```sql
CREATE PROCEDURE mostExpensive()
BEGIN
	select name from Products 
    order by price*quantity desc, name asc limit 1;
END
```
---------------------------------------------------


#### 8. contestLeaderboard
- Preconditions
    - leaderboard Table은 id, name, score 필드로 구성
- Requirements
    - leaderboard Table에서 score 기준 상위 4번째부터 5개의 record.name을 출력
- Solution
```sql
CREATE PROCEDURE contestLeaderboard()
BEGIN
	select name from leaderboard 
    order by score desc limit 3, 5;
END
```
---------------------------------------------------

#### 9. gradeDistribution
- Preconditions
    - Grades Table은 Name, ID, Midterm1, Midterm2, Final 필드로 구성
    - 점수 환산 opt.1
        - Midterm1 : 25%
        - Midterm2 : 25%
        - Final : 50%
    - 점수 환산 opt.2
        - Midterm1 : 50%
        - Midterm2 : 50%
    - 점수 환산 opt.3
        - Final : 100% 
- Requirements
    - Grades Table에서 opt.3이 가장 유리한 학생들의 Name, ID를 출력
    - 정렬 기준
        - Name의 좌측부터 3문자를 기준으로 오름차순
        - ID 기준 오름차순
- Solution
```sql
CREATE PROCEDURE gradeDistribution()
BEGIN
    select Name, ID from Grades
    where Final > Midterm1 * 0.25 + Midterm2 * 0.25 + Final * 0.5 && Final > Midterm1 * 0.5 + Midterm2 * 0.5
    order by left(Name, 3) asc, ID asc;
END
```
---------------------------------------------------

#### 10. mischievousNephews
- Preconditions
    - mischief Table은 mischief_date, author, title 필드로 구성
    - mischief_date는 YYYY-MM-DD 형식
    - author는 "Dewey", "Huey", "Loiue" 중 하나
- Requirements
    - mischief Table에서 mischief_date 기준 요일(0 : 월요일) 필드를 weekday 로 추가
    - 정렬 기준
        - weekday 기준 오름차순
        - author를 "Huey", "Dewey", "Louie" 순서로 정렬
        - mischief_date 기준 오름차순
        - title 기준 오름차순
- Solution
```sql
CREATE PROCEDURE mischievousNephews()
BEGIN
	select weekday(mischief_date) as weekday, mischief_date, author, title 
    from mischief 
    order by weekday asc, field(author, "Huey", "Dewey", "Louie") asc, mischief_date asc, title asc;
END
```
---------------------------------------------------
