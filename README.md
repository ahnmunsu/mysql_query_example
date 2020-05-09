# Useful SQL query examples

## 목차
- **[OUTER JOIN 주의 사항](#OUTER-JOIN-주의-사항)**
- **[ANTI JOIN](#ANTI-JOIN)**
- **[레코드를 칼럼으로 변환](#레코드를-칼럼으로-변환)**
- **[하나의 컬럼을 여러 컬럼으로 분리](#하나의-컬럼을-여러-컬럼으로-분리)**

---
## OUTER JOIN 주의 사항
OUTER JOIN에서 OUTER로 조인되는 테이블의 칼럼에 대한 조건은 모두 ON 절에 명시해야 한다.
```sql
SELECT * FROM employees e
  LEFT JOIN dept_manager mgr ON mgr.emp_no=e.emp_no AND mgr.dept_no='d001';
```
## ANTI JOIN
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

## 출처
개발자와 DBA를 위한 Real MySQL / 이성욱 / 위키북스
