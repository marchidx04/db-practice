# SQL(PostgreSQL)

## 목차

- [쿼리 실행 순서](#쿼리-실행-순서)
- [SELECT](#select)
- [비교 연산자 및 기타](#비교-연산자-및-기타)
- [GROUP BY와 집계 함수](#group-by와-집계-함수)
- [JOINS](#joins)
- [집합 연산자](#집합-연산자)
- [Timestamps and Extract](#timestamps-and-extract)
- [서브쿼리(Sub Query)](#서브쿼리sub-query)
- [데이터베이스 및 테이블 생성](#데이터베이스-및-테이블-생성)
- [조건식과 프로시저(Conditional Expressions and Operators)](#조건식과-프로시저conditional-expressions-and-operators)
- [Views](#views)
- [SQL 개요](#sql-개요)
- [Function](#function)
- [TCL](#tcl)
- [!= NULL, <> NULL, IS NOT NULL 차이](#null--null-is-not-null-차이)
- [고급 집계 함수](#고급-집계-함수)
- [윈도우 함수](#윈도우-함수)
- [PL / SQL](#pl--sql)
- [SQL 최적화](#sql-최적화)
- [조인 기법](#조인-기법)
- [데이터 모델링의 이해](#데이터-모델링의-이해)

## 쿼리 실행 순서

```
SQL Query Execution Order
FROM and JOIN -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY -> LIMIT
                              |
                              └----- window functions execute here
```

SQL 쿼리문을 실행하는데 순서가 존재한다.

### FROM (+ JOIN)

- 쿼리의 첫 번째 실행 순서는 FROM절이다. FROM 절에서는 조회하는 **테이블 전체**를 가져온다.\

### WHERE 절

- WHERE 절에서는 FROM 절에서 읽어온 테이블에서 **조건에 맞는 결과만 갖도록** 데이터를 필터링한다.

### GROUP BY 절

- GROUP BY 절에서는 **선택한 컬럼**으로 **그룹핑**한다.
- GROUP BY 절을 사용하게 되면 해당 컬럼으로 집계함수를 사용할 수 있다.

### HAVING 절

- HAVING 절은 여러 조건들을 처리한 후 남은 데이터에서 어떤 열을 출력해줄지 선택한다.
- **HAVING 절은 각 그룹에 조건을 걸기 때문에 퍼포먼스가 떨어지게 된다.**
- WHERE 절에 있는 내용을 HAVING 절에서 사용할 수 있다.
  - GROUP BY로 그룹별 집계함수를 구하는 경우를 제외하고는 WHERE 절로 한 번에 거는 것이 좋다.(현재는 내부적으로 Optimize해준다.)

### SELECT 절

- SELECT 절은 여러 조건들을 처리한 후 남은 데이터에서 **어떤 열을 출력해줄지 선택**한다.

### ORDER BY 절

- 어떤 열까지 출력할지 정했다면 행의 순서를 어떻게 보여줄지 **정렬**해주는 절이 ORDER BY이다.

### LIMIT 절

- LIMIT 절은 결과 중 몇 개의 행을 보여줄지 선택한다.

## SELECT

> SELECT은 가장 많이 사용되는 SQL 문장으로, 테이블에서 정보를 검색하는데 사용된다.

### SELECT 문 예제

```sql
SELECT column_name FROM table_name
SELECT * FROM table_1
SELECT c1, c2 FROM table_2
```

- 일반적으로, 모든 열을 포함한 테이블의 전체 정보가 필요하지 않은 경우 `*`를 사용하지 않는 것이 좋다.
  - 쿼리 서버와 응용 프로그램 간의 트래픽이 증가하여 결과 검색이 느려질 수 있기 때문에 필요한 컬럼만 조회하는 것이 좋다.
  - 모든 열을 포함한 테이블의 전체 정보가 정말 필요한 경우에만 `*(asterisk)`를 사용한다.

### SELECT DISTINCT

```sql
SELECT DISTINCT column FROM table
SELECT DISTINCT(column) FROM table
```

- 테이블에서 중복값을 가진 컬럼이 포함된 경우, 고유/중복되지 않은 값을 나열하고자 하는 상황이 발생할 때 사용할 수 있다.
- `DISTINCT` 키워드를 사용하여 컬럼에서 고유한 값만 반환할 수 있다.
- `DISTINCT`가 적용되는 컬럼을 명확히 하기 위해 괄호를 사용할 수도 있다.

### COUNT

- COUNT 함수는 쿼리의 특정 조건과 일치하는 행의 수를 반환한다.
- 특정 열에 `COUNT`를 적용하거나 `COUNT(*)`을 전달할 수 있으며 결과는 동일하다.

### SELECT WHERE

- `SELECT`과 `WHERE`은 기본적인 SQL 문장에서 자주 사용된다.
- `WHERE` 문은 반환되는 행에 대한 열 조건을 지정할 수 있다.
- `WHERE` 절은 `SELECT`문의 FROM 절 바로 다음에 나타난다.

### ORDER BY

- PostgreSQL이 가끔 쿼리 결과를 다른 순서로 반환할 수 있다.
- `ORDER BY`를 사용하여 컬럼 값에 따라 행을 오름차순 또는 내림차순으로 정렬할 수 있다.
- `ORDER BY`의 기본값은 `ASC`이다.
- 여러 열을 기준으로 정렬을 할 수도있다.
- `ORDER BY`는 쿼리의 끝 부분에 위치시켜서 선택 및 필터링을 먼저 수행한 후 최종적으로 정렬한다.

### LIMIT

- `LIMIT` 명령은 쿼리에서 반환되는 행의 수를 제한하는데 사용한다.
  - 테이블의 모든 행을 반환하지 않고, 상위 몇 개의 행만 보고 테이블 레이아웃을 파악하려는 경우 유용하다.
- `LIMIT`은 쿼리 요청의 맨 끝에 위치하며 마지막으로 실행된다.
  - 이는 `WHERE` 문으로 필터링하고 `ORDER BY`로 정렬하고, 쿼리하려는 모든 조건이나 필터를 적용한 후에 `LIMIT`을 통해 최종적으로 몇 개의 행으로 표시하고 싶은지를 지정한다.

### BETWEEN

- `BETWEEN` 연산자는 값을 값의 범위와 일치시키는데 사용할 수 있다.
  - `value_1 BETWEEN low AND high`와 같이 사용 가능
- `NOT` 논리 연산자와 결합하여 `BETWEEN`을 사용할 수도 있다.
  - `value_1 NOT BETWEEN low AND high`와 같이 사용 가능
- 날짜에 대한 `BETWEEN` 연산자를 사용할 때는 타임스탬프 정보도 포함된 경우
  - `BETWEEN`, `<=`, `>=` 비교 연산자를 사용하는데 주의해야 한다.
  - 이는 날짜/시간이 `0:00`에서 시작하기 때문이다.
- 2023년 3월 정보를 얻기 위해서는 다음과 같이 작성해야 한다.
  ```sql
  SELECT * FROM table_name WHERE created_at BETWEEN '2023-03-01' AND '2023-04-01';
  ```
  - `2023-03-31`로 작성하면 `2023-03-31 00:00:00` 이전의 정보들을 가져오기 때문에 `2023-04-01`로 작성해야 한다.

## 비교 연산자 및 기타

### 연산자 우선 순위

1. 괄호()
2. NOT 연산자
3. 비교 연산자, SQL 연산자
4. AND
5. OR

### 비교 연산자

- `=`, `<>`, `>=`, `>`, `<=`, `<`
- 모든 자료형에 대해 적용
- 문자열의 크기 비굔느 사전 순으로 수행됨
- 예 : '01' < '02' < '1' < '11' < '2'
- NULL에는 비교 연산자 사용 불가
  - 따라서 `IS NULL`, `IS NOT NULL`을 사용함

### 논리 연산자

- 모든 자료형에 대해 적용
- `NOT`, `AND`, `OR` (우선순위: `NOT` > `AND` > `OR`)
- 다음 SQL문 중 실행 결과가 나머지 두 가지와 다른 것은?
  ```sql
  SELECT PLAYER_NAME, POSITION, HEIGHT FROM PLAYER
  WHERE POSITION <> 'GK' AND HEIGHT > 180;

  SELECT PLAYER_NAME, POSITION, HEIGHT FROM PLAYER
  WHERE NOT(POSITION = 'GK') AND HEIGHT > 180;

  SELECT PLAYER_NAME, POSITION, HEIGHT FROM PLAYER
  WHERE NOT(POSITION = 'GK' AND HEIGHT > 180); -- = NOT(POSITION = 'GK') OR NOT(HEIGHT > 180);
  ```

### IN

- 특정 경우에는 여러 가능한 값을 확인해야 할 수도 있다.
  - 예) 사용자 이름이 아려진 이름 목록에 포함되어 있는지 확인하려는 경우
- `IN` 연산자를 사용하여 다중 옵션 목록에 값이 포함되어 있는지 확인하는 조건을 만들 수 있다.
  ```sql
  value_1 IN (option1, option2, ..., option_n)
  ```

### LIKE & ILIKE

- `LIKE` 연산자를 사용하면 와일드카드 문자를 이용하여 문자열 데이터에 대한 패턴 매칭을 수행할 수 있다.
  - `%`: 임의의 문자열을 대체한다.
  - `_`: 하나의 문자를 대체한다.
- `LIKE`는 대소문자를 구분하므로 대소문자를 구분하지 않는 `ILIKE`를 사용할 수 있다.

### LIKE 예시

- `A`로 시작하는 모든 이름
  ```sql
  WHERE name LIKE 'A%'
  ```
- `a`로 끝나는 모든 이름
  ```sql
  WHERE name LIKE '%a'
  ```
- 모든 시리즈의 미션 임파서블 영화 조회
  ```sql
  WHERE title LIKE 'Mission Impossible _'
  ```
- 'Version#A4', 'Version#B7' 등의 형식으로 버전 문자열 코드 조회
  ```sql
  WHERE value_1 LIKE 'Version#__'
  ```
- 패턴 매칭 연산자를 겨랍하여 더 복잡한 패턴을 생성할 수 있다.
  ```sql
  -- Cheryl, Theresa, Sherri
  WHERE name LIKE '_her%';
  ```
- PostgreSQL은 `LIKE` 및 `ILIKE` 외에 전체적인 정규표현식 기능을 지원한다.
  - https://www.postgresql.org/docs/12/functions-matching.html

### ROWNUM

- TOP N개의 레코드 반환
- 사용자가 아닌 시스템이 관리하는 Pseudo Column
  - 채번, 출력 개수 지정 등에 활용 가능
- 한 행만 반환할 때
  ```sql
  SELECT PLAYER_NAME FROM PLAYER WHERE ROWNUM = 1;
  SELECT PLAYER_NAME FROM PLAYER WHERE ROWNUM <= 1;
  SELECT PLAYER_NAME FROM PLAYER WHERE ROWNUM < 2;
  ```
- 여러 행 반환할 때
  ```sql
  SELECT PLAYER_NAME FROM PLAYER WHERE ROWNUM = 3; -- 안됨
  SELECT PLAYER_NAME FROM PLAYER WHERE ROWNUM <= 3;
  SELECT PLAYER_NAME FROM PLAYER WHERE ROWNUM < 4;
  ```
- 테이블 내의 UNIQUE한 값 설정에도 사용 가능
  - ROWNUM을 이용하여 ID 필드 생성
  - UPDATE 테이블명 SET 칼럼명 = ROWNUM;
    ```sql
    ALTER TABLE PLAYER ADD (ROW_ID NUMBER);
    UPDATE PLAYER SET ROW_ID = ROWNUM;

    SELECT PLAYER+NAME FROM PLAYER WHERE ROW_ID = 3;
    ```

#### PostgreSQL에서 ROWNUM 사용하기

```sql
SELECT *
FROM (SELECT ROW_NUMBER() OVER(ORDER BY NAME) AS ROWNUM, name FROM users) T
WHERE ROWNUM % 2 = 0;
```

- 행 순서를 나타내려면 `ROW_NUMBER() OVER(ORDER BY ...)` 구분을 사용하면 되고, 서브쿼리를 통해 해당 값에 조건을 매길 수 있다.
- 위는 짝수번째 행만 조회하는 쿼리이다.


## GROUP BY와 집계 함수

> GROUP BY는 데이터를 분류하여 데이터가 어떤 범주에 어떻게 분포되는지를 이해하기 위해 데이터를 집계하는데 사용된다.

### 집계 함수

- SQL은 다양한 집계 함수를 제공한다.
- 집계 함수의 주요 아이디어는 여러 입력값을 가져와 하나의 출력값을 반환하는 것이다.
 - https://www.postgresql.org/docs/current/functions-aggregate.html
- 가장 일반적인 집계 함수:
  - `AVG()`: 평균값을 반환
  - `COUNT()`: 값의 개수를 반환
  - `MAX()`: 최대값을 반환
  - `MIN()`: 최소값을 반환
  - `SUM()`: 모든 값의 합계를 반환
- 집계 함수 호출은 `SELECT` 절이나 `HAVING` 절에서만 발생한다.
- 주의 사항
  - `AVG()`는 소수점 이하 많은 자릿수의 부동 소수점 값을 반환한다. (ex: 2.342418)
  - `COUNT()`는 단순히 행 수를 반환하는데, 관례적으로 `COUNT(*)`를 사용한다.

### GROUP BY

- `GROUP BY`를 사용하면 카테고리 별로 열을 집계할 수 있다.
- 구문
  ```sql
  SELECT category_col, AGG(data_col) FROM table GROUP BY category_col
  ```
- `GROUP BY` 절은 반드시 `FROM` 또는 `WHERE` 문 바로 뒤에 나와야 한다.
  ```sql
  SELECT category_col, AGG(data_col)
  FROM table
  WHERE category_col != 'A'
  GROUP BY category_col
  ```
- `SELECT` 문에서 열은 집계 함수를 가지거나 `GROUP BY` 호출에 있어야 한다.
  - 실제 `SELECT` 문에서 열에 집계 함수가 적용되어 있고, 다른 열을 불러오려면 `GROUP BY`는 필수!!
    ```sql
    SELECT company, division, SUM(sales)
    FROM finance_table
    GROUP BY company, division
    ```
      - 이 쿼리의 결과는 회사별 부서별 판매액의 총합이 된다.
- GROUP BY로 그룹별 집계함수를 구하는 경우를 제외하고는 WHERE 절로 한 번에 거는 것이 좋다
  ```sql
  SELECT company, division, SUM(sales)
  FROM finance_table
  WHERE division IN ('marketing', 'transport')
  GROUP BY company, division
  ```
- **`WHERE` 문에서 집계 결과를 참조해서는 안된다.** `HAVING`을 사용하여 결과를 필터링해야 한다.
  - 집계를 바탕으로 결과를 분류하려면 전체 함수를 참조해야 한다.
  ```sql
  SELECT compay, SUM(sales)
  FROM finance_table
  GROUP BY company
  ORDER BY SUM(sales)
  ```
  ```sql
  SELECT DATE(payment_date), SUM(amount) FROM payment
  GROUP BY DATE(payment_date)
  ORDER BY SUM(amount) DESC
  ```

### HAVING

- `HAVING`은 집계 함수의 결과를 필터링하는데 사용되며, `WHERE`와 유사하다.
  - `WHERE` 단일행을 필터링하는데 사용되고, `HAVING`은 집계된 결과 집합을 필터링하는데 사용된다.
- `HAVING` 절은 집계가 **수행된 이후**에 자료를 필터링하기 때문에 `GROUP BY` 뒤에 있어야 한다.
  ```sql
  SELECT column1, AGG(column2)
  FROM table_name
  GROUP BY column1
  HAVING condition;
  ```
- 집계된 결과 중 100 이상의 주문 수를 가진 고객 검색
  ```sql
  SELECT customer_id, COUNT(*) as num_orders
  FROM orders
  GROUP BY customer_id
  HAVING COUNT(*) >= 100;
  ```
- 집계 결과는 `WHERE` 절이 실행된 이후 발생하기 때문에 `WHERE`를 사용하여 필터링할 수 없다.
  ```sql
  -- WHERE 필터링을 적용하고 나서 GROUP BY로 호출한 후에 HAVING에 판매액 총액이 1,000달러보다 큰 값을 조건으로 다시 필터링
  SELECT company, SUM(sales)
  FROM finance_table
  WHERE company != 'Google'
  GROUP BY company
  HAVING SUM(sales) > 1000
  ```
  - `SUM(sales)`를 기준으로 필터링하고 싶다면?
    - 집계 함수는 SELECT 문의 열 목록에 명시되지만, 그룹화된 데이터에 적용된다.
    - 즉, GROUP BY 절에 지정된 열에 따라 결과가 그룹화되고, 집계 함수는 GROUP BY 절에 지정된 각 그룹별로 계산된다.
    - 따라서 `GROUP BY` 절 이후에 실행되기 때문에 `WHERE` 절에서 접근할 수 없다.

## JOINS

> JOIN은 여러개의 테이블을 결합할 수 있도록 해주는 기능이다.    

### INNER JOIN

```
  -----    -----
/      / X \     \
|  A   | X |  B   |
\      \ X /     /
  -----    -----
```

- `INNER JOIN`은 두 테이블을 모두 충족하는 레코드 세트를 결과로 출력한다.
- 구문
  ```sql
  SELECT * FROM Table_A
  INNER JOIN Table_B
  ON Table_A.col_match = Table_B.col_match

  SELECT * FROM Table_B
  INNER JOIN Table_A
  ON Table_A.col_match = Table_B.col_match
  
  SELECT *
  FROM Table_A, Table_B
  WHERE Table_A.col_match = Table_B.col_match
  ```
  - `INNER JOIN`에서는 테이블의 순서가 중요하지 않다.
  - 또한, `JOIN`에 `INNER` 키워드가 없는 경우 PostgreSQL은 `INNER JOIN`으로 처리한다.

### OUTER JOIN

- `OUTER JOIN`에 있는 유형은 결합되는 테이블 중 하나에만존재하는 값들을 어떻게 처리할 지 지정할 수 있게 해준다.
  - `INNER JOIN`에 비해 좀 더 복잡한 `JOIN` 유형이다.
- `OUTER JOIN`의 유형
  - `FULL OUTER JOIN`
  - `LEFT OUTER JOIN`
  - `RIGHT OUTER JOIN`

### FULL OUTER JOIN

```
  ------     ------
/ X X X / X \ X X X \
| X A X | X | X B X |
\ X X X \ X / X X X /
  ------     ------
```
```sql
-- 벤다이어그램이 대칭이기 때문에 테이블 순서를 바꿔도 같은 결과가 나온다.
SELECT * FROM Table_A
FULL OUTER JOIN Table_B
ON Table_A.col_match = Table_B.col_match

SELECT * FROM Table_B
FULL OUTER JION Table_A
ON Table_A.col_match = Table_B.col_match
```

#### FULL OUTER JION Example

```sql
SELECT * FROM registrations FULL OUTER JOIN logins
ON registrations.name = logins.name
```

| REGISTRATIONS |         |
| ------------- | ------- |
| reg_id        | name    |
| 1             | Andrew  |
| 2             | Bob     |
| 3             | Charlie |
| 4             | Davie   |

| LOGINS |         |
| ------ | ------- |
| 1      | Xavier  |
| 2      | Andrew  |
| 3      | Yolanda |
| 4      | Bob     |

| RESULTS |         |        |         |
| ------- | ------- | ------ | ------- |
| red_id  | name    | log_id | name    |
| 1       | Andrew  | 2      | Andrew  |
| 2       | Bob     | 4      | Bob     |
| 3       | Charlie | null   | null    |
| 4       | David   | null   | null    |
| null    | null    | 1      | Xavier  |
| null    | null    | 3      | Yolanda |

#### FULL OUTER JOIN with WHERE

```
  ------     ------
/ X X X /   \ X X X \
| X A X |   | X B X |
\ X X X \   / X X X /
  ------     ------
```

- 두 테이블 모두에 고유한 행을 가져올 수 있다.(두 테이블 모두에서 찾을 수 없는 행)
  ```sql
  SELECT * FROM Table_A
  FULL OUTER JOIN Table_B
  ON Table_A.col_match = Table_B.col_match
  WHERE Table_A.is IS NULL OR Table
  ```
  - 기본적으로 `INNER JOIN`과 정반대되는 개념이다.
- Example
  ```sql
  SELECT * FROM registrations
  FULL OUTER JOIN logins
  ON registrations.name = logins.name
  WHERE registrations.reg_id IS NULL OR logins.log_id IS NULL
  ```

  | RESULTS |         |        |         |
  | ------- | ------- | ------ | ------- |
  | red_id  | name    | log_id | name    |
  | 3       | Charlie | null   | null    |
  | 4       | David   | null   | null    |
  | null    | null    | 1      | Xavier  |
  | null    | null    | 3      | Yolanda |

### LEFT OUTER JOIN

```
  ------     ------
/ X X X / X \       \
| X A X | X |   B   |
\ X X X \ X /       /
  ------     ------
```

- `LEFT OUTER JOIN`은 왼쪽 테이블에 있는 레코드 집합을 생성한다.
  - 오른쪽 테이블과 일치하지 않는 경우, 결과는 null 이다.
  ```sql
  SELECT * FROM Table_A
  LEFT OUTER JOIN Table_B
  ON Table_A.col_match = Table_B.col_match
  ```
  - 테이블 A에서 정보를 가져올 것인데 이 정보 중에서 테이블 A에만 해당되는 것도 있고, 테이블 B에 있는 것 중 테이블 A와 겹치는 부분만 가져온다.

#### LEFT OUTER JOIN Example

```sql
SELECT * FROM registrations
LEFT OUTER JOIN logins
ON registrations.name = logins.name
```

| RESULTS |         |        |        |
| ------- | ------- | ------ | ------ |
| red_id  | name    | log_id | name   |
| 1       | Andrew  | 2      | Andrew |
| 2       | Bob     | 4      | Bob    |
| 3       | Charlie | null   | null   |
| 4       | David   | null   | null   |

#### LEFT OUTER JOIN with WHERE

```
  ------     ------
/ X X X /   \       \
| X A X |   |   B   |
\ X X X \   /       /
  ------     ------
```
```sql
SELECT * FROM Table_A
LEFT OUTER JOIN Table_B
ON Table_A.col_match = Table_B.col_match
WHERE Table_B.id IS NULL
```

- 만약 테이블 A에만 고유한 레코드를 원하는 경우에 위의 쿼리 예시처럼 명령하면 왼쪽 테이블에 고유한 행만 가져올 수 있다.

```sql
SELECT * FROM registrations
LEFT OUTER JOIN logins
ON registrations.name = logins.name
WHERE logins.log_id IS NULL
```

| RESULTS |         |        |      |
| ------- | ------- | ------ | ---- |
| red_id  | name    | log_id | name |
| 3       | Charlie | null   | null |
| 4       | David   | null   | null |

### RIGHT JOIN

```
  ------     ------
/       / X \ X X X \
|   A   | X | X B X |
\       \ X / X X X /
  ------     ------
```
```sql
SELECT * FROM Table_A
RIGHT OUTER JOIN Table_B
ON Table_A.col_match = Table_B.col_match
```

- `RIGHT OUTER JOIN`은 테이블이 전환된다는 점을 제외하고 `LEFT OUTER JOIN`과 동작방식이 동일하다.
  - 이는 `LEFT OUTER JOIN`에서 테이블 순서를 전환하는 것과 동일하다.

#### RIGHT OUTER JOIN with WHERE

```
  ------     ------
/       /   \ X X X \
|   A   |   | X B X |
\       \   / X X X /
  ------     ------
```
```sql
SELECT * FROM Table_A
RIGHT OUTER JOIN Table_B
ON Table_A.col_match = Table_B.col_match
WHERE Table_A.is IS NULL
```

## 집합 연산자

- 여러 질의(Select 문) 결과를 하나로 결합하기 위해 사용
- 집합 연산의 대상이 되는 두 질의는...
  - SELECt 절의 컬럼 수가 동일해야 하고,
  - SELECT 절의 동일 위치에 존재하는 칼럼의 데이터 타입이 상호 호환 가능해야 한다.

### 집합 연산자 종류

|집합 연산자|연산자 의미|
|---|---|
|UNION|여러 SQL문의 결과에 대한 합집합(중복된 행은 제거한 후 하나의 행만 출력)|
|UNION ALL|여러 SQL문의 결과에 대한 합집합(중복된 행도 삭제하지 않고 모두 출력 -> 속도가 빠르므로 우선 고려)|
|INTERSECT|여러 SQL문의 결과에 대한 교집합(중복된 행은 제거한 후 하나의 행만 출력)|
|EXCEPT|앞의 SQL문의 결과에서 뒤의 SQL문의 결과를 뺀 차집합(중복된 행은 제거한 후 하나의 행만 출력)|

### UNION

```
SELECT column_name(s) FROM Table1
UNION
SELECT column_name(s) FROM Table2
```

- `UNION` 연산자를 사용하면 2개 이상의 SELECT 문의 결과 세트를 결합할 수 있다.
- `JOIN`과 `UNION`의 차이는 `UNION`은 두 결과를 직접 붙인 것이다.
- 두 `SELECT` 문의 결과를 서로의 바로 위에 붙여준다.

### INTERSECT

```sql
SELECT team_id, player_name, postion
FROM player
WHERE team_id = 'K06'
INTERSECT
SELECT team_id, player_name, postion
FROM player
WHERE position = 'GK';

-- INTERSECT 연산자는 IN 서브쿼리, EXISTS 서브쿼리로도 표현 가능하다.
SELECT team_id, player_name, postion
FROM player
WHERE team_id = 'K06' AND position = 'GK';
```

### EXCEPT

```sql
SELECT team_id, player_name, postion
FROM player
WHERE team_id = 'K06'
EXCEPT
SELECT team_id, player_name, postion
FROM player
WHERE postion = 'MF';

SELECT team_id, player_name, postion
FROM player
WHERE team_id = 'K06' AND position <> 'MF';

SELECT team_id, player_name, postion
FROM player
WHERE team_id = 'K06'
AND postion NOT IN (SELECT position FROM player WHERE postion = 'MF');
```

## Timestamps and Extract

- 테이블과 데이터베이스를 설계하고 시간 데이터 유형을 선택할 때는 신중하게 고려해야 한다.
  - 상황에 따라 전체 `TIMESTAMPTZ` 레벨이 필요하거나 필요하지 않을 수 있다.
- 타임존 확인
  - `show timezone;`
- 타임존 설정(변경)
  - `set timezone = 'America/Los_Angeles';`

### Date and Time 정보와 관련된 데이터 타입
  - `TIME`: 시간만 포함
    - 15:03:30.484561
  - `DATE`: 날짜만 포함
    - 2023-04-24
  - `TIMESTAMP`: 타임존을 명시하지 않은 날짜와 시간
    - 2023-04-24 15:03:30.484561
  - `TIMESTAMPTZ`: 타임존을 명시한 날짜와 시간
    - 2023-04-24 18:01:40.920477+09
    - 타임존이 변경되면 타임존에 따라 자동으로 계산된다.
    - 저장할 때는 UTC0 기준 timestamp로 변환되어 저장되고, 출력할 일이 있을 때 pg의 timezone 혹은 설정한 존으로 변환해 출력하게 된다.
    - 만약 위에 시차가 17시간 차이나는 로스엔젤레스로 타임존을 변경하게 되면 `2023-04-24 02:01:40.920477-07`(서머타임 적용, 원래는 -8)로 변경된다.
    - `timestamp` 타임존에 따른 시간 변경이 적용되지 않는다.

### 날짜 및 시간 데이터 유형과 관련된 함수
  - `NOW`
    - 현재시간을 타임존을 포함하여 반환한다.
    - 2023-04-24 18:25:24.08764+09
  - `TIMEOFDAY`
    - 해당 함수는 현재 일자(타임존 포함)를 text형으로 반환한다.
    - Mon Apr 24 18:25:10.385280 2023 KST
  - `CURRENT_TIME`
    - 현재 시간을 반환한다.
    - 18:26:27.001682+09
  - `CURRENT_DATE`
    - 현재 날짜를 반환한다.
    - 2023-04-24

### 시간 데이터 유형을 사용하여 정보 추출

#### EXTRACT
  
- `EXTRACT()` 함수는 날짜/시간 데이터에서 year(년도), month(월), day(일)과 같은 요소를 추출/검색하는 함수이다.
- `EXTRACT(field FROM source)`
  - field는 year, month, day 등의 날짜/시간 데이터 요소를 말한다.
  - source는 실제 timestamp 값을 의미한다. ex) '2023-04-25 15:00:00'
- field
  - CENTURY, DAY, DOW, DOY, EPOCH, HOUR, MILLISECOND, MINUTE, MONTH, QUARTER, SECOND, WEEK, YEAR

```sql
SELECT NOW(); -- 2023-04-26 13:46:12.170282+09 <- 수요일
select extract('DOW' FROM NOW()); -- 3
select extract('CENTURY' FROM now()); -- 21

select 
	EXTRACT('CENTURY' FROM NOW()) AS century,
	EXTRACT('DAY' FROM NOW()) AS day,
	EXTRACT('DOW' FROM NOW()) AS dow,
	EXTRACT('DOY' FROM NOW()) AS doy,
	EXTRACT('EPOCH' FROM NOW()) AS epoch,
	EXTRACT('HOUR' FROM NOW()) AS hour,
	EXTRACT('MILLISECOND' FROM NOW()) AS millisecond,
	EXTRACT('MINUTE' FROM NOW()) AS minute,
	EXTRACT('MONTH' FROM NOW()) AS month,
	EXTRACT('QUARTER' FROM NOW()) AS quarter,
	EXTRACT('SECOND' FROM NOW()) AS second,
	EXTRACT('WEEK' FROM NOW()) AS week,
	EXTRACT('YEAR' FROM NOW()) AS year;
```

#### DATE_TRUNC

- `DATE_TRUNC()` 함수는 인수를 사용하여 다양한 날짜, 시간 형식으로 데이터를 자를때 사용한다.
- `DATE_TRUNC()` 함수에서 사용할 수 있는 인수 목록
  - millennium
  - century
  - decade
  - year
  - quarter
  - month
  - week
  - day
  - hour
  - minute
  - second
  - milliseconds
  - microseconds
- 예시
  ```sql
  SELECT 
    DATE_TRUNC('year', NOW()) as year, -- 년 밑으로 자르기
    DATE_TRUNC('month', NOW()) as month, -- 월 밑으로 자르기
    DATE_TRUNC('day', NOW()) as day, -- 일 밑으로 자르기
    DATE_TRUNC('hour', NOW()) as hour, -- 시 밑으로 자르기
    DATE_TRUNC('minute', NOW()) as minute, -- 분 밑으로 자르기
    DATE_TRUNC('second', NOW()) as second; -- 초 밑으로 자르기

  SELECT TO_CHAR(DATE_TRUNC('day', NOW()), 'YYYY.MM.DD.HH24.MI.SS'); -- 2023.05.15.00.00.00
  ```

![image](https://github.com/marchidx04/db-practice/assets/126429401/e5227107-2ec0-413f-b5c6-fbb3e7fdf0c9)


#### AGE

- `AGE()`함수는 특정날짜를 지정하면 해당 날짜에서의 나이를 출력한다.
- `AGE(data_col)`

#### TO_CHAR

- 데이터 유형을 텍스트로 변환하는 일반 기능이다.
- 형식 지정을 위한 타임 스탬프에 유용하다.
  - ex) `TO_CHAR(data_col, 'mm-dd) 
- https://www.postgresql.org/docs/15/functions-formatting.html

### String 함수

```sql
SELECT UPPER(first_name) || ' ' || UPPER(last_name) AS name FROM customer;
```

## 서브쿼리(Sub Query)

> 서브쿼리는 알려지지 않은 기준을 이용한 검색을 위해 사용한다.

- 서브쿼리를 이용하면 다른 쿼리의 결과에 대한 메인쿼리의 일부로 수행하여 복잡한 쿼리를 구성할 수 있다.
- 구문은 간단하며 두 개의 `SELECT` 문을 포함한다.
- 좋은 자료: https://dataonair.or.kr/db-tech-reference/d-guide/sql/?mod=document&uid=349

### SQL문에서 서브쿼리를 사용할 수 있는 절

- SELECT
- FROM
- WHERE
- HAVING
- ORDER BY
- INSERT 문의 VALUES
- UPDATE 문의 SET

### 조인과 서브쿼리 차이점

- 조인은 조인에 참여하는 모든 테이블이 대등한 관계에 있기 때문에 조인에 참여하는 모든 테이블의 칼럼을 어느 위치에서라도 자유롭게 사용할 수 있다.
  - 조직(1)과 사원(M) 테이블을 조인하면 결과는 사원 레벨(M)의 집합이 생성된다.
  - M:N 관계의 테이블을 조인하면 MN(= M * N) 레벨의 집합이 결과로서 생성된다.
  - 1:1 관계의 테이블을 조인하면 1(= 1 * 1) 레벨의 집합이 생성된다.
  - 서브쿼리 레벨과는 상관없이 항상 메인쿼리 레벨로 결과 집합이 생성된다.
- 서브쿼리는 메인쿼리의 칼럼을 모두 사용할 수 있지만 메인쿼리는 서브쿼리의 칼럼을 사용할 수 없다.
  - 따라서 질의 결과에 서브쿼리 칼럼을 표시해야 한다면 조인 방식으로 변환하거나 함수, 스칼라 서브쿼리(Scalar Subquery) 등을 사용해야 한다.
  - 서브쿼리는 서브쿼리 레벨과는 상관없이 항상 메인쿼리 레벨로 결과 집합이 생성된다.
- SQL문에서 서브쿼리 방식을 사용해야 할 때 잘못 판단하여 조인 방식을 사용하는 경우가 있다.
  - 결과는 조직 레벨(1)이고 사원 테이블(M)에서 체크해야 할 조건이 존재하는 경우
  - 이런 상황에서 SQL문을 작성할 때 조인을 사용한다면 결과 집합은 사원(M) 레벨이 될 것이다.
    - 이렇게 되면 원하는 결과가 아니기 때문에 SQL문에 DISTINCT를 추가해서 결과를 다시 조직(1) 레벨로 만들어야 한다.
  - 따라서 조인 방식이 아니라 서브쿼리 방식을 사용해야 한다.
    - 메인쿼리로 조직을 사용하고 서브쿼리로 사원 테이블을 사용하면 결과 집합은 조직 레벨이 되어 원하는 결과를 얻을 수 있다.

### 서브쿼리 사용 시 주의사항

1. 서브쿼리를 괄호로 감싸서 사용한다.
2. 서브쿼리는 단일 행(Single Row) 또는 복수 행(Multiple Row) 비교 연산자와 함께 사용 가능하다.
    - 단일 행 비교 연산자는 서브쿼리의 결과가 반드시 1건 이하이어야 한다.
    - 복수 행 비교 연산자는 서브쿼리의 결과 건수와 상관 없다.
3. 서브쿼리에서 `ORDER BY`를 사용하지 못한다.
    - `ORDER BY` 절은 `SELECT` 절에서 오직 한 개만 올 수 있기 때문에 `ORDER BY`절은 메인쿼리의 마지막 문장에 위치해야 한다.

### 서브쿼리 분류

#### 단일 행(Single Row) 서브쿼리

- 서브쿼리의 실행 결과가 **반드시 1건 이하**인 서브쿼리를 의미한다.
- 단일 행 서브쿼리는 단일 행 비교 연산자와 함께 사용 가능하다.
- 단일 행 비교 연산자: `=`, `<`, `<=`, `>`, `>=`, `<>`

```sql
-- 평균 등수보다 높은 학생들 조회
SELECT student, grade
FROM test_scores
WHERE grade > (SELECT AVG(grade) FROM test_scores);
```

#### 다중 행(Multi Row) 서브쿼리

- 서브쿼리의 실행 결과가 여러 건인 서브쿼리를 의미한다.
- 다중 행 서브쿼리는 다중 행 비교 연산자와 함께 사용된다.
- 다중 행 비교 연산자: `IN`, `ALL`, `ANY`, `SOME`, `EXISTS`
  - `IN (서브쿼리)`: 서브쿼리의 결과에 존재하는 임의의 값과 동일한 조건을 의미한다.
  - `비교연산자 ALL (서브쿼리)`: 서브쿼리의 결과에 존재하는 모든 값을 만족하는 조건을 의미한다.
    - 비교연산자로 `>`를 사용했다면 메인쿼리는 서브쿼리의 모든 결과값을 만족해야 하므로, 서브쿼리 결과의 최대값보다 큰 모든 건이 조건을 만족한다.
  - `비교연산자 ANY (서브쿼리)`: 서브쿼리의 결과에 존재하는 어느 하나의 값이라도 만족하는 조건을 의미한다.
    - 비교연산자로 `>`를 사용했다면 메인쿼리는 서브쿼리의 값들 중 어떤 값이라도 만족하면 되므로,
    - 서브쿼리의 결과의 최솟값보다 큰 모든 건이 조건을 만족한다.
  - `EXISTS`: 서브쿼리의 결과를 만족하는 값이 존재하는지 여부를 확인하는 조건을 의미한다.
    - 여러 건이더라도 1건만 찾으면 더 이상 검색하지 않고 메인쿼리의 결과를 반환한다.
```sql
-- rental이 2023-01-03일자인 영화 제목 조회
SELECT film_id, title -- 영화 제목 조회
FROM film
WHERE film_id IN
(SELECT inventory.film_id -- 렌탈일자가 2023-01-03인 영화 id들 조회
FROM rental
INNER JOIN inventory ON inventory.inventory_id = rantal.inventory_id
WHERE rental_date BETWEEN '2023-01-03' AND '2023-01-04')
```

- `EXISTS` 연산자는 서브쿼리에 행이 있는지 테스트하는데 사용된다.
  - 일반적으로 서브쿼리는 `EXISTS` 함수에서 전달되어 서브쿼리와 함께 반환되는 행이 있는지 확인한다.

#### 다중 컬럼(Multi Column) 서브쿼리

```sql
SELECT team_id, player_name, position, back_no, height
FROM player
WHERE (team_id, height) IN (SELECT team_id, MIN(height)
							FROM   player
                            GROUP BY team_id)
ORDER BY team_id, player_name;
```

- 서브쿼리의 실행 결과로여러 칼럼을 반환한다.
- 메인쿼리의 조건절에 여러 칼럼을 동시에 비교할 수 있다.
- 서브쿼리와 메인쿼리에서 비교하고자 하는 칼럼 개수와 칼럼의 위치가 동일해야 한다.
- 위 쿼리는 소속팀 별로 키가 가장 작은 사람들의 정보를 출력하는 쿼리이지만 꼭 팀별로 한 명씩 출력되는 것이 아니라 팀 별로 MIN(height)를 만족하는 모든 사람들이 출력되는 점을 주의해야 한다.

### 기타 서브쿼리

#### 스칼라 서브쿼리

- SELECT 절에서 사용하는 서브쿼리이다.
- scalar는 '한 번에 한 가지만 처리하는'이라는 뜻을 가지고 있듯이 스칼라 서브쿼리에 의해 나오는 결과는 **하나의 행**이어야 한다.

```sql
-- 고객마다 총구매액 조회
SELECT id, (SELECT SUM(cost_amount) AS avgperid
			FROM orders o
			WHERE o.customer_id = c.id)
FROM customers c;

SELECT c.id, SUM(cost_amount) as avgperid
FROM customers c, orders o
WHERE c.id = o.customer_id
GROUP BY c.id;
```
- 만약 해당 스칼라 서브쿼리가 2개 이상의 레코드를 조회한다면 에러가 발생한다.

#### FROM 절에서 서브쿼리

- FROM 절에서 사용되는 서브쿼리를 인라인 뷰(Inline View)라고 한다.
  - 서브쿼리의 결과가 실행 시 동적으로 생성된 테이블인 것처럼 사용할 수 있다.
  - 인라인 뷰는 SQL문이 실행될 때만 임시적으로 생성되는 동적인 뷰이기 때문에 데이터베이스에 해당 정보가 저장되지 않는다.
    - 그래서 일반적인 뷰를 정적 뷰(Static View)라 하고, 인라인 뷰를 동적 뷰(Dynamic View)라고도 한다.
  - 인라인 뷰는 테이블 명이 올 수 있는 곳에 사용할 수 있고 이를 사용하는 것은 조인 방식을 사용하는 것과 같다.
    - 따라서 인라인 뷰의 칼럼은 SQL 문에서 자유롭게 참조할 수 있다.

```sql
-- 양파를 구매한 고객들을 서브쿼리를 이용해 조회
SELECT c.name, o.bought_at
FROM (SELECT customer_id, created_at as bought_at
FROM orders
WHERE product_name = '양파') o, customers c
WHERE o.customer_id = c.id;
```

## 데이터베이스 및 테이블 생성

- 데이터 타입 관련 공식문서 자료: https://www.postgresql.org/docs/current/datatype.html

### 제약 조건(Constraints)

- `Constraints`는 테이블의 데이터 컬럼에 적용되는 규칙
  - 잘못된 데이터가 데이터베이스에 입력되는 것을 방지하여 데이터의 정확성과 신뢰성을 보장할 수 있다.
- 제약 조건은 주로 두 가지로 나뉜다.
  - `열 제약 조건(Column Constraints)`
    - 열의 데이터가 특정 조건을 준수하도록 제한하는 조건
  - `테이블 제약 조건(Table Constraints)`
    - 개별 컬럼이 아닌 전체 테이블에 적용되는 조건

#### 가장 일반적으로 사용되는 제약 조건

- `NOT NULL` 제약 조건
  - 해당 컬럼을 `NULL` 값을 가질 수 없도록 제한
- `UNIQUE` 제약 조건
  - 해당 컬럼에서의 값이 유일하도록 제한
- `PRIMARY KEY`
  - 데이터베이스 테이블에서 각 행(레코드)를 고유하게 식별할 수 있는 키
- `FOREIGN KEY`
  - 해당 컬럼을 다른 테이블의 PK를 참조하도록 제한
- `CHECK` 제약 조건
  - 행의 모든 값이 특정한 조건을 만족하도록 한다.
    ```sql
    CREATE TABLE products (
      product_no integer PRIMARY KEY, -- 오직 정수만 저장
      name text,
      price numeric CHECK (price > 0) -- 정수 또는 소수값 저장
    )
    ```
- `EXCLUSION` 제약 조건
  - 특정 오퍼레이터를 사용한 특정 열이나 식에서 어떤 두 열이 비교될 때 모든 비교 값이 참으로 판명되지 않아야 한다는 조건
    ```sql
    CREATE TABLE company (
      ID INT PRIMARY KEY NOT NULL,
      NAME TEXT,
      AGE INT,
      ADDRESS CHAR(50),
      SALARY numeric,
      EXCLUDE USING gist
      (NAME WITH =,
      AGE WITH <>)
    )
    ```
    ```sql
    INSERT INTO COMPANY7 VALUES(1, 'Paul', 32, 'California', 20000.00 );
    INSERT INTO COMPANY7 VALUES(2, 'Paul', 32, 'Texas', 20000.00 );
    INSERT INTO COMPANY7 VALUES(3, 'Paul', 42, 'California', 20000.00 );
    ```
    ```sql
    ERROR:  conflicting key value violates exclusion constraint "company7_name_age_excl"
    DETAIL:  Key (name, age)=(Paul, 42) conflicts with existing key (name, age)=(Paul, 32).
    ```
### CASCADE

- 참조 테이블(부모 테이블)의 해당 행(row)이 삭제되거나 업데이트될 떄, 자동적으로 기본 테이블(자식 테이블)에서도 매치되는 행이 똑같이 삭제 또는 업데이트됨
- 공식 문서는 `CASCADE`를 FOREIN KEY CONSTRAINTS에서 옵션으로 사용할 수 있는 [Referential Actions](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html#foreign-key-referential-actions)라고 설명함

#### ON DELETE CASCADE 설정

- `ON DELETE CASCADE` 옵션을 적용하면 부모 테이블에서 row를 삭제할 경우 연결된 자식 테이블의 row가 함께 삭제된다.
  - 연결된 데이터를 한 번에 지울 수 있어 데이터의 관리가 편리하고 일관성을 유지할 수 있다는 장점이 있다.
- 마찬가지로 `ON UPDATE CASCADE`를 설정하면 UPDATE할 때 `CASCADE` 옵션이 적용된다.
  - 근데 PK 같은 경우는 바뀌면 안되기 때문에(UPDATE가 되면 안됨) 쓸 일이 없다고 봐도 무방하다.

### CREATE

#### Syntax

```sql
CREATE TABLE table_name (
  column_name TYPE column_constraints,
  column_name TYPE column_constraints,
  table_constraints table_constraint
) INHERITS existing_table_name; 
```

#### Example

```sql
CREATE TABLE account(
	user_id SERIAL PRIMARY KEY,
	username VARCHAR(50) UNIQUE NOT NULL,
	password VARCHAR(50) NOT NULL,
	email VARCHAR(250) UNIQUE NOT NULL,
	created_on TIMESTAMP NOT NULL,
	last_login TIMESTAMP
)

CREATE TABLE job(
	job_id SERIAL PRIMARY KEY,
	job_name VARCHAR(200) UNIQUE NOT NULL
)

CREATE TABLE account_job(
	user_id INTEGER REFERENCES account(user_id),
	job_id INTEGER REFERENCES job(job_id),
	hire_date TIMESTAMP
)
```

- `SERIAL`
  - PostgreSQL에서 sequence는 정수 시퀀스를 생성하는 특수한 종료의 데이터베이스 개체이다.
  - 스퀀스는 종종 테이블의 PK 컬럼에 이용된다.
  - 시퀀스가 생성되고 시퀀스에서 생성된 다음 값(next value)를 해당 컬럼의 기본값으로 설정한다.
  - 삽입(INSERT) 시 자동으로 고유한 정수 항목을 기록하므로 PK에 적합하다.
  
### INSERT

- `INSERT`는 테이블에 레코드를 추가할 수 있도록 수행한다.
- 문자 또는 날짜 값의 경우 작은 따옴표로 묶음
  - 숫자 데이터는 작은 따옴표 없이 사용

#### Syntax

```sql
-- 일부 칼럼에 대응되는 값만 입력
INSERT INTO table (column1, column2, ...) -- COLUMN_LIST
VALUES (value1, value2), (value1, value2), ...; -- VALUE_LIST

-- 전체 칼럼에 대응되는 값을 모두 입력
INSERT INTO table VALUES (전체 COLUMN의 VALUE_LIST);
```

#### Example

```sql
INSERT INTO account(username, password, email, created_on)
VALUES ('Jaewon', 'password', 'jaewon@mail.com', CURRENT_TIMESTAMP);

INSERT INTO job(job_name) VALUES ('Web Programmer');

INSERT INTO account_job(user_id, job_id, hire_date)
VALUES (1, 1, CURRENT_TIMESTAMP);

INSERT INTO 
    links (url, name)
VALUES
    ('https://www.google.com','Google'),
    ('https://www.yahoo.com','Yahoo'),
    ('https://www.bing.com','Bing');
```

### UPDATE

- `UPDATE`는 테이블에 있는 컬럼의 값을 수정, 변경할 수 있도록 수행한다.

#### Syntax

```sql
UPDATE table
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

#### Example

```sql
UPDATE account
SET last_login = CURRENT_TIMESTAMP
WHERE last_login IS NULL;
```

#### UPDATE JOIN

```sql
-- 다른 테이블의 값을 사용하여 업데이트할 수 있다.
UPDATE Table_A
SET original_col = Table_B.new_col
FROM Table_B
WHERE Table_A.id = Table_B.id

UPDATE account_job
SET hire_date = account.created_on
FROM account
WHERE account_job.user_id = account.user_id;
```

#### 결과값에 영향이 있는 값을 불러오기

```sql
UPDATE account
SET last_login = created_on
RETURNING account_id, last_login
```

- `UPDATE` 명령을 사용할 때, 결과가 보여지지 않지만 영향을 받은 특정 세로단을 확인하고 싶은 경우 명령할 수 있다.

### DELETE

`DELETE`은 테이블의 레코드를 삭제할 수 있도록 해준다.

```sql
DELETE FROM table
WHERE row_id = 1;
```

### ALTER

- `ALTER`는 현재 테이블 구조를 변경할 수 있도록 해준다.
  - 컬럼을 추가, 삭제 및 이름을 변경할 수 있다.
  - 컬럼의 데이터 타입을 변경할 수 있다.
  - 컬럼의 `CHECK` 제약 조건을 추가할 수 있다.
  - 컬럼에 대한 `DEFAULT`값을 설정할 수 있다.
  - 테이블의 이름을 변경할 수 있다.

#### Syntax

```sql
ALTER TABLE table_name action
```

#### Renaming Table

```sql
ALTER TABLE information RENAME TO new_info
```

#### Adding Columns

```sql
ALTER TABLE table_name
ADD COLUMN new_col TYPE
```

#### Removing Columns

```sql
ALTER TABLE table_name
DROP COLUMN col_name
```

#### Alter constraints

```sql
ALTER TABLE table_name
ALTER COLUMN col_name
SET DEFAULT value -- SET NO NULL, DROP DEFAULT, DROP NOT NULL, ADD CONSTRAINT constraint_name
```

### DROP

- `DROP`은 테이블의 컬럼을 삭제할 수 있도록 해준다.
- PostgreSQL에서는 자동적으로 해당 컬럼과 관련된 인덱스 및 제약조건들을 삭제해준다.
- 그러나 추가 `CASCADE` 절이 없으면 VIEWS, TRIGGERS, 또는 Stored Procedures에 사용되는 열은 삭제되지 않는다.

#### Syntax

```sql
ALTER TABLE table_name
DROP COLUMN col_name
```

#### 모든 종속 관계까지 제거(Remove all dependencies)

```sql
ALTER TABLE table_name
DROP COLUMN col_name CASCADE
```

#### 해당 컬럼 체크 후 삭제

```sql
ALTER TABLE table_name
DROP COLUMN IF EXISTS col_name -- 해당 컬럼 없으면 에러가 발생하기 때문에 에러 피하기 위해 작성
```

#### 여러 컬럼 삭제

```sql
ALTER TABLE table_name
DROP COLUMN col_one,
DROP COLUMN col_two
```

## 조건식과 프로시저(Conditional Expressions and Operators)

### CASE

- `CASE`는 특정 조건을 만날 때 SQL code를 실행시키기 위해 사용한다.
- 다른 프로그래밍 언어들의 `IF/ELSE`와 비슷하다

#### Searched Case Expression

```sql
CASE
  WHEN condition1 THEN result1
  WHEN condition2 THEN result2
  ELSE some_other_result
END

SELECT a,                             ---------------------
CASE WHEN a = 1 THEN 'one'           |    a     |   label  |
     WHEN a = 2 THEN 'two'           |---------------------|
ELSE 'other' AS label                |    1     |    one   |
END                                  |    2     |    two   |
FROM test;                            ---------------------
```

- 다양한 조건 사용 가능

#### Simple Case Expression

```sql
CASE expression
  WHEN value1 THEN result1
  WHEN value2 THEN result2
  ELSE some_other_result
END

SELECT a,
  CASE a WHEN 1 THEN 'one'
         WHEN 2 THEN 'two'
         ELSE 'other'
  END
FROM test;
```

- 동등(=) 비교에만 사용 가능

#### CASE Example

```sql
SELECT customer_id,
CASE
      WHEN (customer_id <= 100) THEN 'Preminum'
      WHEN (customer_id BETWEEN 101 and 200) THEN 'Plus'
      ELSE 'Normal'
END AS customer_class
FROM customer;

SELECT customer_id,
CASE customer_id
        WHEN 2 THEN 'Winner'
        WHEN 5 THEN 'Second Place'
        ELSE 'Normal'
END AS customer_class
FROM customer;

SELECT
SUM(CASE rental_rate
          WHEN 0.99 THEN 1
          ELSE 0
END) AS number_of_bargains,
SUM(CASE rental_rate
          WHEN 2.99 THEN 1
          WHEN 0
END) AS regular,
SUM(CASE rental_rate
          WHEN 4.99 THEN 1
          ELSE 0
END) AS preminum
FROM film; -- 각 개수 구하는데에도 CASE가 사용될 수 있다.
```

### COALESCE

- `COALESCE` 함수는 `NULL`이 아닌 첫 번쨰 인수를 반환한다.
  - `COALESCE` 함수는 인수에 제한이 없다. 
  - 모든 인수가 `NULL`이면 `COALESCE` 함수도 `NULL`을 반환한다.
- `COALESCE (arg_1, arg_2, ..., arg_n)`
  - `SELECT COALESCE (1, 2)` --> 1
  - `SELECT COALESCE (NULL, 2, 3)` --> 2

#### Example

```sql
SELECT item, (price - COALESCE(discount, 0)) AS final FROM table;

-- CASE 구문으로 표현 가능
SELECT item, (price - 
        CASE WHEN discount IS NOT NULL THEN discount
             ELSE 0
        END) AS final
FROM table;
```

- `COALESCE` 함수는 `NULL`을 포함하는 테이블에 대해 질의문을 작성할 때 유용하다.
  - 만약 해당 값이 `NULL`이면 다른 값으로 대체하도록 만들 수 있다.

### CAST

- `CAST`는 다른 타입으로 변환하기 위해 사용한다.
- `CAST` 함수 문법
  - `SELECT CAST('5' AS INTEGER)`
- PostgreSQL CAST 연산자
  - `SELECT '5'::INTEGER`

#### Example

```sql
-- inventory_id -> integer
SELECT CHAR_LENGTH(CAST(inventory_id AS VARCHAR)) FROM rental;

-- ERROR: function char_length(integer) does not exist
SELECT CHAR_LENGTH(inventory_id) FROM rantal;
```

### NULLIF

- `NULLIF` 함수는 2개의 인자를 받고 두 인자가 같은 값이면 `NULL`을 반환하고, 다르다면 첫 번째 인자를 반환한다.
  - `NULLIF(arg1, arg2)`
- Example
  - `NULLIF(10, 10)` --> NULL
  - `NULLIF(10, 12)` --> 10

#### Example

|Name|Department|
|---|---|
|Lauren|A|
|Vinton|A|
|Claire|B|

```sql
-- 2가 나온다.
SELECT (
        SUM(CASE WHEN department = 'A' THEN 1 ELSE 0 END) /
        SUM(CASE WHEN department = 'B' THEN 1 ELSE 0 END)
) AS department_ratio
FROM depts

DELETE FROM depts
WHERE department = 'B'

-- ERROR: division by zero
SELECT (
        SUM(CASE WHEN department = 'A' THEN 1 ELSE 0 END) /
        SUM(CASE WHEN department = 'B' THEN 1 ELSE 0 END)
) AS department_ratio
FROM depts

-- 해결 -> NULL 값이 있는 산술 연산은 무조건 결과값이 NULL이다.
SELECT (
        SUM(CASE WHEN department = 'A' THEN 1 ELSE 0 END) /
        NULLIF(SUM(CASE WHEN department = 'B' THEN 1 ELSE 0 END), 0)
) AS department_ratio
FROM depts
```

## Views

- 종종 프로젝트에 자주 사용하는 테이블과 조건의 특정 조합을 발견하는 경우가 많다.
- 동일한 쿼리를 반복적으로 수행하는 대신 VIEW를 만들어 간단한 호출로 반복되는 쿼리를 빠르게 조회할 수 있다.
- VIEW는 저장된 쿼리의 데이터베이스 개체이다.
- VIEW는 PostgreSQL에서 가상 테이블로서 접근할 수 있다.
- VIEW는 물리적으로 데이터를 저장하는 것이 아니라 단순히 쿼리만 저장하는 것이다.
- 따라서 기존의 VIEW를 업데이트하고 변경할 수 있다.

```sql
CREATE (OR REPLACE) VIEW customer_info AS
SELECT first_name, last_name, address
FROM customer
INNER JOIN address
ON customer.address_id = address.address_id;

SELECT * FROM customer_info

DROP VIEW IF EXISTS customer_info;

ALTER VIEW customer_info RENAME TO c_info;
```

## SQL 개요

### SQL(<- SEQUEL)(<- Structured Query Language)

- 관계형 데이터베이스에서 데이터 정의(`DDL`), 조작(`DML`), 제어(`DCL`)를 위해 사용하는 언어
  - 추가로 `TCL(Transaction Control Language)` 4가지로 구성되어 있다.
- 표준 SQL: ISO의 표준 규격을 따르는 SQL
- SQL 기본 작성 규칙
  - 문장 마지막은 세미콜론(;)으로 끝남
  - 명령어, 객체명, 변수명은 대/소문자 구분이 없음
    - **데이터 값은 대/소문자를 구분함**
  - 날짜와 문자열에는 작은 따옴표를 사용
    - age = 20
    - name = 'KIM'
  - 단어와 단어 사이는 공백 또는 줄바꿈으로 구분
  - 주석문
    - `-- 이것은 주석입니다.`
    - `/* 여기부터 여기까지 주석입니다. */`

### SQL 구문 유형

- 데이터 정의어(DDL: Data Definition Language)
  - 데이터의 구조(스키마)를 정의하기 위한 명령어
  - CREATE, ALTER, DROP, RENAME, TRUNCATE
- 데이터 조작어(DML: Data Manipulation Language)
  - 데이터를 검색 또는 변형하기 위한 명령어
  - SELECT, INSERT, UPDATE, DELETE
- 데이터 제어어(DCL: Data Control Language)
  - 사용자에게 객체에 대한 권한을 부여/취소하기 위한 명령어
  - GRANT, REVOKE
- 트랜잭션 제어어(TCL: Transaction Control Language)
  - 변경 내용을 확정/취소하기 위한 명령어
  - COMMIT, ROLLBACK

## Function

### 단일 행 함수 특징

```sql
SELECT PLAYER_NAME, LENGTH(PLAYER_NAME) AS 이름길이 FROM PLAYER;
```

- 각 행(Row)에 대해 개별적으로 작용하고 그 결과를 반환함
  - 단일 행 내에 있는 하나 또는 복수의 값을 인수로 사용
  - 여러 행에 걸친 값을 사용할 수 없음
- 함수 중첩(함수의 인자로 함수를 사용)이 가능함
- SELECT, WHERE, ORDER BY 절에 사용 가능함

### 단일행 내장 함수

- [문자형 함수](https://www.w3resource.com/PostgreSQL/chr-function.php)
  - 문자형 변수 처리
  - LOWER, UPPER, ASCII, CHR, CONCAT, SUBSTR, LENGTH, LTRIM, RTRIM, TRIM, ...
- [숫자형 함수](https://www.postgresqltutorial.com/postgresql-math-functions)
  - 숫자형 변수 처리
  - ABS, MOD, CEIL, FLOOR, ROUND, ...
- [변환 함수](https://www.postgresql.org/docs/current/functions-formatting.html)
  - 문자, 숫자, DATE형 값의 타입 변환
  - CAST, TO_CHAR, TO_NUMBER, TO_DATE, TO_TIMESTAMP
    - `select to_char(now(), 'HH12:MI:SS'); -- 12:18:52`
- 날짜형 함수
  - NOW, EXTRACT, ...
- 제어 함수
  - 논리값에 따른 값의 처리
  - CASE, ...
- NULL 관련 함수
  - NULL 처리
  - COALESCE, NULLIF

## TCL

### 트랜잭션

- 데이터베이스의 논리적 연산 단위
  - 의미적으로 분할할 수 없는 최소의 단위
  - 일반적으로 하나의 트랜잭션은 여러 SQL 문장을 포함함
  - **성공 시 모든 연산을 반영, 취소 시 모든 연산을 취소함 -> All or Nothing**
- 트랜잭션의 예
  - 도서 주문
    - 재고 수량 감소, 주문 내역 생성, 결제, 포인트 적립 => 하나의 트랜잭션
  - 계좌 이체
    - 원 계좌의 잔액 감소, 다른 계좌의 잔액 증가 => 하나의 트랜잭션
    - 이체라는 하나의 행위를 실행하기 위해서는 한 쪽에서 잔액을 감소하고, 다른 한 쪽에서 잔액을 증가시키는 두 연산이 발생한다.
  - 교통카드 충전
    - 잔액 증가, 결제 => 하나의 트랜잭션

### 트랜잭션의 특성 (ACID 특성)

|특성|설명|
|--|--|
|원자성(Atomicity)|트랜잭션에서 정의된 연산들은 모두 성공적으로 실행되던지 아니면 전혀 실행되지 않은 상태로 남아 있어야 한다.(all or nothing)|
|일관성(Consistency)|트랜잭션이 실행되기 전의 데이터베이스 내용이 잘못되어 있지 않다면, 트랜잭션이 실행된 이후에도 데이터베이스의 내용에 잘못이 있으면 안된다.|
|고립성(Isolation)|트랜잭션이 실행되는 도중에 다른 트랜잭션의 영향을 받아서는 안된다.|
|지속성(Durability)|트랜잭션이 성공적으로 수행되면 그 트랜잭션이 갱신한 데이터베이스의 내용은 영구적으로 저장된다.|

- 지속성
  - 트랜잭션을 수행하기 위해서는 데이터베이스에 있는 테이블들을 메모리로 가지고 와야한다.
  - 메모리에서 연산을 모두 수행하고 나서, 문제없이 종료되면 DB에 기록하라는 commit 명령어 실행
  - 반영되고 난 데이터베이스의 내용은 다른 트랜잭션이 수행되지 않는 한 영구적으로 안바뀐다는 의미.

### 트랜잭션의 ACID 특성을 보장하기 위해 DBMS는 동시성 제어(Concurrency Control) 수행

- DBMS에서는 트랜잭션 관리자 라는 모듈을 따로 둔다. 이 트랜잭션 관리자가 동시성 제어를 수행.
  - 트랜잭션 따로 따로 수행하기 되면 비효율적이기 때문에 섞여서 수행되는데 섞여도 수행되지만 혼자 따로 따로 고립되어서 수행된 것처럼 보장해 주는 알고리즘이 동시성 제어 알고리즘.
- Lock 기반, Timestamp 기반

### TCL

- COMMIT 실행 전 상태에서는
  ```sql
  UPDATE player SET height = height + 10;
  ```
  - 변경된 내용은 메모리에 임시로 저장됨
  - 현재 사용자는 증가한 HEIGHT 값을 읽을 수 있음(메모리에 저장되어 있어서)
  - 다른 사용자는 증가 전 HEIGHT 값만 읽을 수 있음(아직 DB에는 반영 X)
  - HEIGHT에는 잠금(Locking)이 설정되어 다른 사용자는 값을 변경할 수 없음
- COMMIT 실행 후
  ```sql
  COMMIT;
  ```
  - 변경된 내용은 DB에 저장됨
  - 변경 내용은 모든 다른 사용자가 볼 수 있음
  - 이전 데이터는 모두 사라짐(별도 로그 보관 시 복구 가능)
  - 관련된 행에 대한 잠금이 해제되어 모든 사용자가 변경할 수 있음
- 트랜잭션을 제어하기 위한 명령어
  - COMMIT: 변경된 내용을 DB에 영구적으로 반영
  - ROLLBACK:
    - 기본 - 변경된 내용을 버리고 변경 전 상태(마지막 COMMIT)로 복귀
    - SAVEPOINT를 지정한 경우, 지정한 저장점까지만 복귀
    - SAVEPOINT: 부분 복귀를 위해 지정한 저장점
- 트랜잭션은 SQL문 실행 시 자동 시작, COMMIT/ROLLBACK 실행 시 종료
- 자동 커밋 / 자동 롤백
  - DDL 문장 수행 시 DDL 수행 전에 자동으로 커밋(auto commit)
  - DB를 정상적으로 접속 종료하면 자동 커밋
  - 애플리케이션의 이상 종료로 DB와의 접속이 단절되었을 때 자동 롤백

### ROLLBACK

- https://www.postgresql.org/docs/current/sql-rollback-to.html

![image](https://github.com/marchidx04/db-practice/assets/126429401/de220a76-f0bf-460d-bc7f-c1154ad82aee)

- 위 사진에서 현재 위치에서 ROLLBACK이 일어나면 갈 수 있는 곳은 3가지
  - 트랜잭션이 시작하는 부분(그냥 ROLLBACK하면 시작 위치가 감)
  - SAVEPOINT A
  - SAVEPOINT B

```sql
COMMIT;

UPDATE player SET height = height + 10;
DELETE FROM player;

ROLLBACK;
```

- 변경한 내용이 모두 취소됨
- 이전 데이터가 다시 재저장됨
- 관련된 행에 대한 잠금이 해제되어 모든 사용자가 변경할 수 있음

#### SAVEPOINT

- 미리 지정한 SAVEPOINT까지만 ROLLBACK
  - 특정 저장점까지 롤백하면 그 이후의 명령과 저장점은 모두 무효가 됨
- 일부 tool에서는 지원되지 않음
- 동일 이름으로 여러 저장점 정의시 나중에 정의한 저장점이 유효
  ```sql
  ...
  SAVEPOINT PT1;

  ...
  SAVEPOINT PT2;

  ...
  SAVEPOINT PT3;

  ...
  ROLLBACK TO PT1; -- SAVEPOINT PT1으로 이동 -> 이후 명령은 모두 무효가 됨
  ```

## != NULL, <> NULL, IS NOT NULL 차이

### !=, <>, IS NOT 차이

- 의미상으로는 같지 않다를 나타내어 모두 동일한 결과를 반환할 것이라고 생각할 수 있다.
- 하지만,,
  - !=, <>, IS NOT에는 타입 비교 대상의 범위 차이가 있다.
  - != 와 <> 연산자는 동일한 기능을 하며 원시 타입(Primitive Type)에 대해서만 동작하고,
  - IS NOT NULL은 모든 타입에 대해 동작한다.

### !=, <>, IS NOT NULL 예시
 
#### != NULL을 사용한 경우

![image](https://github.com/marchidx04/db-practice/assets/126429401/13bdb082-9def-4eca-8f81-3b5f96eb79bf)

  - != NULL을 사용한 경우에는 출력한 레코드가 없는 것을 확인할 수 있다.

#### IS NOT NULL을 사용한 경우

![image](https://github.com/marchidx04/db-practice/assets/126429401/1eb5c365-8545-463f-9297-d3e09b4ce1bf)

  - IS NOT NULL을 사용한 경우에는 delv_msg 컬럼이 NULL인 아닌 레코드들을 반환한 것을 확인할 수 있다.

### 결론

- 결론은 NULL이 아닌지를 비교하려면 무조건 IS NOT NULL 조건식을 사용해야 한다!
- 그리고 NULL이 아닌 값(varchar, int 등)을 비교하려면 !=, <>를 사용하자.

## 고급 집계 함수

### ROLLUP

```sql
select category, gubun1, count(*)
from selling_products
group by rollup(category, gubun1)
order by category, gubun1;
```

- 소그룹 별 소계 계산 추가 (순서 중요)
  - 각 category, gubun1 별 집계
  - 각 category 별 집계
  - 전체 집계

#### GOUPING 함수 사용

```sql
select category, grouping(category), gubun1, grouping(gubun1), count(*)
from selling_products
group by rollup(category, gubun1)
order by category, gubun1;
```

- GROUPING(컬럼)
  - 컬럼을 모두 합쳐서 집계를 하면 1, 아니면 0
  - 즉, 집계에 포함된 것이 아니면 1, 포함된 경우면 0
  - 어디까지 집계된 것인지 알려주는 용도로 사용됨

#### GROUPING + CASE 사용

```sql
select
  case grouping(category)
    when 1 then 'All Category'
    else category
  end as category,
  case grouping(gubun1)
    when 1 then 'All gubun1'
    else gubun1
  end as gubun1,
  count(*)
from selling_products
group by rollup(category, gubun1)
order by category, gubun1;
```

### CUBE

- 다차원 소계 계산 추가 (순서 무관)
- 모든 조합의 집계 계산 -> 시스템 부하가 크다.

```sql
select category, gubun1, count(*)
from selling_products
group by cube(category, gubun1)
order by category, gubun1;

select category, gubun1, count(*)
from selling_products
group by cube(gubun1, category)
order by category, gubun1;

-- ROLLUP과 UNION을 사용하여 구현할 수도 있음
select category, gubun1, count(*)
from selling_products
group by rollup(category, gubun1)
order by category, gubun1;

UNION -- UNION ALL을 하면 겹치는 부분이 두 번 반환된다.

select category, gubun1, count(*)
from selling_products
group by rollup(gubun1, category)
order by category, gubun1;
```

#### GROUPING SETS

- 여러 컬럼 각각에 대해 반복적으로 그룹화

```sql
-- category로만 집계, gubun1로만 집계한 레코드를 반환
select category, gubun1, count(*)
from selling_products
group by grouping sets(gubun1, category);
```

### COALESCE + GROUPING SETS

```sql
select
 	coalesce(category, 'All Category'),
	coalesce(gubun1, 'All Gubun1'),
	count(*)
from selling_products
group by grouping sets(gubun1, category);
```

- NULL 값을 채울 수 있음

### GROUP BY와 UNION ALL

```sql
-- GROUP BY와 UNION ALL을 사용하여 GROUPING SETS 구현 가능
select 'All Category' as category, gubun1, count(*)
from selling_products
group by gubun1

union all

select category, 'All Gubun1' as gubun1, count(*)
from selling_products
group by category;
```

## 윈도우 함수

- 기존 관계형 DB는 커럼간 연산은 쉽지만 행간의 연산은 어렵다.
- 행 간의 관계 정의를 위해 윈도우 함수가 고안되었다.
  - ex: 각 직원이 속한 부서 내에서 급여 순위는?
- 중첩(Nested) 사용 불가
- 서브쿼리에서도 사용 가능
- 종류
  - 순위: RANK, DENSE_RANK
  - 집계: SUM, MAX, MIN, AVG, COUNT
    - 현재 나의위치(값)과 연계해서 집계를 하기 때문에 일반 집계함수와는 조금 다르다.
  - 행 순서: FIRST_VAVLUE, LAST_VALUE, LAG, LEAD
  - 비율: RATIO_TO_REPORT, PERCENT_RANK, NTILE
  - 통계: CORR, STDDEV, VARIANCE 등

### 윈도우 함수

```sql
SELECT WINDOW_FUNCTION (ARGUMENTS) OVER ([PARTITION BY 컬럼] [ORDER BY 컬럼] [WINDOWNIG 절])
FROM table_name;
```

- WINDOW_FUNCTION: 기존 함수 or WINDOW 함수로 추가된 함수
- ARGUMENTS (인수): 함수에 따라 0 ~ N개의 인수 지정
- `PARTITION BY` 절: 전체 집합을 기준에 의해 소그룹으로 나눌 수 있음
- ORDER BY 절: 순서를 지정할 기준 항목
- WINDOWING 절: 함수의 대상이 되는 행 기준의 범위를 지정
  - `ROWS / RANGE` 중 하나를 선택하여 사용
    - `ROWS`: 행의 수를 기준으로 한 범위
    - `RANGE`: 값을 기준으로 한 범위
    - 자기를 기준으로 위 아래를 찾아볼 수 있는데 행의 수/값을 기준으로 범위를 지정할 수 있다.

#### WINDOWING 절의 사용 예

```sql
-- 해당 파티션 내에서 앞의 한 행, 현재 행, 뒤의 한 행을 범위로 지정 => 총 3개의 레코드 반환
ROWS BETWEEN 1 PRECEDNIG AND 1 FOLLOWING

-- 해당 파티션 내에서 (현재 행의 값 - 50) ~ (현재 행의 값 + 150)을 범위로 지정
RANGE BETWEEN 50 PRECEDING AND 150 FOLLOWING

-- 현재 파티션의 첫 행부터 현재 행까지 지정
RANGE UNBOUNDED PRECEDING
```

### RANK 함수

- 동일한 값에는 동일한 순위 부여
- 동일한 순위를 여러 건으로 취급
  - ex: 1등이 2명인 경우 -> 1등, 1등, 3등

#### 전체 급여 순위, JOB 내에서 급여 순위 출력

```sql
SELECT job, ename, sal
      RANK() OVER (ORDER BY sal DESC) AS ALL_RANK, -- 전체 급여 순위
      RANK() OVER (PARTITION BY job ORDER BY sal DESC) AS JOB_RANK -- 같은 직무를 가진 사람 중에 내 sal의 순위
FROM emp;
```

![image](https://github.com/MarchIDX/march_erp/assets/126429401/5dc74e76-0c3b-4eef-8047-2c6ff9e9af12)

#### RANK, DENSE_RANK, ROW_NUMBER 비교

- `RANK`
  - 동일값에 동일 순위 비교
  - 동일 순위를 여러 건으로 취급(1등, 1등, 3등, ...)
- `DENSE_RANK`
  - 동일값에 동일 순위 부여
  - 동일 순위를 한 건으로 취급(1등, 1등, 2등, ...)
- `ROW_NUMBER`
  - 동일값에 다른 순위 부여(1등, 2등, 3등)

```sql
SELECT job, ename, sal
      RANK() OVER (ORDER BY sal DESC) AS ALL_RANK, -- 전체 급여 순위
      DENSE_RANK() OVER (ORDER BY sal DESC) AS DENSE_RANK, -- 전체 급여 순위
      ROW_NUMBER() OVER (ORDER BY sal DESC) AS ROW_NUMBER, -- 전체 급여 순위
FROM emp;
```

![image](https://github.com/MarchIDX/march_erp/assets/126429401/1fde7e32-4851-4674-879e-54884e63e955)

### 집계 윈도우 함수

#### MAX / MIN

- 각 직원이 속한 직업 내에서 급여의 최대값을 함께 출력하기 위한 질의

```sql
SELECT job, ename, sal,
      MAX(sal) OVER (PARTITION BY job) AS job_max,
FROM emp
ORDER BY job, ename;
```

![image](https://github.com/MarchIDX/march_erp/assets/126429401/48e0768a-7a97-4ca4-bfab-0a2c48c92943)

#### SUM / AVG

- 각 직업 내에서, 본인보다 높은 급여를 받는 직원의 급여 총합(본인 포함)

```sql
SELECT job, ename, sal,
      -- 값이 동일한 경우 동시에 반영
      SUM(sal) OVER (PARTITION BY job ORDER BY sal DESC RANGE UNBOUNDED PRECEDING)
FROM emp;
```

- `RANGE UNBOUNDED PRECEDING)`
  - 해당 파티션의 맨 처음 로우부터 자신까지

![image](https://github.com/MarchIDX/march_erp/assets/126429401/ebaed8d8-3f6a-4a33-ac05-9d10c6618959)

- JOB이 ANALYST인 경우네는 SAL의 값이 같기 때문에 동시에 반영된 것을 확인할 수 있다.
- SALESMANE의 경우도 3100에서 1250을 더하는 것이 아니라 2500을 더한 것을 확인할 수 있다.

#### SUM / AVG (Cont'd)

- 각 직업 내에서, 본인 바로 위 + 본인 + 본인 바로 아래의 급여 합 출력

```sql
SELECT job, ename, sal,
      SUM(sal) OVER (PARTITION BY job ORDER BY sal ASC ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)
FROM emp;
```

![image](https://github.com/MarchIDX/march_erp/assets/126429401/1e1d01a7-e540-4a25-be77-b755d7ad2ba7)

#### COUNT

- 본인보다 급여가 100 적은 직원부터 200 많은 직원까지의 총 직원 수

```sql
SELECT ename, sal,
      COUNT(*) OVER (ORDER BY sal RANGE BETWEEN 100 PRECEDING AND 200 FOLLOWING) AS mov_count
FROM emp;
```

![image](https://github.com/MarchIDX/march_erp/assets/126429401/5924a155-c0a6-40fa-a65b-cf82edd155c5)

### FIRST_VALUE (or LAST_VALUE)

- 각 파티션에서 가장 먼저 (또는 나중에) 나온 값

```sql
SELECT deptno, ename, sal, FIRST_VALUE(ename) OVER
  (PARTITION BY deptno ORDER BY sal DESC, ename ASC) AS rich_emp -- deptno 기준 sal이 많고, ename이 먼저인 값
FROM emp;
```

![image](https://github.com/MarchIDX/march_erp/assets/126429401/5629dcea-8d67-4bef-96ba-279f149b8934)\

### 행 순서 윈도우 함수

#### LAG (or LEAD)

- 각 파티션에서 해당 행의 몇 번째 이전 (또는 이후) 행의 값을 가져옴
- `job = 'SALESMAN'`인 모든 직원에 대해서, 급여 기준 본인 바로 윗 사람의 급여와 아랫 사람의 급여를 출력하는 질의를 완성하시오.
  - `LAG(sal, 1) = LAG(sal)` / `LEAD(sal, 1) = LEAD(sal)`

```sql
SELECT ename, sal,
  LAG(sal, 1) OVER(ORDER BY sal DESC) AS higher_sal,
  LEAD(sal, 1) OVER(ORDER BY sal DESC) AS lower_sal
FROM emp
WHERE job = 'SALESMAN';
```

![image](https://github.com/MarchIDX/march_erp/assets/126429401/498ff57c-05db-4dbe-aab8-dc7890fc60dd)

#### LAG (or LEAD) 함수 (Cont'd)

- `LAG(sal, 2, 0)`
  - 2번째 앞의 행을 가져오고, 가져올 행이 없는 처음 두 행의 값은 0으로 채움
    - `0`을 생략하면 해당 두 행은 NULL로 표시됨
  - ex: 급여 기준 본인보다 2칸 위 직원의 급여를 출력하시오.
    ```sql
    SELECT ename, sal,
      LAG(sal, 2, 0) OVER (ORDER BY sal DESC) AS higher_sal -- 가져올 행이 없으면 0으로 채움
    FROM emp;
    ```

![image](https://github.com/MarchIDX/march_erp/assets/126429401/78eefb13-3ae7-4f28-98e1-3b608c025a3a)
  
### 비율 윈도우 함수

#### RATIO_TO_REPORT 함수

- 파티션 내 전체 SUM(컬럼) 값에 대한 행별 백분율(`양`에 대한 비율)

```sql
-- 동일 JOB 내에서, 본인의 급여가 차지하는 비율 출력
SELECT job, ename, sal,
  ROUND(RATIO_TO_REPORT(sal) OVER (PARTITION BY job), 2) AS r_r
FROM emp
ORDER BY job;
```

![image](https://github.com/MarchIDX/march_erp/assets/126429401/ed1c1d39-a057-4da5-8f39-1a779918fe01)

#### PERCENT_RANK 함수

- 행의 순서별 백분율을 구함
  - 가장 먼저 나온 행 = 0, 가장 나중에 나온 행 = 1

```sql
-- 동일 JOB 내에서, 본인의 급여가 상위 몇 %에 있는지 출력
SELECT deptno, ename, sal,
      100*(PERCENT_RANK() OVER (PARTITION BY deptno ORDER BY sal DESC)) || '%' AS p_r
FROM emp;
```

- `deptno = 20`의 경우 총 5건
  - 0, 0.25, 0.5, 0.75, 1
  - 동일 값은 작은 백분율(0%가 두개, 60%가 두개)을 중복 적용

#### CUME_DIST 함수

- 현재 행에 대해, 현재 행보다 작거나 같은 건수에 대한 누적 백분율
  - 0 초과, 1 이하의 값을 가짐

```sql
-- 동일 JOB 내에서, 본인의 급여가 누적 순서 몇 %에 있는지 출력
SELECT deptno, ename, sal,
      100*(CUME_DIST() OVER (PARTITION BY deptno ORDER BY sal DESC)) || '%' AS cume
FROM emp;
```

![image](https://github.com/MarchIDX/march_erp/assets/126429401/c42f509a-567b-4f91-b4c2-2b4d2b2a9055)

- `deptno = 20`의 경우 총 5건
  - 0.2, 0.4, 0.6, 0.8, 1
  - 동일값은 큰 백분율(20%가 아닌 40%가 두개)을 중복 적용

#### NTILE 함수

- 파티션별 전체 건수를 N등분한 결과를 구함

```sql
-- 전체 사원을 급여 순으로 정렬하고, 급여 기준 4개의 그룹으로 분리하는 질의를 작성
SELECT ename, sal, NTILE(4) OVER (ORDER BY sal DESC) AS 급여구간
FROM emp;
```

![image](https://github.com/MarchIDX/march_erp/assets/126429401/aa4b2ff0-7c2e-4c98-9f97-b29fa3675650)

- N 구간으로 나누어서 자기가 급여 기준 몇 그룹에 있는지 확인 가능하다.

## PL / SQL

### 절차형 SQL (PL/SQL)

- PL(Procedural Language)/SQL 개념 및 특징
  - SQL에서도 절차적인 프로그래밍이 가능하도록 지원
    - SQL문의 반복 실행, 조건에 따른 분기 등 가능
  - Block 구조로 설계되어 각 기능별 모듈화 가능
    - 여러 SQL 문장을 Block으로 묶어서 한 번에 서버로 보냄 -> 통신량 감소
  - 변수, 상수 등을 선언하여 SQL 문장 간 값 교환 가능
  - DBMS 정의 에러, 사용자 정의 에러 사용 가능
- PL/SQL 유형
  - Procedure
  - User Defined Function
  - Trigger
   
### PL/SQL Block 구조

![image](https://github.com/MarchIDX/march_erp/assets/126429401/ae5a5856-ac59-4415-b5a0-385cd38eda6b)

- Header에 기본적인 정보가 담김
- Declaration Section - 선언 구간
  - `IN`, `OUT` 외에 임시적으로 사용할 변수 선언
- Execution Section - 연산을 수행하는 구간
- Exception Section - 예외 처리 구간

### Procedure

- `CREATE Procedure`
  - `CREATE OR REPLACE` 사용 시 동일 이름의 Procedure를 덮어씀
  - 삭제 시 `DROP Procedure`
- `Mode`
  - `IN`: 운영체제에서 프로시저로 전달하는 변수
  - `OUT`: 프로시저에서 운영체제로 전달되는 변수
  - `INOUT`: 거의 사용하지 않음
- `/`
  - 프로시저 컴파일 시작

```sql
CREATE [OR REPLACE] Procedure [Procedure_name]
(argument1 [mode] data_type1, argument2 [mode] data_type2, ...)
IS ...
BEGIN ...
    EXCEPTION ...
END;
/
```

### Procedure 생성 예

![image](https://github.com/MarchIDX/march_erp/assets/126429401/0e5bd738-9095-4e6c-9d5c-9ffb4728b1c3)

- DEPT 테이블에 새로운 부서를 등록하는 Procedure
  - 부서가 존재하는 경우 -> 이미 데이터가 존재합니다 반환
  - 부서가 없는 경우 -> 부서데이터 입력
  - 그동안 바로 INSERT를 했던거와 달리 확인할 수 있음

```sql
-- ORACLE
CREATE OR REPLACE PROCEDURE p_dept_insert
(v_deptno IN NUMBER, v_dname IN VARCHAR2, v_loc IN VARCHAR2, v_result OUT VARCHAR2)

IS
    cnt NUMBER := 0;
BEGIN
    SELECT COUNT(*) INTO cnt -- cnt 변수에 할당
    FROM dept
    WHERE deptno = v_deptno;
    if cnt > 9 then
      v_result := '이미 등록된 부서번호이다.';
    else
      INSERT INTO dept (deptno, dname, loc) VALUES (v_deptno, v_dname, v_loc);
      COMMIT;
      v_result := '입력 완료!!';
    end if;
EXCEPTION
  WHEN OTHERS THEN
    ROLLBACK;
    v_result := 'ERROR 발생';
END;
/ 
```

### Procedure 실행 예

- 10번 부서가 이미 등록되어 있는 경우

```sql
variable rslt varchar2(30);

EXECUTE p_dept_insert(10, 'dev', 'seoul', :rslt);
print rslt;

RSLT -- 이미 등록된 부서번호이다.

EXECUTE p_dept_insert(50, 'dev', 'seoul', :rslt);
print rslt;

RSLT; -- 입력 완료!
```

### 사용자 정의 함수(User Defined Function)

- Procedure처럼 절차형 SQL을 로직과 함께 저장한 명령문의 집합
  - cf) 내장 함수(Built-in Function) - 벤더에 의해 정의된 함수
- Procedure와 달리 반드시 **수행 결과값을 Return**해야 함
  - ex) 절대값을 반환하는 사용자 정의 함수 UTIL_ABS의 정의 및 호출 예

```sql
CREATE OR REPLACE FUNCTION
    util_abs (v_input IN NUMBER) RETURN NUMBER
IS
    v_return NUMBER := 0;
BEGIN
    IF v_input < 0 THEN
        v_return := v_input * (-1);
    ELSE
        v_return := v_input;
    END IF;
RETURN v_return;
END:
/
```

### 트리거(Trigger)

- DML문 수행 시, 이와 연결된 동작을 자동으로 수행하도록 작성된 프로그램
- 사용자가 명시적으로 호출하지 않으며, 조건이 맞으면 자동으로 수행됨
  - Procedure: Execute 명령어로 실행
  - Function: 함수 이름으로 실행
  - Trigger: 생성된 후 DML에 의해 자동으로 실행
- 주로 데이터 무결성 보장을 위해 FK처럼 동작하거나, 실시간 집계성 테이블 생성에 사용됨
- 보안 적용, 유효하지 않은 트랜잭션 예방, 업무 규칙 적용, 감사 제공 등에도 사용
- OLTP 시스템에서는 부하로 인해 성능이 저하될 수 있음
- ROLLBACK 시, 원 트랜잭션 뿐 아니라 Trigger에 의해 실행된 연산도 모두 취소됨
  - Trigger는 INSERT, DELETE, UPDATE문과 연결된 하나의 트랜잭션 내에서 수행되는 작업으로 이해해야 함
    - Procedure: Begin ~ End 사이에 COMMIT, ROLLBACK 사용 가능
    - Trigger: Begin ~ End 사이에 COMMIT, ROLLBACK 사용 불가
  - Row Trigger와 Statement Trigger로 구분

### 트리거(Trigger) 주요 구문

- **FOR EACH ROW**
  - Row Trigger / Statement Trigger의 지정을 위한 구문
  - `FOR EACH ROW` 사용 -> Row Level Trigger -> SQL 문장의 각 행마다 Trigger 발생
  - `FOR EACH ROW` 생략 -> Statement Level Trigger -> SQL 한 문장에 한 번만 Trigger 발생
- **AFTER/BEFORE**
  - Trigger 수행 시점 명시
- **:NEW/:OLD**
  - `:NEW`는 문잣 수행 후의 정보를 갖는 구조체
    - ex) o_prod = :NEW.product
  - `:OLD`는 문장 수행 전의 정보를 갖는 구조체
  |구분|:OLD|:NEW|
  |---|---|---|
  |INSERT|NULL|입력된 레코드 값|
  |UPDATE|UPDATE되기 전의 레코드의 값|UPDATE된 후의 레코드 값|
  |DELETE|레코드가 삭제되기 전 값|NULL|
- **변수 선언**
  - ORDER.order_date%TYPE;
    - ORDER 테이블의 order_date 컬럼과 동일한 타입으로 선언

### 트리거 생성 예

- 새로운 주문이 입력되면 판매 집계 테이블이 업데이트되는 시나리오
- 주문 관리 테이블
  - ORDER: ORDER_DATE, PRODUCT, QTY, AMOUNT
- 판매 실적 관리 테이블
  - SALES: SALE_DATE, PRODUCT, QTY, AMOUNT
- 새로운 주문 입력 시
  - ORDER 테이블에 새로운 주문 추가 -> `Trigger` -> SALES 테이블 갱신 또는 추가
    - SALES에 해당 주문일자, 해당 상품이 있으면 -> 기존 수량/금액 업데이트
    - 예: (2023-06-01, "P01", 5, 250,000) -> (2023-06-01, "P01", 6, 300,000)
    - SALES에 해당 주문일자, 해당 상품이 없으면 -> 새 레코드 추가
    
```sql
CREATE OR REPLACE TRIGGER summary_sales
  AFTER INSERT
  ON ORDER
  FOR EACH ROW

DECLARE
  o_date order.order_date%TYPE;
  o_prod order.product%TYPE;

BEGIN
  o_date = :NEW.order_date;
  o_prod = :NEW.product;
  UPDATE sales SET qty = qty + :NEW.qty, amount = amount + :NEW.amount; -- amount는 sales 테이블의 amount, :NEW.amount는 order 테이블의 INSERT 후의 amount
  WHERE sales_date = o_date AND product = o_prod;

  IF SQL%NOTFOUND THEN
    INSERT INTO sales VALUES (o_date, o_prod, :NEW.qty, :NEW.amount);
  END iF;
END;
/
```

## SQL 최적화

### 주요 내용

- 옵티마이저와 실행 계획
  - 규칙기반 옵티마이저
  - 비용기반 옵티마이저
- 인덱스 기본
  - B-Tree 인덱스
  - Index Scan / Full Scan
- 조인 수행 원리
  - NL Join
  - Sort Merge Join
  - Hash Join
  - 조인을 어떻게 찾을지에 대한 이슈
- 성능 모델링하고는 다르고 SQL 문을 명령할 때 어떻게 해야 더 효율적으로 실행시킬 수 있을지에 초점

### 옵티마이저와 실행 계획

#### 옵티마이저(Optimizer) 개념

- SQL은 사용자의 요구사항만 기술할 뿐 처리 과정은 기술하지 않음
  - 실제로 SQL문을 실행하기 전에 최적의 실행계획을 찾아야 함
- 옵티마이저: 사용자의 요구사항을 만족하는 다양한 실행계획(Execution Plan) 중 최적의 실행계획을 결정
  - SQL을 어떤 순서로 어떻게 실행할지 결정하는 작업 수행

#### 옵티마이저의 종류

- 규칙기반 옵티마이저(RBO: Rule-Based Optimizer)
  - 거의 사용하지 않지만, 하위 버전 호환성을 위해 유지
- 비용기반 옵티마이저(CBO: Cost-Based Optimizer)

### 옵티마이저 동작 구조

![image](https://github.com/MarchIDX/march_erp/assets/126429401/44495273-5881-4b15-b014-eece6d4ebe32)

- 사용자가 SQL을 입력하면 파서가 해석(* 바로 실행하는 것이 아니라 어떻게 실행할지를 먼저 고민)
- 옵티마이저에서 최적의 실행계획을 찾는 과정을 한다.
  - 규칙 기반 옵티마이저는 규칙이 이미 다 정의가 되어 있는 상태에서 계산을 통해 바로 실행계획이 나온다.
  - 비용 기반 옵티마이저의 경우에는 바로 계산이 되는 것이 아니라 통계가 필요하다.
    - 통계치를 이용해서 비용을 뽑아야 하기 때문에 딕셔너리라는 것을 사용한다.
    - 즉, 딕셔너리를 사용해서 비용을 어느정도 예측한 다음에 총 비용이 가장 적게 드는 것을 실행계획으로 결정한다.
- 옵티마이저에서 결정한 실행계획에 따라서 SQL 실행 엔진을 통해 실제 실행해서 결과를 사용자에게 반환한다.

### 규칙 기반 옵티마이저

![image](https://github.com/MarchIDX/march_erp/assets/126429401/320ab476-0def-4206-855d-88ff9aa79119)

- 1 ~ 14번까지는 인덱스 스캔(Index Scan)을 사용한다.
- 마지막 15번이 Full Scan으로 최악의 경우를 말한다.
- **규칙 1. Single row by Rowid**
  - ROWID를 통해서 테이블에서 하나의 행을 액세스
- **규칙 2. Single row by unique or primary key**
  - Unique Index를 통해서 하나의 행을 액세스하는 방식
  - 인덱스를 먼저 액세스하고 인덱스의 ROWID를 추출하여 행을 액세스함
- **규칙 8. Composite index**
  - 복합 인덱스로 검색하는 경우
  - 복합 인덱스 사이의 우선 순위 규칙
    - 인덱스 매칭률이 높을수록 우선
    - 인덱스 매칭률이 동일하면, 구성 컬럼이 많을수록 우선
    - ex) (1) A/B/C 조합과 (2) D/E 조합이 있을 때, 매칭률이 동일하면 (1) 우선
- **규칙 9. Single column index**
  - 단일 컬럼 인덱스에 `=` 조건으로 검색
- **규칙 10. Bounded range search on indexed columns**
  - 인덱스가 생성되어 있는 컬럼에 양쪽 범위를 한정하는 형태로 검색
    - LIKE, BETWEEN
  - 예: `X BETWEEN '10' AND '20'` / 예: `X LIKE '1%'`
- **규칙 11. Unbounded range search on indexed columns**
  - 인덱스가 생성되어 있는 컬럼에 한쪽 범위만 한정하는 형태로 검색
    - >, >=, <, <=
  - 예: A > 10
- **규칙 15. Full Table Scan (전체 테이블 액세스)**
  - 일반적으로 속도가 느리지만 병렬 처리 가능

#### 인덱스 매칭률

- 예
  - index1 <- (학번, 나이, 생년월일)로 구성
  - index2 <- (주민번호, 학년)으로 구성
- 다음 질의에서는 어떤 인덱스를 사용하는 것이 효율적인가?
  ```sql
  SELECT *
  FROM student
  WHERE 학번 = '123' AND 나이 = 20 AND 주민번호 = '123456' AND 학년 = 3
  ```
  - `인덱스 매칭률 = (질의문에서 Equal(=) 조건으로 사용된 인덱스 컬럼 수) / 인덱스를 구성하는 컬럼 수
    - **단 첫 컬럼부터 순서로 연속되는 경우만을 Matching으로 인정함**
  - index1의 경우는 학번, 나이가 Equal 조건으로 사용된 인덱스 컬럼이므로 매칭률은 `2/3`
  - index2의 경우는 주민번호, 학번이 Equal 조건으로 사용된 인덱스 컬럼이므로 매칭률은 `2/2`

#### 인덱스 매칭률 계산 예

- 인덱스가 순서대로 (시, 구, 동)으로 구성된 경우
  - WHERE 시 = '서울시';
    - 매칭률: `1/3`
  - WHERE 시 = '서울시' AND 구 = '강남구';
    - 매칭률: `2/3`
  - WHERE 시 = '서울시' AND 구 = '강남구' AND 동 = '역삼동';
    - 매칭률: `3/3`
  - WHERE 시 = '서울시' AND 동 = '역삼동';
    - 매칭률: `1/3`
    - 시까지만 매칭
  - WHERE 시 = '서울시' AND 구 LIKE '강%' AND 동 = '역삼동';
    - 매칭률: `1/3`
    - 시까지만 매칭
  - WHERE 시 = '서울시' AND 구 = '강남구' AND 동 LIKE '역%';
    - 매칭률: `2/3`
  - WHERE 동 = '역삼동' AND 시 = '서울시' AND 구 = '강남구';
    - 매칭률: `2/3`
    - 시, 구까지 매칭
  - WHERE 구 = '강남구' AND 시 = '서울시' AND 동 = '역삼동';
    - 매칭률: `3/3`

### 비용기반 옵티마이저

- 규칙기반 옵티마이저의 한계
  - 몇 개의 규칙만으로 현실의 모든 상황을 설명하기 어려무 
    - 예: 항상 `=`이 BETWEEN보다 빠른가?
  - SQL문 처리에 예상되는 비용(소요시간, 자원사용량)을 최소화하기 위한 방법 필요
  - 테이블, 인덱스 컬럼등의 객체에 대한 통계정보, 시스템 통계정보 활용
    - 정확한 통계정보 관리는 비용기반 최적화의 중요한 요소임
  - 통계정보, DBMS 버전, DBMS 설정 정보에 따라 동일한 SQL문도 서로 다른 실행계획이 생성될 수 있음

#### 비용기반 옵티마이저의 구성

![image](https://github.com/MarchIDX/march_erp/assets/126429401/44e13e6d-74d4-41e1-949f-b5a39de34c19)

- 질의 변환기
  - SQL문을 보다 작업이 용이한 형태로 변환
- 대안 계획 생성기
  - 동일한 결과를 생성하는 다양한 대안 계획 생성(통계를 기반)
    - 가능한 모든 계획을 생성하지는 않으므로, 최적 대안이 누락되는 경우도 있음
- 비용 예측기
  - 다양한 통계 정보를 활용하여 대안 계획의 비용 예측

#### 실행 계획

- 실행계획은 조인 순서, 조인 기법, 액세스 기법, 최적화 정보 등으로 구성
- 조인 기법: NL Join, Sort Merge Join, Hash Join 등 사용
- 액세스 기법: Index Scan, Full Table Scan
- 최적화 정보는 실제 실행 결과가 아닌 통계 정보 바탕의 예측치
  - Cost: 상대적인 비용 정보(숫자가 낮을수록 유리)
  - Card: 주어진 조건을 만족하는 행의 수
  - Bytes: 결과 집합이 차지하는 메모리의 양
- 비용기반 옵티마이저의 실행 계획 예
  ![image](https://github.com/MarchIDX/march_erp/assets/126429401/ca8f2f5b-2442-4211-885b-c5406e189867)

### Full Table Scan(FTS)

- 테이블 내의 모든 데이터를 읽어가면서 조건 검색
  - 고수위 마크(HWM, High Water Mark) 아래의 모든 블록을 읽으므로 시간이 많이 소요됨
- 다음의 경우 옵티마이저는 FTS 선택
  - SQL문에 조건이 존재하지 않는 경우 -> 모든 데이터가 답
  - SQL문 조건에 사용 가능한 인덱스가 없는 경우
  - 조건을 만족하는 데이터가 매우 많은 경우 (옵티마이저의 선택)
    - 인덱스 스캔은 한 번의 I/O 요청에 한 Block씩 데이터를 읽음
    - FTS는 한 번의 I/O 요청으로 여러 Block을 동시에 읽음
  - 병렬처리 방식으로 처리하는 경우

### 인덱스(Index)

- 검색 속도의 향상을 위한 기술
  - 지나치게 많은 인덱스 생성 시 시간 및 공간 낭비
  - 인덱스된 필드의 업데이트시 시간 증가
- B-Tree 인덱스가 가장 보편적
  - 일치(=) 검색과 범위(BETWEEN, <, >) 검색에 모두 적합
- 유형
  - 인덱스 유일 스캔(Index Unique Scan)
    - Unique Index를 사용하여 단 하나의 데이터를 추출하는 방식
    - Index 구성 컬럼에 조건이 모두 `=`로 주어진 경우
  - 인덱스 범위 스캔(Index Range Scan)
    - 한 건 이상의 데이터를 추출하는 방식
    - Non-Unique Index를 이용하는 경우
    - Index 구성 컬럼에 `=` 이외의 조건이 주어진 경우

#### B-Tree Index

![image](https://github.com/MarchIDX/march_erp/assets/126429401/ff7bfc3e-7a81-4105-b71c-ce97692cef51)

- Root Block
  - Branch Block 중 최상위
- Branch Block
  - 분기를 목적으로 함
  - 다음 단계를 가리키는 포인터를 갖고 있음
- Leaf Block
  - 트리의 가장 아래 단계
  - **인덱스 구성 컬럼의 데이터와 해당 행의 위치를 가리키는 식별자로 구성됨**
  - Leaf Block 간 양방향 링크를 갖고 있음 -> 오름 차순, 내림 차순 검색 가능

#### B-Tree 검색 예

![image](https://github.com/MarchIDX/march_erp/assets/126429401/64018cd4-5fca-42d9-b77e-bff1f944adf4)

- 37 이상 55 미만의 값 검색
  - 37을 Root Node에서부터 찾는다.
  - Leaf Block에서 37값을 찾고 다시 Root Node에서 38 -> 39 -> 40 순으로 찾는 것이 아니라
  - Leaf Block의 양방향 링크를 통해 다음 값을 찾는다.

## 조인 기법

### NL(Nested Loop) 조인

![image](https://github.com/MarchIDX/march_erp/assets/126429401/a7f38a6d-c075-4b7e-a890-2683fb0fdec8)

- 선행 테이블(외부 테이블)과 후행 테이블(내부 테이블) 조인
- 선행 테이블의 조건을 만족하는 행 추출 -> 후행 테이블 읽으면서 조인
  - 이 과정을 선행 테이블의 조건을 만족하는 행 수만큼 반복
- 결과 행의 수가 적은 테이블을 조인 순서상 선행 테이블로 두는 것이 유리
  - 결과 행의 수가 많은 것을 선행 테이블로 두는 경우에는 그 다음 후행 테이블까지 들여다봐야 하기 때문에 비효율적이다.
  - 따라서 결과 행의 수가 적은 것을 선행 테이블로 둬 그 다음 후행에서 덜 들여다볼 수 있도록 하는 것이 효율적이다.

### Sort Merge 조인

![image](https://github.com/MarchIDX/march_erp/assets/126429401/2300490e-067f-4af4-a68d-ecca8223df91)

- 조인 컬럼 기준으로 데이터를 정렬한 후 조인 수행
- 정렬할 데이터가 많은 경우에는 성능 저하
- Non-Equi 조인 가능

### Hash 조인

![image](https://github.com/MarchIDX/march_erp/assets/126429401/44c0d844-13ec-47e0-85f8-cb4d31732095)

- 조인 컬럼 기준으로 해쉬 함수를 수행하여, 동일한 해쉬 값을 갖는 경우에만 실제 값을 비교하여 조인 수행
- 해시 테이블을 메모리에 생성해야 함
  - 결과 행의 수가 적은 테이블을 선행 테이블(Build Input)로 사용하는 것이 좋음
  - Build Input, Probe Input

### 튜닝 Big3 조인

- NL(Nested Loop) 조인
  - 소규모 데이터 인덱스를 사용하여 작업하는 OLTP 업무에 적합
  - 연결 조건 인덱스 및 드라이빙 테이블의 우선순위에 따라 성능 결정
- Sort Merge 조인
  - 인덱스를 사용하지 않으며, Non-Equi Join도 가능
  - 정렬 수행에 비용이 소요됨
- Hash 조인
  - Equi Join만 가능하며, 데이터가 많은 경우 유리
  - 해시 함수 계산에 비용이 소모됨
  - 결과 행의 수가 적은 테이블을 선행 테이블로 사용하는 경우 성능 향상
- 단계별로 다른 조인 적용 가능
  - ex: A/B 조인에 NL Join, 그 결과와 C 조인에 Hash Join


## 데이터 모델링의 이해

### 데이터 모델링

#### 모델링이란?

- 복잡한 현실세계를 추상화, 단순화하여 일정한 표기법에 의해 명확하게 표현하는 것
- 추상화, 단순화, 명확화
  - 추상화(모형화, 가설적): 현실세계를 일정한 형식에 맞추어 표현을 한다는 의미로 정리할 수 있다.
  - 단순화: 복잡한 현실세계를 약속된 규약에 의해 제한된 표기법이나 언어로 표현하여 쉽게 이해할 수 있도록 하는 개념을 의미한다.
  - 명확화: 누구나 이해하기 쉽게 하기 위해 대상에 대한 애매모호함을 제거하고 정확하게 현상을 기술하는 것을 의미한다.
  - 구체화, 복잡화, 일반화 등이 아님
- 모델(Model) 현실 세계의 추상화된 반영

#### 모델링의 관점

- **데이터 관점(What)**
  - 데이터와 데이터 간 관계, 업무와 데이터 간 관계를 모델링
  - 데이터에 접근하는 방법(How), 사람(Who)과는 무관
- **프로세스 관점(How)**
  - 업무가 실제로 하고 있는 일 또는 해야할 일을 모델링
- 데이터와 프로세스의 상관 관점(Interaction)
  - 업무 처리 방법에 따라 데이터가 받는 영향을 모델링

### 데이터 모델링의 3단계 진행

- 개념적 모델링
  - ERD 도출
  - 주로 Chen 타입의 ERD 사용
  - 관계가 자체 속성을 가질 수 있음
- 논리적 모델링
  - 테이블 도출
  - 기본키(PK)와 외래키(FK) 지정
  - 정규화 및 반정규화 수행(물리적 모델리의 요구 반영)
- 물리적 모델링
  - 실제 DBMS에 맞는 테이블 구축
  - 데이터 타입 정의, 인덱스 설계, 뷰 설계

### 데이터베이스 3단계 구조

- 외부 스키마(External Schema)
  - 각 사용자 또는 응용프로그램이 바라보는 스키마
- 개념 스키마(Conceptual Schema)
  - 모든 사용자의 관점을 통합한 스키마
  - DB에 저장되는 데이터와 그들 간의 관계를 표현
- 내부 스키마(Physical Schema)
  - DB가 물리적으로 저장된형식
- 데이터 모델링은 통합 관점의 개념 스키마를 만들어 가는 과정

### 데이터의 독립성과 종속성

#### 데이터 종속성

- 응용 프로그램에 대한 데이터의 종속성
  - 예) 파일 시스템
    - 응용 프로그램과 데이터가 상호 의존적
    - 데이터를 저장한 파일 구조가 변경되면 이에 대응되는 응용 프로그램도 변경되어야 함

### 데이터 독립성

- 데이터 구조가 변경되어도 응용 프로그램이 변경될 필요가 없음
- 논리적 독립성 + 물리적 독립성으로 실현됨
- 데이터 독립성이 유지되지 않으면?
  - 데이터의 중복성 및 복잡도 증가
  - 요구사항 대응 난이도 증가 -> 데이터 유지보수 비용 증가
  