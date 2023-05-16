# SQL(PostgreSQL)

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
  FROM Table_A.col_match = Table_B.col_match
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

### UNIONS

```
SELECT column_name(s) FROM Table1
UNION
SELECT column_name(s) FROM Table2
```

- `UNION` 연산자를 사용하면 2개 이상의 SELECT 문의 결과 세트를 결합할 수 있다.
- `JOIN`과 `UNION`의 차이는 `UNION`은 두 결과를 직접 붙인 것이다.
- 두 `SELECT` 문의 결과를 서로의 바로 위에 붙여준다.

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

- 서브쿼리의 실행 결과로여러 칼럼을 반환한다.
- 메인쿼리의 조건절에 여러 칼럼을 동시에 비교할 수 있다.
- 서브쿼리와 메인쿼리에서 비교하고자 하는 칼럼 개수와 칼럼의 위치가 동일해야 한다.

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
  - 현재 사용자는 증가한 HEIGHT 값을 읽을 수 있음
  - 다른 사용자는 증가 전 HEIGHT 값만 읽을 수 있음
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

![image](https://github.com/marchidx04/db-practice/assets/126429401/de220a76-f0bf-460d-bc7f-c1154ad82aee)

```sql
COMMIT;

UPDATE player SET height = height + 10;
DELETE FROM player;

ROLLBACK;
```

- 변경한 내용이 모두 취소됨
- 이전 데이터가 다시 재저장됨
- 관련된 행에 대한 잠금이 해제되어 모든 사용자가 변경할 수 있음