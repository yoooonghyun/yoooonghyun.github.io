---
title: 1.3 In/Like/Between (SQL)
date: 2023-12-25 14:10:00 +0800
categories: [Languages, SQL]
tags: [database, sql]
render_with_liquid: false
pin: true
mermaid: true
---


데이터를 조회하는 업무를 진행하거나, 개발을 하면서 조회를 하기 위한 쿼리를 작성하는 경우, 기본 연산자만으로는 기능적인 한계가 있다. 이번 포스트에서는 WHERE 절을 더욱 효과적으로 사용할 수 있게 해주는 IN, BETWEEN, LIKE 구문에 대해 다룰 예정이다.

## IN

먼저, IN 구문에 대해 알아보자. 이 구문은 특정 컬럼이 여러 값 중 하나에 해당할 때 사용된다. 예를 들어, 여러 회원의 이름을 한 번에 조회하고 싶을 때를 생각해보면 아래와 같이 쿼리문을 작성할 수있다:

``` sql
SELECT columns... FROM table_name WHERE col_name = val1 OR col_name = val2;
```

이 구문을 IN을 이용하여 작성하면 다음과 같이 표현할 수 있다: 

``` sql
SELECT columns... FROM table_name WHERE col_name IN (val1, val2);
```

_Table 0: Customer Table_

|name |created_at|gender|address         |email            |
|-----|----------|------|----------------|-----------------|
|김철수|2022-01-01|male  |서울 강남구 역삼동...|ironsoo@naver.com|
|이영희|2022-03-01|female|서울 강남구 서초동...|202@gmail.com    |
|홍길동|2022-04-01|male  |서울 서초구 잠원동...|hkd@gmail.com    |

 Customer Table을 통한 예시를 들어보자. 김철수, 김영희에 해당하는 회원정보를 출력한다고 했을떄 name column에 IN 구문을 사용하여 쿼리를 작성할 수 있다.


``` sql
SELECT * FROM customer WHERE name IN ('김철수', '김영희');
-- 조회 결과:
-- +--------+------------+--------+------------------+-------------------+
-- |  name  | created_at | gender |      address     |       email       |
-- +--------+------------+--------+------------------+-------------------+
-- | 김철수  | 2022-01-01 |  male  | 서울 강남구 역삼동... | ironsoo@naver.com |
-- | 이영희  | 2022-03-01 | female | 서울 강남구 서초동... | 202@gmail.com     |
-- +-------+-------------+--------+------------------+-------------------+

```

## BETWEEN

이어서, BETWEEN 구문을 살펴보자. 이 구문은 특정 범위의 값, 예를 들어 날짜나 숫자 범위를 조회할 때 유용하다. 만약 특정 column이 일정 범위에 있는 데이터를 조회하려 할때는 다음과 같이 AND를 이용하여 표현할 수 있다.

``` sql
SELECT columns... FROM table_name WHERE col_name <= val1 AND col_name >= val2;
```

이 구문을 BETWEEN을 이용하여 작성하면 다음과 같이 표현할 수 있다: 

``` sql
SELECT columns... FROM table_name WHERE col_name BETWEEN val1 AND val2;
```

Table 0을 예로 들어 3월 1일부터 4월 30일까지 가입한 회원을 조회하고 싶다면, BETWEEN 구문을 이용하여 이렇게 쿼리문을 작성할 수 있다:

``` sql
SELECT * FROM customer WHERE created_at BETWEEN '2022-03-01' AND '2022-04-30';
-- 조회 결과:
-- +--------+------------+--------+------------------+-------------------+
-- |  name  | created_at | gender |      address     |       email       |
-- +--------+------------+--------+------------------+-------------------+
-- | 이영희  | 2022-03-01 | female | 서울 강남구 서초동... | 202@gmail.com     |
-- | 홍길동  | 2022-04-01 |  male  | 서울 서초구 잠원동... | hkd@gmail.com     |
-- +-------+-------------+--------+------------------+-------------------+
```

BETWEEN은 시작 값과 끝 값 모두를 포함하는 범위를 지정한다는 점을 기억하자. 만약 시작과 끝을 포함하고싶지 않다면, `>`와 `<` 같은 비교 연산자를 사용해야한다.

## LIKE

마지막으로, LIKE 구문에 대해 알아보자. 이 구문은 문자열 검색에 사용되며, 특히 일부 문자열이 일치하는 경우를 찾을 때 유용하다. 문자열의 일치여부는 `=` 연산자를 이용하여 쿼리문을 작성할 수있따. 하지만 문자열의 시작/끝/일부가 일치하는 경우는 비교연산자를 이용하여 표현할 수 없다. 이때 사용할 수 있는 구문이 LIKE 구문이다.

LIKE 를 이용한 쿼리는 다음과 같이 작성할 수 있다:

``` sql
SELECT columns... FROM table_name WHERE col_name LIKE pattern;
```

### LIKE pattern

위의 쿼리문에서 기본적인 LIKE pattern은 문자열과 wildcard를 이용하여 표현할 수 있다.

- 일치
    - 검색하려는 문자열을 그대로 입력한다.
    - e.g. `name LIKE '김철수'` -- name이 '김철수'인 데이터 조회
- _ wildcard
    - 한 개의 문자를 대체
    - e.g. `name LIKE '_철수'` -- name이 'X철수'인 데이터 조회
    - e.g. `email LIKE '김__'` -- name이 '김XX'인 데이터 조회
- % wildcard
    - 0개 이상의 문자를 대체
    - e.g. `email LIKE '%@gmail.com'` -- email이 '@gamil.com'으로 끝나는 데이터 조회
    - e.g. `address LIKE '%강남구%'` -- address에 '강남구'가 들어가는 데이터 조회
  

Table 0을 예로 들어, 이메일 주소가 '@gmail.com'으로 끝나는 회원을 조회하고 싶다면, LIKE 구문을 이렇게 사용할 수 있다:

``` sql
SELECT * FROM customer WHERE email LIKE '%@gmail.com';
-- 조회 결과:
-- +--------+------------+--------+------------------+-------------------+
-- |  name  | created_at | gender |      address     |       email       |
-- +--------+------------+--------+------------------+-------------------+
-- | 이영희  | 2022-03-01 | female | 서울 강남구 서초동... | 202@gmail.com     |
-- | 홍길동  | 2022-04-01 |  male  | 서울 서초구 잠원동... | hkd@gmail.com     |
-- +-------+-------------+--------+------------------+-------------------+
```

여기서 % 기호는 어떤 문자열이든 상관없다는 것을 의미한다. 즉, '@gmail.com'으로 끝나는 모든 이메일을 찾을 수 있다.

이상으로 IN, BETWEEN, LIKE 구문의 기본적인 사용 방법을 알아보았다. 이 구문들을 잘 활용하면 SQL 쿼리 작성이 훨씬 효과적으로 변한다.