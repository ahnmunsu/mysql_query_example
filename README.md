# Useful SQL query examples

## 목차
- **[OUTER JOIN 주의 사항](#OUTER-JOIN-주의-사항)**
- **[ANTI JOIN](#ANTI-JOIN)**
- **[레코드를 칼럼으로 변환](#레코드를-칼럼으로-변환)**
- **[하나의 컬럼을 여러 컬럼으로 분리](#하나의-컬럼을-여러-컬럼으로-분리)**
- **[AUTO INCREMENT 컬럼 규칙](#AUTO-INCREMENT-컬럼-규칙)**
- **[AUTO INCREMENT 증가값 조회](#AUTO-INCREMENT-증가값-조회)**
- **[정렬과 함께 순서 부여](#정렬과-함께-순서-부여)**
- **[N번째 레코드만 가져오기](#N번째-레코드만-가져오기)**
- **[누적 합계 구하기](#누적-합계-구하기)**
- **[그룹별 랭킹 쿼리](#그룹별-랭킹-쿼리)**
- **[랭킹 업데이트 하기](#랭킹-업데이트-하기)**
- **[페이징 쿼리](#페이징-)**
---
## OUTER JOIN 주의 사항
OUTER JOIN에서 OUTER로 조인되는 테이블의 칼럼에 대한 조건은 모두 ON 절에 명시해야 한다.
```sql
SELECT * FROM employees e
  LEFT JOIN dept_manager mgr ON mgr.emp_no=e.emp_no AND mgr.dept_no='d001';
```
## ANTI JOIN
tab_test1 테이블에는 있지만 tab_test2에는 없는 id 값만 가져오는 쿼리이다.  
서브 쿼리와 NOT EXIST를 사용하는 방법도 있지만 레코드 건수가 많다면 ANTI JOIN이 효과적이다.
```sql
SELECT t1.id
FROM tab_test t1
  LEFT JOIN tab_test2 t2 ON t1.id=t2.id
WHERE t2.id IS NULL
```
## 레코드를 칼럼으로 변환
```sql
SELECT
  SUM(CASE WHEN dept_no='d001' THEN emp_count ELSE 0 END) AS count_d001,
  SUM(CASE WHEN dept_no='d002' THEN emp_count ELSE 0 END) AS count_d002,
  SUM(CASE WHEN dept_no='d003' THEN emp_count ELSE 0 END) AS count_d003,
  SUM(CASE WHEN dept_no='d004' THEN emp_count ELSE 0 END) AS count_d004,
  SUM(CASE WHEN dept_no='d005' THEN emp_count ELSE 0 END) AS count_d005,
  SUM(CASE WHEN dept_no='d006' THEN emp_count ELSE 0 END) AS count_d006,
  SUM(CASE WHEN dept_no='d007' THEN emp_count ELSE 0 END) AS count_d007,
  SUM(CASE WHEN dept_no='d008' THEN emp_count ELSE 0 END) AS count_d008,
  SUM(CASE WHEN dept_no='d009' THEN emp_count ELSE 0 END) AS count_d009,
  SUM(emp_count) AS count_total
FROM (
  SELECT dept_no, COUNT(*) AS emp_count FROM dept_emp GROUP BY dept_no
) tb_derived;

```
|count_d001|count_d002|count_d003|...|count_total|
|---|---|---|---|---|
|20211|17346|17786|...|331603|


---
## 하나의 컬럼을 여러 컬럼으로 분리
```sql
SELECT de.dept_no,
  SUM(CASE WHEN e.hire_date BETWEEN '1980-01-01' AND '1989-12-31' THEN 1 ELSE 0 END) AS cnt_1980,
  SUM(CASE WHEN e.hire_date BETWEEN '1990-01-01' AND '1999-12-31' THEN 1 ELSE 0 END) AS cnt_1990,
  SUM(CASE WHEN e.hire_date BETWEEN '2000-01-01' AND '2009-12-31' THEN 1 ELSE 0 END) AS cnt_2000,
  COUNT(*) AS cnt_total
FROM dept_emp de, employees e
WHERE e.emp_no=de.emp_no
GROUP BY de.dept_no;
```
|dept_no|cnt_1980|cnt_1990|cnt_2000|cnt_total|
|---|---|---|---|---|
|d001|11038|9171|2|20211|
|d002|9580|7765|1|17346|
|d003|9714|8068|4|17786|
|...|...|...|...|...|

---
## AUTO INCREMENT 컬럼 규칙
- AUTO_INCREMENT 속성을 가진 컬럼은 반드시 프라이머리 키나 유니크 키의 일부로 정의돼야 한다.
- AUTO_INCREMENT 속성을 가진 컬럼 하나로 프라이머리 키를 생성할 때는 아무런 제약이 없다.
- 여러 개의 컬럼으로 프라이머리 키를 생성할 때
  - AUTO_INCREMENT 속성의 컬럼이 제일 앞일 때
    AUTO_INCREMENT 속성의 컬럼이 프라이머리 키의 제일 앞쪽에 위치하면 MyISAM이나 InnoDB 테이블에서 아무런 제약이 없다.
  - AUTO_INCREMENT 속성의 컬럼이 제일 앞이 아닐 때
    다음 예제에서, tb_test 테이블은 fd1과 fd2 컬럼으로 구성된 프라이머리 키만을 가지고 있다. MyISAM 테이블에서는 이와 같이 사용할 수 있지만 InnoDB 테이블에서는 이렇게 생성할 수 없다. InnoDB에서는 반드시 UNIQUE INDEX (fd2)와 같이 fd2로 시작하는 UNIQUE 키를 하나 더 생성해야만 한다.
    
 ``` sql
 CREATE TABLE tb_test {
   fd1 CHAR,
   fd2 INT AUTO_INCREMENT,
   PRIMARY KEY(fd1, fd2)
 }
 ```

---
## AUTO INCREMENT 증가값 조회
"SELECT MAX(member_id) FROM ..."과 같은 쿼리는 잘못된 결과를 반환할 수도 있으므로 다음과 같이 조회하는 것이 좋다.
```sql
CREATE TABLE tb_autoincrement (
  seq_no INT NOT NULL AUTO_INCREMENT,
  name VARCHAR(30),
  PRIMARY KEY (seq_no)
);

INSERT INTO tb_autoincrement VALUES (NULL, 'Gerogi Fellona');
SELECT LAST_INSERT_ID();
```

---
## 정렬과 함께 순서 부여
```sql
SET @ranking:=0;

UPDATE salaries
  SET ranking=( @ranking := @ranking + 1 )
ORDER BY salary DESC;
```
---
## N번째 레코드만 가져오기
```sql
SELECT *
FROM departments, (SELECT @rn:=0) x
HAVING (@rn:=@rn+1)=3
ORDER BY dept_name;
```
---
## 누적 합계 구하기
FROM절의 (SELECT @acc_salary:=0)는 누적 합계 값을 임시로 저장해 두는 @acc_salary 변수를 0으로 초기화하기 위한 서브 쿼리로 사용됐다.
```sql
SELECT emp_no, salary, (@acc_salary:=@acc_salary+salary) AS acc_salary
FROM salaries, (SELECT @acc_salary:=0) x
LIMIT 10;
```
---
## 그룹별 랭킹 쿼리
```sql
SELECT
  emp_no, first_name, last_name,
  @prev_firstname, @rank,
  IF(@prev_firstname=first_name,
    @rank:=@rank+1, @rank:=1+LEAST(0,@prev_firstname:=first_name)) rank,
  @prev_firstname, @rank
FROM employees, (SELECT @rank:=0) x1, (SELECT @prev_firstname:=NULL) x2
WHERE first_name IN ('Georgi', 'Bezalel')
ORDER BY first_name, last_name;
```
|emp_no|first_name|last_name|@prev_firstname|@rank|rank|@prev_firstname|@rank|
|---|---|---|---|---|---|---|---|
|297135|Bezalel|Acton|NULL|0|1|Bezalel|1|
|25442|Bezalel|Adachi|Bezalel|1|2|Bezalel|2|
|...|...|...|...|...|...|...|...|
|428804|Bezalel|Zallocco|Bezalel|227|228|Bezalel|228|
|90035|Georgi|Aamodt|Bezalel|228|1|Georgi|1|
|...|...|...|...|...|...|...|...|
---
## 랭킹 업데이트 하기
```sql
SELECT @rank:=0;

UPDATE tb_ranking r
SET r.rank_no = (@rank:=@rank+1)
ORDER BY r.member_score DESC;
```
---
## 페이징 쿼리
#### 일반적으로 사용하는 테이블의 구조와 페이징 쿼리
페이지를 이동할 때마다 디스크를 읽기 부하가 늘어나는 문제가 있다.
```sql
/* 테이블 구조 */
CREATE TABLE tb_article (
  board_id INT NOT NULL,
  article_id INT NOT NULL AUTO_INCREMENT,
  article_title VARCHAR(100) NOT NULL,
  ...
  PRIMARY KEY (article_id),
  INDEX ix_boardid (board_id, article_id)
);

/* 페이징 쿼리 */
SELECT *
FROM tb_article WHERE board_id=1
ORDER BY article_id DESC LIMIT n, m;
```
#### 불필요한 접근을 제거하기 위한 페이징 쿼리
이전 페이지의 가장 마지막 article_id를 조건절에 넣어서 디스크 읽기 부하를 줄일 수 있다.
```sql
SELECT *
FROM tb_article WHERE board_id=1 AND article_id<165
ORDER BY article_id DESC LIMIT 0, 20;
```
---
## 출처
개발자와 DBA를 위한 Real MySQL / 이성욱 / 위키북스
https://wikibook.co.kr/real-mysql/
