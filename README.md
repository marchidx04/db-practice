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
  # Cheryl, Theresa, Sherri
  WHERE name LIKE '_her%';
  ```
- PostgreSQL은 `LIKE` 및 `ILIKE` 외에 전체적인 정규표현식 기능을 지원한다.
  - https://www.postgresql.org/docs/12/functions-matching.html

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
  # WHERE 필터링을 적용하고 나서 GROUP BY로 호출한 후에 HAVING에 판매액 총액이 1,000달러보다 큰 값을 조건으로 다시 필터링
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
# 벤다이어그램이 대칭이기 때문에 테이블 순서를 바꿔도 같은 결과가 나온다.
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

- 타임존 확인
  - `show timezone;`
- 타임존 설정(변경)
  - `set timezone = 'America/Los_Angeles';`
- Date and Time 정보와 관련된 데이터 타입:
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
    - 만약 위에 시차가 17시간 차이나는 로스엔젤레스로 타임존을 변경하게 되면 `2023-04-24 02:01:40.920477-08`(서머타임 적용)로 변경된다.
    - `timestamp` 타임존에 따른 시간 변경이 적용되지 않는다.
