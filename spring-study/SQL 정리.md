# SQL 정리

#### 인덱스를 사용하지 못하는 SQL 문장은 지양

1. INDEX COLUMN의 변형

   ```sql
   SELECT *
   FROM DEPT
   WHERE SUBSTR(DNAME, 1, 3) = 'ABC'
   ```

1. NOT Operator

   ```mysql
   SELECT *
   FROM DEPT
   WHERE SUBSTR(DNAME, 1, 3) = 'ABC'
   ```


2. NULL or NOT NULL VALUE의 비교

   ```sql
   SELECT *
   FROM EMP
   WHERE ENAME IS NOT NULL
   ```


3. Optimizer의 취사선택

   ```SQL
    SELECT  *
    FROM    EMP
    WHERE   JOB   LIKE 'AB%'
    AND     EMPNO = '7890'
   ```






### SQL 연습

```SQL
-- 10th October 2012의 평균 주문량보다 큰 주문량을 모두 구하시오
SELECT 		*
FROM		ORDERS
WHERE 		PURCH_AMT > (
					SELECT 	AVG(PURCH_AMT)
					FROM 	ORDERS
					WHERE	ORD_DATE = '10/10/2012');
```



```SQL
--Write a query in SQL to list all the information of the actors who played a role in the movie 'Annie Hall'
SELECT *
FROM 	ACTOR A
WHERE 	ACT_ID IN (
			SELECT 	ACT_ID
			FROM 	MOVIE_CAST B
			WHERE 	MOV_ID IN (
            			SELECT MOV_ID
            			FROM MOVIE
            			WHERE MOV_TITLE = 'Annie Hall'
            ));
```



```SQL
-- Write a query in SQL to find the titles of all movies directed by the director whose first and last name sare Woddy Allen.
SELECT 	MOV_TITLE
FROM	MOVIE
WHERE	MOV_ID IN (
			SELECT 	MOV_ID
			FROM	MOVIE_DIRECTION
			WHERE	DIR_ID IN (
            			SELECT 	DIR_ID
            			FROM	DIRECTOR
            			WHERE 	DIR_FNAME = 'Woody' AND DIR_LNAME = 'Allen'));
```

