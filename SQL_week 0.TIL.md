# SQL Advanced 0주차 과제

## 15.2.15 서브쿼리
-서브쿼리는 다른 명령문 내의 `SELECT` 명령문이다.

**장점**

-명령문의 각 부분을 분리할 수 있도록 쿼리를 구조화

-복잡한 `JOIN`이나 `UNION`을 사용해야하는 작업을 다른 방식으로  수행하게 해줌

-좋은 가독성: SQL(Structured Query Language)로 불리게 된 계기

**예시**

```sql
DELETE FROM t1
WHERE s11 > ANY
 (SELECT COUNT(*) /* no hint */ FROM t2
  WHERE NOT EXISTS
   (SELECT * FROM t3
    WHERE ROW(5*t2.s1,77)=
     (SELECT 50,11*s1 FROM t4 UNION SELECT 50,77 FROM
      (SELECT * FROM t5) AS t5)));
```
-서브쿼리는 스칼라, 단일 행 / 열, 테이블 등을 반환

-`DISTINCT`,`GROUP BY`, `ORDER BY`, `LIMIT` 등 `SELECT`가 포함할 수 있는 절들을 가짐

## 15.2.15.2 서브쿼리 사용의 비교

**가장 일반적인 형태**

```sql
non_subquery_operand comparison_operator (subquery)
```

또는

```sql
non_subquery_operand LIKE (subquery)
```

```sql
=  >  <  >=  <=  <>  !=  <=>
```

위의 비교 연산자를 사용하여 아래 예시와 같은 형태가 나옴

```sql
... WHERE 'a' = (SELECT column1 FROM t1)
```

-서브쿼리를 스칼라 값과 비교하려면 반드시 단일 값(스칼라)을 반환해야 함

-서브쿼리를 행(row) 생성자와 비교하려면 반드시 행(row) 서브쿼리여야 하며 행 생성자와 동일한 개수의 값을 반환해야 함

## 15.2.15.3 ANY, IN, SOME

-`ANY` 키워드는 비교 연산자와 함께 사용되며 서브쿼리가 반환하는 컬럼의 값 중 하나라도 비교 조건을 만족하면 `TRUE`를 반환

-예시
```sql
SELECT s1 FROM t1 WHERE s1 > ANY (SELECT s1 FROM t2);
```
-`IN`은 =`ANY`와 같으므로 아래 두 예시는 같은 명령문, 그러나 =`ANY`와 달리 `IN`만 표현식 목록을 취할 수 있음
```sql
SELECT s1 FROM t1 WHERE s1 = ANY (SELECT s1 FROM t2);
SELECT s1 FROM t1 WHERE s1 IN    (SELECT s1 FROM t2);
```
-`SOME`은 쿼리에서 "a와 같지 않은 b가 존재한다"는 의미로 사용됨
```sql
SELECT s1 FROM t1 WHERE s1 <> SOME (TABLE t2);
```
## 15.2.15.4 ALL

-`ALL`은은 반드시 비교 연산자 뒤에 사용, 서브쿼리가 반환하는 컬럼의 모든 값에 대해 비교 조건이 참이면 TRUE를 반환한다는 의미
```sql
SELECT s1 FROM t1 WHERE s1 > ALL (SELECT s1 FROM t2);
```
-`NOT IN`은 <>`ALL`과 같으므로 아래 두 예시는 같은 명령문
```sql
SELECT s1 FROM t1 WHERE s1 <> ALL (SELECT s1 FROM t2);
SELECT s1 FROM t1 WHERE s1 NOT IN (SELECT s1 FROM t2);
```
아래 두 조건 하에 ALL과 NOT IN에서 TABLE을 사용 가능능

1.서브쿼리 내의 테이블은 오직 하나의 컬럼만 포함해야 함

2.서브쿼리는 컬럼 표현식에 의존하지 않아야 함
```sql
SELECT s1 FROM t1 WHERE s1 <> ALL (TABLE t2);
SELECT s1 FROM t1 WHERE s1 NOT IN (TABLE t2);
```
## 15.2.15.6 EXISTS or NOT EXISTS

-서브쿼리가 어떤 행을 반환하면, `EXISTS` 서브쿼리는 TRUE, `NOT EXISTS` 서브쿼리는 FALSE를 의미
```sql
SELECT column1 FROM t1 WHERE EXISTS (SELECT * FROM t2);
```
-보통 `EXISTS` 서브쿼리는 `SELECT *`로 시작하지만, `SELECT 5`나 `SELECT column1` 도 가능

 `NOT EXISTS` 서브쿼리는 거의 항상 상관 관계(correlation)를 포함하기 때문에 `t2`에 NULL 값만 포함된 행이라도 존재하면 `EXISTS` 조건은 TRUE가 된다

 **예시: 모든 도시에 존재하는 가게는 무엇인가?**
```sql
 SELECT DISTINCT store_type FROM stores
  WHERE NOT EXISTS (
    SELECT * FROM cities WHERE NOT EXISTS (
      SELECT * FROM cities_stores
       WHERE cities_stores.city = cities.city
       AND cities_stores.store_type = stores.store_type));
```
-위 예시는 이중 중첩된 `NOT EXISTS` 쿼리

-`NOT EXISTS`가 또 다른 `NOT EXISTS` 내에 포함되어 있으며  중첩된 `NOT EXISTS`는 "모든 y에 대해 x가 TRUE인가?"라는 질문에 답한다

## 15.2.15.10 서브쿼리 에러

- 지원되지 않는 명령문 사용
```sql
SELECT * FROM t1 WHERE s1 IN (SELECT s2 FROM t2 ORDER BY s1 LIMIT 1)
```

- 서브쿼리 칼럼(열) 수의 오류
```sql
SELECT (SELECT column1, column2 FROM t2) FROM t1;
```

- 서브쿼리 행의 수의 오류
```sql
SELECT * FROM t1 WHERE column1 = (SELECT column1 FROM t2);
```
서브쿼리가 최대 하나의 행을 반환해야 하지만 두 개 이상이 나오는 경우 쿼리를 재작성해야함

- 서브쿼리에서 테이블 사용의 오류
```sql
UPDATE t1 SET column2 = (SELECT MAX(column1) FROM t1);
```
공통 테이블 표현식이나 파생 테이블 사용으로 해결

## CTE - WITH
**CTE (공통 테이블 표현식)은 단일 명령문에서 일시적인 결과 집합을 생성하고 이를 동일한 쿼리 내에서 여러 번 참조가 가능하게 함**

- CTE를 지정하려면 하위 절이 하나 이상 있는 `WITH`절을 사용

- 각 하위절(subclause)은 서브쿼리를 포함하며 이 서브쿼리는 결과 집합을 생성하고 별칭을 지음

**예시**
```sql
WITH
  cte1 AS (SELECT a, b FROM table1),
  cte2 AS (SELECT c, d FROM table2)
SELECT b, d FROM cte1 JOIN cte2
WHERE cte1.a = cte2.c;
```
- WITH 절을 사용하여 CTE를 정의하며 다른 CTE에서 기존 CTE를 참조 가능하고, 선택적인 문법 요소이며, DML 문에서도 활용 가능함

**CTE의 칼럼 이름 결정**

- CTE 이름 뒤에 괄호로 묶인 이름 목록이 있는 경우 그것이 칼럼 이름
```sql
WITH cte (col1, col2) AS
(
  SELECT 1, 2
  UNION ALL
  SELECT 3, 4
)
SELECT col1, col2 FROM cte;
```
- 목록에 있는 이름의 수는 결과 집합에 있는 열의 수와 같아야 하며 그렇지 않은 경우 열 이름은 SELECT 부분의 첫 번째 선택 목록에서 가져옴
```sql
WITH cte AS
(
  SELECT 1 AS col1, 2 AS col2
  UNION ALL
  SELECT 3, 4
)
SELECT col1, col2 FROM cte;
```

**`WITH`가 허용되는 경우**

-서브 쿼리나 `SELECT`, `UPDATE`, 및 `DELETE` 문장의 시작 부분
```sql
WITH ... SELECT ...
WITH ... UPDATE ...
WITH ... DELETE ...
```
```sql
SELECT ... WHERE id IN (WITH ... SELECT ...) ...
SELECT * FROM (WITH ... SELECT ...) AS dt ...
```
- `WITH`는 동일한 수준에서는 하나 의 절만 허용되지만 서로 다른 수준에서 발생하는 경우 여러 개의 절을 포함
```sql
WITH cte1 AS (SELECT 1)
SELECT * FROM (WITH cte2 AS (SELECT 2) SELECT * FROM cte2 JOIN cte1) AS dt;
```

## 문제 풀이 - 많이 주문한 테이블 찾기
```sql
SELECT *
FROM tips
WHERE total_bill > (SELECT AVG(total_bill) FROM tips);
```
테이블 전체의 평균 식사 금액 계산 후 평균보다 높은 경우 출력

![.](image/KakaoTalk_20250318_152014163.png)

## 문제 풀이 - 레스토랑의 대목
```sql
WITH weekday_sales AS (
    SELECT day, SUM(total_bill) AS total_sales
    FROM tips
    GROUP BY day
    HAVING SUM(total_bill) >= 1500
)

SELECT t.*
FROM tips t
JOIN weekday_sales ws ON t.day = ws.day;
```
요일별 매출액 합계를 구한 후 매출이 1500 달러 이상인 요일의 결제 내역을 출력

![.](image/KakaoTalk_20250318_154346980.png)

## 문제 풀이 - 식품분류별 가장 비싼 식품의 정보 조회하기

**1. 서브쿼리를 이용한 풀이**
```sql
SELECT category, price, product_name
FROM FOOD_PRODUCT
WHERE (category, price) IN (
    SELECT category, MAX(price) AS price
    FROM FOOD_PRODUCT
    GROUP BY category
    HAVING category IN ('과자','국','김치','식용유'))
ORDER BY price DESC;
```

- 과자, 국 등 특정 카테고리만 골라낸 후 같은 카테고리에서 최대 가격을 가진 제품만 출력, 가격이 높은 순으로 정렬

![.](image/subsub.png)


**2. CTE / WITH를 이용한 풀이**
```sql
WITH max_price AS (
    SELECT CATEGORY, MAX(PRICE) AS MAX_PRICE
    FROM FOOD_PRODUCT
    WHERE CATEGORY IN ('과자', '국', '김치', '식용유')
    GROUP BY CATEGORY
)
SELECT f.CATEGORY, m.MAX_PRICE, f.PRODUCT_NAME
FROM FOOD_PRODUCT f
JOIN max_price m ON f.CATEGORY = m.CATEGORY AND f.PRICE = m.MAX_PRICE
ORDER BY m.MAX_PRICE DESC;
```

- CTE (max price)를 생성하고 메인 테이블에서 FOOD_PRODUCT와 JOIN
- 가격이 CTE에서 구한 MAX_PRICE와 같은 제품만 선택 후 가격이 높은 순으로 정렬

![.](image/with.png) 

**두 방식 비교**

- 서브쿼리를 활용한 풀이는 간단한 쿼리에 적합하지만 카테고리별 MAX(PRICE)를 찾을 때마다 서브쿼리가 실행된다는 비효율적인 단점이 존재

- `WITH`를 활용했을 때는 MAX(PRICE)를 한 번만 계산 후 재사용할 수 있으며 가독성이 좋아 효율적이라는 특징이 존재, 따라서 데이터가 많아질 경우 CTE를 활용하는 것이 더 좋아보임