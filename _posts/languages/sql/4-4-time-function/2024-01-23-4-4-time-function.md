---
title: 4.4 Time Functions (SQL)
date: 2024-01-23 14:10:00 +0800
categories: [Languages, SQL]
tags: [database, sql]
render_with_liquid: false
pin: true
mermaid: true
---

SQL에서는 Timestamp와 Date 타입과 관련된 연산자와 함수를 제공한다. 이번 포스트에서는 PostgreSQL을 기준으로, SQL에서 제공하는 Timestamp와 Date를 처리하는 연산자/함수에 대해서 알아볼 것이다. 그리고 시간 연산자/함수를 응용하여 간단한 쿼리문을 작성해본다.

## 시간 타입

연산자와 함수에 대해서 알아보기 전에 시간 타입에 대해서 알아보자. 이번 포스트에서 다루는 시간타입은 크게 Date, Timestamp, Interval이 있다.

- Date
    - 날짜를 의미하는 자료형으로 '년도-월-일'을 의미하는 'YYYY-MM-DD' 의 형태로 값의 정의가 가능하다.
- Timestamp
    - 날짜와 시간이 함께 있는 형태로 '년도-월-일 시:분:초'을 의미하는 'YYYY-MM-DD HH:MI:SS'의 형태로 값을 정의할 수 있다.
- Interval
    - Interval은 Timestamp간의 차이를 표현하는 자료형이다.
    - INTERVAL '1 day'와 같은 형태로 숫자와 년/월/일과 같은 기간을 나타내는 약속된 단어들을 이용하여 정의할 수 있다. 또한 약어를 이용하여 INTERVAL 1 d와 같이 정의도 가능하다.

## 시간 연산자

PostgreSQL에서 지원하는 시간과 관련된 함수와 연산자는 [[0]](https://www.postgresql.org/docs/9.5/functions-datetime.html)에 기술되어 있다. 이중 주요한 연산자를 추려 Table 0에 표현해 보았다.

_Table 0: Time Operators_

| 연산                   | 리턴 타입   | 예시                                                         | 결과                 |
|-----------------------|-----------|-------------------------------------------------------------|---------------------|
| Date ± Integer        | Date      | DATE '2023-01-01' + 7                                       | 2023-01-08          |
| Date ± Interval       | Timestamp | DATE '2023-01-01' + INTERVAL '1 hour'                       | 2023-01-01 01:00:00 |
| Interval ± Interval   | Interval  | INTERVAL '1 day' + INTERVAL '1 hour'                        | 1 day 01:00:00      |
| Timestamp ± Interval  | Timestamp | TIMESTAMP '2023-01-01 00:00' + INTERVAL '23 hours'          | 2023-01-01 23:00:00 |
| Timestamp - Timestamp | Interval  | TIMESTAMP '2023-03-01 01:00' - TIMESTAMP '2023-01-01 01:00' | 59 days             |

위 시간 연산자에 대한 부연설명은 아래와 같다:

- `Date ± Integer`
    - Date에 +/- 정수를 입력하게되면 정수는 day로 인식하여 증감이 된다.
- `Date ± Interval`
    - Date에 Interval을 더하거나 뺌을로써 과거/미래의 시간정보를 계산할 수 있다. 
    - Interval은 시분초를 모두 포함하는 data type이기 때문에 이 결과는 Timestamp로 반환된다.
- `Interval + Interval`
    - Interval은 Interval과 더하고 뻄으로써 새로운 Interval을 만들수 있다.
- `Timestamp ± Interval`
    - Date와 마찬가지로 Timestamp에 Interval을 더하고 뺄수 있는데 그 결과는 Timestamp이다.
- `Timestamp - Timestamp`
    - Timestamp간에는 - 연산을 할 수 있는데, 그 결과로는 Interval이 반환된다.

## 시간 함수

다음은 시간과 관련된 함수이다. 아래 보이는 Table 1은 자주 사용되는 함수들을 정리한 표이다.

_Table 1: Time Functions_

| 함수                         | 리턴 타입   | 예시                                                 | 결과                    |
|-----------------------------|-----------|-----------------------------------------------------|------------------------|
| now()                       | Timestamp | NOW() or CURRENT_TIMESTAMP()                        | 2023-01-01 20:38:40    |
| to_char(timestamp, text)    | Text      | TO_CHAR(NOW(), 'YYYY년MM월DD일HH24:MI:SS')            | 2023년01월01일 20:38:40 |
| to_timestamp(text, text)    | Timestamp | TO_TIMESTAMP('20230101203840', 'YYYYMMDDHH24MISS')  | 2023-01-01 20:38:40    |
| date_trunc(text, timestamp) | Timestamp | DATE_TRUNC('hour', NOW())                           | 2023-01-01 20:00:00    |

- `now`
    - 현재시각에 해당하는 Timestamp를 반환하는 함수이다.
    - current_timestamp라는 함수로도 동일한 동작을 기대할 수 있다.
    - 예시의 이해를 위해서 화면의 표에서 now의 결과는 모두 동일하게 통일했다.
- `to_char`
    - Timestamp를 지정한 포맷의 문자열로 변환해주는 함수이다. 함수의 첫번째 인자에는 변환할 시간이, 두번째 인자에는 결과 포맷이 들어갑니다.
    - 어떤 포맷들을 지원하는지는 [포맷](<#포맷>)에 기술되어있다.
- `to_timestamp`
    - to_char와는 반대로 문자열과 문자열이 해당하는 시간 포맷을 입려하면 Timestamp로 변환하는 함수이다.
- `date_truncate`
    - 원하는 단위 아래의 시간은 모두 버리는 함수이다. 예시에서는 now의 시간 아래의 값을 버림으로써 38분 40초가 무시되었다.
    - date_truncate함수는 일간,주간,월간등 기간별 통계를 볼때 사용하기 유용한 함수이다.

## 포맷

이전 설명에도 계속 등장했듯이 SQL에서는 시간을 다루기 위해 약속된 포맷이 존재한다. Interval 혹은 date_trunc에서 사용되는 시간의 단위는 `Table 2: Interval Patterns`에 기술되어있다. 첫 번째 행을 해석해보면 year는 년을 나타내고 y라는 약자로 사용할수도 있다.

_Table 2: Interval Patterns_

| Interval | 기간   | 약자 |
|----------|-------|-----|
| year     | 년     | y   |
| month    | 월     | m   |
| week     | 주     | w   |
| day      | 일     | d   |
| hour     | 시     | h   |
| minute   | 분     | m   |
| second   | 초     | s   |
| decade   | 10 년  | dec |
| century  | 100 년 | c   |


그리고 timestamp와 character사이의 변환을 위해서 사용하는 포맷은 `Table 3: Charater Patterns`에 기술되어있다. 기본적으로 년월일시분초를 숫자로 표현할 수 있으며, 요일과 영어로된 월 정보도 제공한다.

_Table 3: Charater Patterns_

| Pattern    | 설명       | 예시        | 
|------------|-----------|------------|
| YYYY/YY    | 년        | 2023/23     |
| Month/Mon  | 월 (문자)  | January/Jan |
| MM         | 월 (숫자)  | 01          |
| DD         | 일        | 01          |
| Day/Dy     | 요일 (문자) | Sunday/Sun |
| D          | 요일 (숫자) | 1          |
| HH/H12/H24 | 시        | 8/8/20     |
| MM         | 분        | 38         |
| SS         | 초        | 40         |

## 시간 함수/연산자의 활용

_Table 4: Payment Table_

| payment_id | customer_id | amount | paid_at                 |
|------------|-------------|--------|-------------------------|
| 1          | 3           | 2.99   | 2023-01-01 05:12:32.000 |
| 2          | 2           | 4.99   | 2023-01-01 16:55:36.000 |
| 3          | 4           | 0.99   | 2023-01-01 21:40:11.000 |
| 4          | 3           | 2.99   | 2023-01-02 09:07:42.000 |
| 5          | 1           | 4.99   | 2023-01-02 14:11:05.000 |

Payment 테이블은 결제 내역을 담은 테이블로 각각의 컬럼은 다음을 의미한다:

- payment_id: 결제 내역의 고유식별값
- customer_id: 결제한 고객의 고유식별값
- amount: 결제 금액
- paid_at: 결제 일시

이제 몇가지 예제를 통해서 Payment로부터 데이터를 조회하는 query문을 작성해보자. 

- Q1: 현재로부터 24시간 이내의 결제 내역 조회
- Q2: 일자별 내역 조회
- Q3: 결제 일시를 양식에 따라 조회 : YYYY년MM월DD일 HH24시MI분SS초

### Q1

Payment 테이블에는 paid_at을 통해서 결제 금액을 저장한다. paid_at이 지금(now)으로부터 24시간을 뺀 값보다 큰 결제건들을 조회하면 쿼리문을 완성할 수 있다.

``` sql
-- now(): 2023-01-03 00:00:00.000
SELECT * FROM payment p WHERE p.paid_at > NOW() - INTERVAL '1 day';

-- 조회 결과:
-- +------------+-------------+--------+-------------------------+
-- | payment_id | customer_id | amount | paid_at                 |
-- +------------+-------------+--------+-------------------------+
-- | 4          | 3           | 2.99   | 2023-01-02 09:07:42.000 |
-- | 5          | 1           | 4.99   | 2023-01-02 14:11:05.000 |
-- +------------+-------------+--------+-------------------------+
```

### Q2

일자별 내역 조회를 위해서는 day 아래의 값들을 제거해야한다. 특정 기준 아래로 절삭하는 동작은 date_truncate함수를 사용하여 수행할수 있다. date_truncate함수를 이용하여 paid_at의 day 아래의 값을 버리게되면, 결제 일자만을 추출할수 있다. 그리고 ORDER BY를 이용하여 결제 일자를 기준으로 정렬하면 결제일 별로 모아진 데이터를 조회할 수 있다.

``` sql
SELECT DATE_TRUNC('day', paid_at) AS paid_date, * FROM payment p ORDER BY paid_date;

-- 조회 결과:
-- +------------+------------+-------------+--------+-------------------------+
-- | paid_date  | payment_id | customer_id | amount | paid_at                 |
-- +------------+------------+-------------+--------+-------------------------+
-- | 2023-01-01 | 1          | 3           | 2.99   | 2023-01-01 05:12:32.000 |
-- | 2023-01-01 | 2          | 2           | 4.99   | 2023-01-01 16:55:36.000 |
-- | 2023-01-01 | 3          | 4           | 0.99   | 2023-01-01 21:40:11.000 |
-- | 2023-01-02 | 4          | 3           | 2.99   | 2023-01-02 09:07:42.000 |
-- | 2023-01-02 | 5          | 1           | 4.99   | 2023-01-02 14:11:05.000 |
-- +------------+------------+-------------+--------+-------------------------+
```

만약, 기간별 통계값이 필요하다면 GROUP BY를 이용하여 쿼리문을 작성할 수도 있다.

### Q3

시간데이터를 문자로 변환하기 위해서는 to_char 함수를 이용하면 된다.

``` sql
SELECT TO_CHAR(paid_at, 'YYYY년MM월DD일 HH24시MI분SS초') AS paid_datetime, * FROM payment p;

-- 조회 결과:
-- +--------------------------+------------+-------------+--------+-------------------------+
-- | paid_datetime            | payment_id | customer_id | amount | paid_at                 |
-- +--------------------------+------------+-------------+--------+-------------------------+
-- | 2023년01월01일 05시12분32초 | 1          | 3           | 2.99   | 2023-01-01 05:12:32.000 |
-- | 2023년01월01일 16시55분36초 | 2          | 2           | 4.99   | 2023-01-01 16:55:36.000 |
-- | 2023년01월01일 21시40분11초 | 3          | 4           | 0.99   | 2023-01-01 21:40:11.000 |
-- | 2023년01월02일 09시07분42초 | 4          | 3           | 2.99   | 2023-01-02 09:07:42.000 |
-- | 2023년01월02일 14시11분05초 | 5          | 1           | 4.99   | 2023-01-02 14:11:05.000 |
-- +--------------------------+------------+-------------+--------+-------------------------+
```

# References

- [0] : [https://www.postgresql.org/docs/9.5/functions-datetime.html](https://www.postgresql.org/docs/9.5/functions-datetime.html)


