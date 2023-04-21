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

- 쿼리의 첫 번째 실행 순서는 FROM절이다. FROM 절에서는 조회하는 **테이블 전체**를 가져온다.

### WHERE 절

- WHERE 절에서는 FROM 절에서 읽어온 테이블에서 **조건에 맞는 결과만 갖도록** 데이터를 필터링한다.

### HAVING 절

- HAVING 절은 여러 조건들을 처리한 후 남은 데이터에서 어떤 열을 출력해줄지 선택한다.

### SELECT 절

- SELECT 절은 여러 조건들을 처리한 후 남은 데이터에서 **어떤 열을 출력해줄지 선택**한다.

### ORDER BY 절

- 어떤 열까지 출력할지 정했다면 행의 순서를 어떻게 보여줄지 **정렬**해주는 절이 ORDER BY이다.
- **HAVING 절은 각 그룹에 조건을 걸기 때문에 포퍼먼스가 떨어지게 된다.**

### LIMIT 절

- LIMIT 절은 결과 중 몇 개의 행을 보여줄지 선택한다.


## SELECT

> SELECT은 가장 많이 사용되는 SQL 문장으로, 테이블에서 정보를 검색하는데 사용된다.

### SELECT 문 예제

```bash
SELECT column_name FROM table_name
SELECT * FROM table_1
SELECT c1, c2 FROM table_2
```

- 일반적으로, 모든 열을 포함한 테이블의 전체 정보가 필요하지 않은 경우 `*`를 사용하지 않는 것이 좋다.
  - 쿼리 서버와 응용 프로그램 간의 트래픽이 증가하여 결과 검색이 느려질 수 있기 때문에 필요한 컬럼만 조회하는 것이 좋다.
  - 모든 열을 포함한 테이블의 전체 정보가 정말 필요한 경우에만 *(asterisk)를 사용한다.

### SELECT DISTINCT

```bash
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
  ```bash
  SELECT * FROM table_name WHERE created_at BETWEEN '2023-03-01' AND '2023-04-01';
  ```
  - `2023-03-31`로 작성하면 `2023-03-31 00:00:00` 이전의 정보들을 가져오기 때문에 `2023-04-01`로 작성해야 한다.

### IN

- 특정 경우에는 여러 가능한 값을 확인해야 할 수도 있다.
  - 예) 사용자 이름이 아려진 이름 목록에 포함되어 있는지 확인하려는 경우
- `IN` 연산자를 사용하여 다중 옵션 목록에 값이 포함되어 있는지 확인하는 조건을 만들 수 있다.
  ```bash
  value_1 IN (option1, option2, ..., option_n)
  ```
