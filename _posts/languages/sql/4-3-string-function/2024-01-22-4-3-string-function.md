---
title: 4.3 String Functions (SQL)
date: 2024-01-22 14:10:00 +0800
categories: [Languages, SQL]
tags: [database, sql]
render_with_liquid: false
pin: true
mermaid: true
---

SQL에서는 문자열의 처리를 지원하기 위한 연산자와 함수를 제공한다. 이번 포스트에서는 PostgreSQL을 기준으로, SQL에서 제공하는 문자열 처리 연산자/함수에 대해서 알아볼 것이다. 그리고 문자열 연산자/함수를 이용하여 간단한 쿼리문을 작성해본다.

## 문자열 연산자/함수

PostgreSQL에서 지원하는 문자열과 관련된 함수와 연산자는 [[0]](https://www.postgresql.org/docs/9.4/functions-string.html)에 기술되어 있다. 이중 주요한 기능을 추려 Table 0에 표현해 보았다.

_Table 0: String Operators & Functions_

| 연산                                               | 리턴 타입| 설명               | 예시                                       | 결과        |
|---------------------------------------------------|--------|-------------------|-------------------------------------------|------------|
| string \|\| string                                | string | 병합               | 'ice' \|\| 'cream'                        | 'icecream' |
| string \|\| non-string                            | string | 병합               | 'a = ' \|\| 1                             | 'a = 1'    |
| char_length(string)                               | int    | 문자열 길이          | char_length('cat')                        | 3          |
| overlay(string placing string from int [for int]) | string | 문자열 변경          | overlay('sky' placing 'now' from 2 for 2) | 'snow'     |
| position(substring in string)                     | int    | 문자의 위치          | position('day' in 'today')                | 3          |
| substring(string [from int] [for int])            | string | 문자열 추출          | substring('line' from 2 for 3)            | 'ine'      |
| lower(string) / upper(string)                     | string | 소문자 / 대문자로 변경 |  -                                        | -          |
         
### 병합 `||`

문자열을 병합하는 연산자로 vertical line이라고 불리는 기호를 두개 입력해서 사용할 수 있다. 이 기호가 생소하신 분들도 있을텐데, 오른쪽 엔터키 위에 있는 원화 기호(₩)를 shift와 함께 누르면 입력할수 있다.

또한 두번째 줄의 사용과 같이 문자열이 아닌 값과 해당 연산자를 사용한다면, 문자열로 자동 변환되어 연산을 수행합니다.

### char_length

char_length 함수는 문자열을 파라미터로 넣어 실행하면 문자열의 길이를 반환하는 함수이다.

### overlay

overlay함수는 문자열의 일부를 치환하는 함수로 그 사용법이 다소 복잡하다. `overlay(string placing string from int [for int])` 와 같이 입력하는데 from과 to 뒤에 의미하는 숫자는 다음을 의미한다:

- from: 변경할 문자열의 시작
- for: 변경할 문자열의 수

### position

position 함수는 문자열 내에서 특정 문자/문자열의 위치를 찾아낸다. 예시의 day 라는 문자열은 today의 3번째부터 시작한다.

### substring

substring함수는 문자열의 일부를 추출하기 위한 함수이다. 추출 범위는 overlay함수와 마찬가지로 from-for의 형태로 입력한다. 예시에서는 line이라는 문자열에서 ine라는 문자열을 추출했다.

### lower / upper

대소문자 변환을 위한 lower와 upper함수를 지원합니다.

### 문자열 함수 / 연산자 예시

_Table 1: Customer Table_

|name |address      |detailed_address |email            |
|-----|-------------|-----------------|-----------------|
|김철수|서울 강남구 역삼동|201동 105호       |ironsoo@naver.com|
|이영희|서울 강남구 서초동|103동 201호       |202@gmail.com    |
|홍길동|서울 서초구 잠원동|205동 302호       |hkd@gmail.com    |

위의 Customer 테이블에서 string 함수를 이용한 쿼리문을 작성해보자.

- Q1: 각 회원의 성을 출력하는 쿼리문
- Q2: 각 회원의 전체주소 (address + detailed_address)
- Q3: 각 회원별 사용하는 이메일 도메인

#### Q1

각 회원의 성을 출력하기 위해서는 이름을 저장하는 `name` 컬럼의 값에서 첫번째 문자만 취하면 된다. 문자열을 나누는 함수인 `substring` 함수를 이용하면 다음과 같이 쿼리를 작성할 수 있다: 

``` sql 
SELECT name, SUBSTRING(name) AS family_name FROM customer;
-- 조회 결과:
-- +------+-----------+
-- |name  |family_name|
-- +------+-----------+
-- |'김철수'|    '김'   |
-- |'이영희'|    '이'   |
-- |'홍길동'|    '홍'   |
-- +------+-----------+
```

#### Q2

각 회원의 주소는 `address`, `detailed_address` 컬럼에 나누어 저장되어있다. 사용자의 전체 주소를 출력하기 위해서는 두 문자열을 병합한 결과를 출력하면 된다. 

``` sql 
SELECT name, address || ' ' || detailed_address AS full_address FROM customer;
-- 조회 결과:
-- +------+---------------------------+
-- |name  |full_address               |
-- +------+---------------------------+
-- |'김철수'|'서울 강남구 역삼동 201동 105호'|
-- |'이영희'|'서울 강남구 서초동 103동 201호'|
-- |'홍길동'|'서울 서초구 잠원동 205동 302호'|
-- +------+---------------------------+
```

#### Q3

이메일 도메인 @ 기호 뒤에 이어진다. 따라서 @ 기호의 위치에 1을 더한 값이 도메인의 시작이다. @의 위치를 position 함수를 이용하여 구하고, substring 함수의 위치를 지정한다면 이메일의 도메인을 찾을 수 있다.

``` sql 
SELECT name, SUBSTRING(email FROM POSITION('@' IN email) + 1) AS domain FROM customer;

-- 조회 결과:
-- +------+-----------+
-- |name  |domain     |
-- +------+-----------+
-- |'김철수'|'naver.com'|
-- |'이영희'|'gmail.com'|
-- |'홍길동'|'gmail.com'|
-- +------+-----------+
```

# References

- [0] : [https://www.postgresql.org/docs/9.5/functions-string.html](https://www.postgresql.org/docs/9.5/functions-string.html)
