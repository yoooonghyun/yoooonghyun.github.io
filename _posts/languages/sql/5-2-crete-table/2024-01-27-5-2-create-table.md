---
title: 5.2 Create Table
date: 2024-01-27 14:10:00 +0800
categories: [Languages, SQL]
tags: [database, sql]
render_with_liquid: false
pin: true
mermaid: true
---

## 테이블 생성 구문

`CREATE TABLE`은 테이블을 생성하기 위한 DDL구문이다. 테이블 생성 구문은 다음과 같이 사용할 수 있다:

``` sql
CREATE TABLE tb_name (
    col_name_1 TYPE column_constraint_1,
    col_name_2 TYPE column_constraint_2,
    ...
    CONSTAINT table_constraint_1,
    CONSTAINT table_constraint_2,
    ...);
```
여기서 필수로 정의해야하는 정보는 `테이블명`, `컬럼명`, `TYPE`으로, Column Constraint와 Table Constraint는 필요하다면 추가로 정의하면 된다.

## Employee 테이블 생성

_Table 0: Employee Table_

| employee_id | name | created_at          | address           | email             |
|-------------|------|---------------------|-------------------|-------------------|
|      1      | 김철수 | 2023-01-01 00:00:00 | 서울 강남구 역삼동... | ironsoo@naver.com |
|      2      | 이영희 | 2023-01-02 15:20:00 | 서울 강남구 서초동... | 202@gmail.com     |
|      3      | 홍길동 | 2023-01-03 09:55:00 | 서울 서초구 잠원동... | hkd@gmail.com     |


한번 CREATE TABLE구문을 이용하여 위에 보이는 Employee 테이블을 생성해 보자. Employee 테이블을 보면 다음과 같은 컬럼으로 구성을 가지고있다.

- employee_id: 식별값 (PRIMARY KEY). 순차적으로 증가 (SERIAL).
- created_at: 등록일시 (TIMESTAMP).
- name: 직원의 이름 (TEXT).
- email: 직원의 이메일 (TEXT). 유니크한 값 (UNIQUE).

모든 컬럼이 NULL을 허용하지 않는다할때 employee 테이블을 생성하는 쿼리문은 다음과 같다:

``` sql
CREATE TABLE employee (
    employee_id SERIAL PRIMARY KEY,
    created_at TIMESTAMP NOT NULL,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL);
```

그리고 PK는 Table Constraint를 통해서도 지정할수 있다:

``` sql
CREATE TABLE employee (
    employee_id SERIAL,
    created_at TIMESTAMP NOT NULL,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    CONSTRAINT employee_pkey PRIMARY KEY (employee_id));
```

## Job 테이블 생성

_Table 1: Job Table_

| job_id | name | created_at          |
|--------|------|---------------------|
| 1      | 개발자 | 2023-01-01 00:00:00 |
| 2      | 기획자 | 2023-01-02 00:-0:00 |

직원을 의미하는 Employee 테이블을 생성했으니, 직책을 의미하는 Job 테이블을 생성해보자. Job 테이블의 컬럼 구성은 다음과 같다.

- job_id: 식별값 (PRIMARY KEY), 순차적으로 증가 (SERIAL).
- created_at: 등록일시 (TIMESTAMP).
- name: 직책의 이름 (TEXT).

Job 테이블 또한 모든 컬럼이 NULL을 허용하지 않는다할 때 Job 테이블을 생성하는 쿼리문은 다음과 같다:

``` sql
CREATE TABLE job (
    job_id SERIAL PRIMARY KEY,
    created_at TIMESTAMP NOT NULL,
    name TEXT NOT NULL);
```

## Employee 테이블과 Job 테이블의 연결

일반 적인 회사라면 직원에게는 직책이 부여된다. 직원에게 직책을 부여하기 위해선, Employee 테이블과 Job 테이블 사이의 연결관계를 만들어야한다.

연결관계를 만들기위해 Employee 테이블에 job.job_id 를 가리키는 FK를 추가할수도 있지만, 이번 포스트에서는 두 테이블 사이를 연결하는 EmployeeJob 테이블을 만들것이다.

_Table 2: EmployeeJob Table_

| employee_id | job_id | hired_at |
|--------|------|---------------------|
|||

- employee_id: 직원의 식별값 (FOREIGN KEY). employee.employee_id 를 가리킴 (REFERENCES ...).
- job_id: 직책의 식별값 (FOREIGN KEY). job.job_id 를 가리킴 (REFERENCES ...).
- hired_at: 직책을 지정한 일시 (TIMESTAMP).

EmployeeJob 테이블의 PK는 employee_id와 job_id의 조합으로 지정한다 할때, EmployeeJob 테이블의 생성 쿼리문은 다음과 같다:

``` sql
CREATE TABLE job_employee (
    job_id INTEGER NOT NULL,
    employee_id INTEGER NOT NULL,
    hired_at TIMESTAMP NOT NULL,
    CONSTRAINT job_employee_pkey PRIMARY KEY (job_id, employee_id),
    CONSTRAINT job_employee_job_fkey FOREIGN KEY (job_id) REFERENCES job(job_id),
    CONSTRAINT job_employee_employee_fkey FOREIGN KEY (employee_id) REFERENCES employee(employee_id));  
```

이렇게 EmployeeJob 테이블을 생성함으로써 Job과 Employee의 연결관계를 저장하고, `JOIN`을 이용하여 조회할 수있다.