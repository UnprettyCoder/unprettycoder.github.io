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
- Preconditions : Projects Table은 internal_id, project_name, team_size, team_lead, income 필드로 구성
- Requirements : Projects Table에서 internal_id, team_size 필드 없이 출력
- Solution
~~~ mysql
CREATE PROCEDURE projectList()
BEGIN
  alter table Projects drop internal_id;
  alter table Projects drop team_size;
    
  select * from Projects;
  /* select project_name, team_lead, income from Projects; 로 대체 가능 */
END
~~~
---------------------------------------------------

#### 2. countriesSelection
- Preconditions : countries Table은 name, continent, population 필드로 구성
- Requirements : countries Table에서 continent == "Africa" 인 레코드를 name 기준 오름차순 정렬
- Solution
~~~ mysql
CREATE PROCEDURE countriesSelection()
BEGIN
	select * from countries where continent = "Africa" order by name;
END
~~~
---------------------------------------------------

#### 3. monthlyScholarships
- Preconditions : scholarship Table은 id, scholarship 필드로 구성
                  scholarship의 값은 연간 장학액수
- Requirements : scholarship Table에서 id, scholarship을 출력하되, scholarship은 연간이 아니라 월간으로 출력
- Solution
~~~ mysql
CREATE PROCEDURE monthlyScholarships()
BEGIN
	select id, scholarship/12 as scholarship from scholarships;
END
~~~
