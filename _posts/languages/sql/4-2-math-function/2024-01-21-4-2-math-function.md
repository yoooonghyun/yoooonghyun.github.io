---
title: 4.2 Mathematical Functions (SQL)
date: 2024-01-21 14:10:00 +0800
categories: [Languages, SQL]
tags: [database, sql]
render_with_liquid: false
pin: true
mermaid: true
---

SQL혹은 다른 대부분의 프로그래밍 언어에서도 값을 계산/변환하는 방법으로 연산자와 함수를 제공하고 있습니다.

## 연산자
연산자는 수학기호로 포현할수 있는 연산을 지원한다. 예를 들어 `1 + 1`이라는 표현식이 존재할 때 `+`가 연산자에 해당한다. SQL은 숫자/문자/시간과 관련된 연산자를 제공한다.

## 함수
연산자를 두개의 값 혹은 하나의 값에 대한 연산만을 지원하기 떄문에 복잡한 기능을 지원하기 어렵다. 하지만 함수를 통해서 비교적 본작한 기능을 구현할 수 있다. `round(-1.1)` 라는 표현식은 소수를 반올림하는 동작을 한다. 이처럼 함수는 `함수명-괄호-파라미터`로 구성되어 있는데, 함수에 따라 여러개의 파라미터를 요구할 수 있다. SQL에서는 함수도 숫자/문자/시간과 관련하여 제공한다. 하지만 이 외에도 복합적 동작을 위한 함수를 제공하며, 사용자가 직접 함수를 생성할 수 도 있다.

## 수학연산자 (Mathematical Operators)

PostgreSQL에서 지원하는 수학관련된 함수와 연산자는 [[0]](https://www.postgresql.org/docs/9.5/functions-math.html)에 기술되어 있다.

이중 주요한 수학연산자는 다음의 테이블과 같다.

_Table 0: Mathematical Operators_

| 연산 |  설명  |  예시  | 결과 |
|-----|-------|-------|-----|
|  +  | 더하기  | 3 + 2 |  5 |
|  -  |  빼기  | 3 - 2 |  1  |
|  *  | 곱하기  | 3 * 2 |  6 |
|  /  | 나누기  | 3 / 2 |  1 |
|  %  | 나머지  | 3 % 2 |  1 |
|  ^  |  제곱  | 3 ^ 2 |  9  |

### 수학연산자 예시

_Table 1: Payment Table_

|payment_id|customer_id|amount|
|----------|-----------|------|
|     1    |     3     | 2.99 |
|     2    |     2     | 4.99 |
|     3    |     4     | 0.99 |
|     4    |     3     | 2.99 |
|     5    |     1     | 4.99 |

쿼리에 수학연산자가 사용되는 예시를 보자. Payment 테이블에는 각 결제건 별로 대여 요금에 해당하는 amount는 USD로 표시되어 있다. 이를 원으로 환산해서 보고 싶다면, 현재 환율을 곱해서 금액을 조회하면 된다.
현재 환율이 1300 KRW/USD라고 했을 때 대여 요금의 KRW를 조회하는 쿼리문은 다음과 같다:
``` sql
SELECT payment_id AS payment_id, amount * 1300 AS amount_won FROM payment;
-- 조회 결과:
-- +----------+----------+
-- |payment_id|amount_won|
-- +----------+----------+
-- |     1    |   3889   |
-- |     2    |   6487   |
-- |     3    |   1287   |
-- |     4    |   3887   |
-- |     5    |   6487   |
-- +----------+----------+
```

## 수학함수 (Mathematical Functions)

다음으로는 수학 함수에 대해서 대표적인 수학 함수들을 추린 표이다.

_Table 2: Mathematical Functions_

| 함수                       |  반환 타입  | 설명          | 예시               | 결과   |
|---------------------------|-----------|--------------|------------------|-------|
| div(y numeric, x numeric) | numeric   | 나눗셈         | div(9,4)         | 2     |
| ceil(double or numeric)   | input type| 정수로 올림     | ceil(-42.8)      | -42   |
| floor(double or numeric)  | input type| 정수로 내림     | floor(-42.8)     | -43   |
| round(v numeric, s int)   | numeric   | s자리까지 반올림 | round(42.4382, 2)| 42.44 |
| random()                  | double    | 난수 생성      | -                | -     |

### div
첫번째 부터 보면 연산자로도 제공하는 나누기를 함수로도 지원하고 있다. 하지만 편의성이나 가독성에 있어 연산자를 사용하는 것을 권장한다.

### ceil/floor/round
실무에 가장 많이 사용하게될 함수는 올림, 내림, 반올림을 지원하는 ceil, floor, round 함수이다. ceil, floor함수는 함수안에 소수를 넣어 정수로 반환하는 함수이다. 그리고 round함수는 함수안에 숫자와 자릿수를 넣어 원하는 자릿 수 까지 반올림한 결과를 반환한다.

### random
random함수는 0이상 1미만읜 random한 숫자를 반환하는 함수이다.

### 수학함수 예시

[수학연산자 예시](<#수학연산자 예시>)에서 환율을 구한 결과를 보면 1의 자리수까지 표현이 되어 있다. 이를 100의 자리까지 반올림한 결과를 조회하려고한다. ROUND 함수를 이용하여 쿼리를 작성해보면:
``` sql
SELECT payment_id AS payment_id, ROUND(amount * 1300, -2) AS amount_won  FROM payment;
-- 조회 결과:
-- +----------+----------+
-- |payment_id|amount_won|
-- +----------+----------+
-- |     1    |   3900   |
-- |     2    |   6500   |
-- |     3    |   1300   |
-- |     4    |   3900   |
-- |     5    |   6500   |
-- +----------+----------+
```

# References

- [0] : [https://www.postgresql.org/docs/9.5/functions-math.html](https://www.postgresql.org/docs/9.5/functions-math.html)