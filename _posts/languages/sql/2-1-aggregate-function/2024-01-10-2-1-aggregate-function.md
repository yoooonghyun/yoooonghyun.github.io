---
title: 2.1 Aggregate Function (SQL)
date: 2024-01-10 14:10:00 +0800
categories: [Languages, SQL]
tags: [database, sql]
render_with_liquid: false
pin: true
mermaid: true
---

이번 포스트에서는 SQL에서 데이터를 효과적으로 집계하는 데 사용되는 집계 함수에 대해서 다룰 예정이다. Chapter 1에서 SELECT 구문을 통해 raw 데이터를 조회하는 방법을 배웠다. 하지만 때때로 특정 조건에 해당하는 row 수를 알아내거나, 특정 컬럼의 합계, 평균, 최소값, 최대값을 알아야 할 때가 있다. 이럴 때 사용하는 것이 바로 집계 함수들이다. SQL에서 지원하는 집계함수로는 COUNT, SUM, AVG, MIN, MAX 등이 있다.

## COUNT
먼저, COUNT 함수는 조회된 row의 수를 카운팅하는 함수다. 사용 방법은 다음과 같다:

``` sql
SELECT COUNT(col_name) FROM table_name;
```

_Table 0: Customer Table_

|name |created_at|gender|age|num_uses|
|-----|----------|------|---|--------|
|김철수|2022-01-01|male  |20 |   3    | 
|이영희|2022-03-01|female|20 |   5    |
|홍길동|2022-04-01|male  |30 |   4    |
|조용현|2022-02-01|male  |25 |   5    |
|홍길순|2022-03-01|female|25 |   3    |

Customer 테이블에서 전체 회원 수를 조회하려면 다음과 같은 쿼리를 작성할 수 있다:

``` sql
SELECT COUNT(name) FROM customer;

-- Esterisk를 사용해도 무방하다.
SELECT COUNT(*) FROM customer;

-- 조회 결과:
-- +-----+
-- |count|
-- +-----+
-- |  5  |
-- +-----+
```

이 때 COUNT 함수 안에 들어가는 컬럼명은 일반적으로 어떤 값을 넣든 조회된 row의 개수를 반환한다.

또한 이전에 배운 WHERE 구문을 조합하여 남성회원의 수를 조회한다면 다음과 같이 쿼리를 작성할 수 있다:

``` sql
SELECT COUNT(*) FROM customer WHERE gender = 'male';

-- 조회 결과:
-- +-----+
-- |count|
-- +-----+
-- |  3  |
-- +-----+
```

## SUM

SUM 함수는 숫자 데이터의 합계를 반환한다. Customer Table의 num_uses는 회원별 사용 횟수를 의미한다. 그리고 전체 회원의 사용 횟수는 모든 회원의 num_uses를 합하여 계산할 수 있다. SUM 함수를 이용한다면 sum_uses 합을 손쉽게 구할 수 있다:

``` sql
SELECT SUM(num_uses) FROM customer;

-- 조회 결과:
-- +-----+
-- | sum |
-- +-----+
-- |  20 |
-- +-----+
```

SUM 함수도 COUNT와 마찬가지로 WHERE문을 조합하여 쿼리를 작성할 수 있다. 2022년 3월 1일 이전에 가입한 회원들의 총 사용횟 수를 조회 할 수 있다:


``` sql
SELECT SUM(num_uses) FROM customer WHERE created_at < '2022-03-01';

-- 조회 결과:
-- +-----+
-- | sum |
-- +-----+
-- |  8  |
-- +-----+
```

## AVG

AVG 함수는 데이터의 평균을 반환하는 함수이다. num_uses와 AVG를 이용하여 전체 회원 평균 사용 횟수를 조회할 수 있다:

``` sql
SELECT AVG(num_uses) FROM customer;

-- 조회 결과:
-- +-----+
-- | avg |
-- +-----+
-- |  4  |
-- +-----+
```

AVG 함수에 WHERE문을 조합하여 20대 회원의 평균 나이를 구할수 있다:

``` sql
SELECT AVG(age) FROM customer WHERE age BETWEEN 20 AND 29;

-- 조회 결과:
-- +-----+
-- | avg |
-- +-----+
-- | 22.5|
-- +-----+
```

## MIN/MAX

MIN/MAX 함수는 주어진 데이터에서 최소/최대 값을 구하는 함수이다. 다음의 예제와 같이 사용할 수 있다:

``` sql
-- 최초 회원 가입 일시
SELECT MIN(created_at) FROM customer;

-- 조회 결과:
-- +------------+
-- |     min    |
-- +------------+
-- | 2022-01-01 |
-- +------------+

-- 전체 회원 중 최대 사용 횟수
SELECT max(num_uses) FROM customer;

-- 조회 결과:
-- +-----+
-- | max |
-- +-----+
-- |  5  |
-- +-----+
```