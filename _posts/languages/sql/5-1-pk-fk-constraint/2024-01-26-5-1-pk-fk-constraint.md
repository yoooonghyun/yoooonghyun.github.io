---
title: 5.1 PK/FK Constraint
date: 2024-01-26 14:10:00 +0800
categories: [Languages, SQL]
tags: [database, sql]
render_with_liquid: false
pin: true
mermaid: true
---
SQL에서는 DQL, DDL, DML 총 3가지 분류의 명령을 지원한다.

**DQL**
- Data query language, DQL은 데이터 조회를 하기위한 구문이다. 
- `SELECT`로 시작하는 구문이 DQL에 해당하며, 이전 챕터까지는 DQL에 대해서 다뤘다.

**DDL**
- Data definition language, DDL은 데이터를 정의하기 위한 구문이다.
- 테이블의 생성과 삭제 그리고 테이블에 존재하는 Column과 제약조건의 생성, 변경, 삭제를 수행해주는 구문이 DDL이다. 
- `CREATE`, `ALTER`, `DROP`으로 시작하는 명령들이 DDL에 해당한다.

**DML**
- Data manipulation language, DML은 DB에서 데이터 즉 record를 저장, 삭제, 변경하는 구문이다.
- `INSERT`, `UPDATE`, `DELETE가` 이에 해당한다.

Chap5에서는 이중 DDL에 집중해서 다룬다. 이번 포스트에서는 DDL 구문을 직접 다루기 전에 `Primary Key`, `Foriegn Key`, `Contraint`에 대해서 설명한다. 이 개념들은 테이블을 구성하고 있는 정보들로, DDL 구문을 작성하는데 사용되는 필수 적인 개념입니다.


_Table 0: Customer Table_

|name |created_at|gender|age|num_uses|
|-----|----------|------|---|--------|
|김철수|2022-01-01|male  |20 |   3    | 
|이영희|2022-03-01|female|20 |   5    |
|홍길동|2022-04-01|male  |30 |   4    |
|조용현|2022-02-01|male  |25 |   5    |
|홍길순|2022-03-01|female|25 |   3    |

_Table 1: Payment Table_

|payment_id|customer_id|amount|
|----------|-----------|------|
|     1    |     3     | 2.99 |
|     2    |     2     | 4.99 |
|     3    |     4     | 0.99 |
|     4    |     3     | 2.99 |
|     5    |     1     | 4.99 |

## Primary Key

Primary Key 혹은 약자로 PK라고도 부르는 이 값은 테이블에서 각 레코드를 특정하기 위한 식별값이다. 흔히 _id로 정의하는 값으로, customer 테이블의 customer_id, payment 테이블의 payment_id가 이에 해당한다. 각 레코드를 특정해야하기 때문에 중복이 불가능하며, 중복을 피하기 위해서 serial이나 uuid로 데이터 타입을 정의하는게 일반적이다.

- serial
    - 순차적으로 증가하는 숫자
    - 레코드 생성을 시도할때마다 1씩 증가시켜서 값의 중복을 피한다
- uuid
    - 32자리의 16진수 값
    - 중복이 발생할수 있지만 그 확률이 매우 작기 때문에 유니크한 값을 부여할때 사용
    
PK는 테이블에 필수로 정의되어 있어야하며, NULL을 지정하는 것이 불가능하다. 그리고 테이블을 검색을 효과적으로 해주는 index 정보를 자동으로 생성합니다.

## Foreign Ky
Foreign key 혹은 FK는 테이블간의 mapping을 특정하기 위한 참조값이다. 외부 테이블과의 join을 위해서 사용할 PK를 미리 column 값으로 갖고 있는다. 보통 FK는 자식 테이블에서 가지고 있다. 테이블간의 부모, 자식 관계는 두 테이블간에 연관이 있을때 의존성이 어떤 방향에 따라 정해진다. payment와 customer를 예로들면 customer는 payment 없이 존재할 수 있지만 payment는 customer없이 존재할수가 없다. 따라서 payment는 customer에 의존성이 있고, payment가 자식, customer가 부모 테이블이 된다. 그리고 payment 테이블의 customer_id는 FK가 된다.


## Constraint
Constraint는 잘못된 사용을 막기위해서 금지하는 제약 사항들을 정의해 놓은 것을 의마한다. DDL, DML 구문을 통해서 어떤 값을 저장하거나 변경하려 할때는 Constraint를 확인하여, Constraint로 정의한 금지된 내용들을 수행하려고 할때는 error를 반환한다. 가령, PK또한 하나의 Contraint에 해당하며, 중복된 값을 저장하려하면 error를 반환하며 쿼리문이 실패한다.

Contraint는 Column에 대한 제약사항을 다루는 Column Constraint와 Table 제약사항에 해당하는 Table Constaint로 나뉘어 있다.

### Column Constraint

아래 테이블은 대표적인 Column Constraint와 그 역할에 대한 설명이다:

_Table 2: Column Constraints_

| Constraint  | 역할 |
|-------------|-----|
| Not Null    | Column에 null값을 저장하기 못하게 한다. |
| Unique      | Column에 레코드간 중복값을 사용하지 못하게 한다. |
| Primary Key | Column을 PK로 지정한다. |
| Foriegn Key | Column을 FK로 지정한다. |
| Check       | 값이 check 조건으로 명시한 특정 조건을 만족하는지 검사한다. |

### Table Constraint

아래 테이블은 대표적인 Table Constraint와 역할에 대한 설명이다. Column Contraint와 동일한 이름의 Contraint도 존재하는 것을 확인할수 있다. Column Constraint로 지정된 Constraint는 하나의 컬럼을 이용한 제약 사항을 설정할 수 있다. 하지만 Table Constraint를 이용하면 여러 Column을 이용한 제약사항을 설정할 수 있습니다.

_Table 2: Column Constraints_

| Constraint | 역할 |
|------------|-----|
| Check | 레코드가 추가 되거나 변경될때 특정 조건을 검사합니다. |
| References | 저장할 값일 다른 테이블에 존재하는 값인지 검사합니다. |
| Unique (column list) | Table contraint로 지정하면 1개 이상 지정한 컬럼의 조합값이 unique합니다. |
| Primary Key (column list) | 1개 이상 지정한 컬럼의 조합이 PK가 됩니다. |