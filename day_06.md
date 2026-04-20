# 📘 Day 06 — ROWNUM / 서브쿼리 심화 / VIEW

## 0. 핵심 문법 빠른 참조

| 구문 | 예시 | 결과 |
|------|------|------|
| `ROWNUM` 상위 N개 | `WHERE rownum <= 5` | 처음 5행 추출 |
| 페이징 (인라인뷰) | `FROM (SELECT * FROM emp ORDER BY sal DESC) WHERE rownum <= 5` | 정렬 후 상위 5개 |
| 스칼라 서브쿼리 | `SELECT ename, (SELECT dname FROM dept WHERE deptno = e.deptno) FROM emp e` | 컬럼 대신 사용 |
| 인라인 뷰 | `FROM (SELECT ename, sal FROM emp ORDER BY sal DESC)` | 테이블 대신 사용 |
| `CREATE VIEW` | `CREATE VIEW v_emp AS SELECT * FROM emp WHERE deptno = 10` | 뷰 생성 |
| `CREATE OR REPLACE VIEW` | `CREATE OR REPLACE VIEW v_emp AS SELECT ...` | 뷰 수정 |
| `DROP VIEW` | `DROP VIEW v_emp` | 뷰 삭제 |
| 뷰 내용 확인 | `SELECT text FROM user_views WHERE view_name = 'V_EMP'` | 뷰 SQL 조회 |
| `WITH READ ONLY` | `CREATE VIEW v_emp AS SELECT ... WITH READ ONLY` | 읽기 전용 뷰 |
| `WITH CHECK OPTION` | `CREATE VIEW v_emp AS SELECT ... WITH CHECK OPTION` | DML 허용 뷰 |

---

## 1. ROWNUM

> 오라클이 조회된 행에 **자동으로 순번을 부여**하는 가상 컬럼
> 모든 테이블에 적용되며 별도 생성 불필요

### 사용처
- 인기 순위 Top N
- 게시판 페이징
- 이전/다음 글 처리

### 주의점

```sql
-- ❌ 이렇게 하면 안됨 — 정렬 전에 rownum이 먼저 붙어서 의미없음
SELECT empno, ename, sal, rownum
FROM emp
WHERE rownum <= 5
ORDER BY sal DESC;

-- ✅ 인라인뷰로 먼저 정렬한 뒤 rownum 적용
SELECT empno, ename, sal, rownum
FROM (SELECT * FROM emp ORDER BY sal DESC)
WHERE rownum <= 5;
```

> ⚠️ ROWNUM은 **처음부터만 자를 수 있음** — 중간 범위 (`rownum BETWEEN 6 AND 10`) 는 인라인뷰 중첩 필요

---

## 2. 서브쿼리 심화

### 서브쿼리 위치별 종류 총정리

| 위치 | 종류 | 역할 | 특징 |
|------|------|------|------|
| `WHERE` 뒤 | 중첩 서브쿼리 | 조건값 | 단일행 / 다중행 / 다중컬럼 |
| `SELECT` 뒤 | 스칼라 서브쿼리 | 컬럼 대체 | 반드시 결과 1개 |
| `FROM` 뒤 | 인라인 뷰 | 테이블 대체 | 페이징·보안에 활용 |

### WHERE 뒤 — 연산자 정리

| 연산자 | 결과 개수 | 설명 |
|--------|-----------|------|
| `=`, `>`, `<` 등 | 1개 | 단일행 서브쿼리 |
| `IN`, `NOT IN` | 여러 개 | 집합 비교 |
| `ANY`, `SOME` | 여러 개 | 최솟값 초과 or 최댓값 미만 |
| `ALL` | 여러 개 | 최댓값 초과 or 최솟값 미만 |
| `EXISTS` | 존재 여부 | 참/거짓만 반환, 속도 빠름 |

### 스칼라 서브쿼리 (SELECT 뒤)

```sql
-- JOIN 대신 스칼라 서브쿼리로 부서명 출력
SELECT ename,
       (SELECT dname FROM dept WHERE deptno = e.deptno) AS dname
FROM emp e;
```

> - 반드시 결과값 **1행 1컬럼**만 반환해야 함
> - 집계함수와 함께 자주 사용
> - `CASE`와 조합해서 조건별 계산에 활용

### 인라인 뷰 (FROM 뒤)

```sql
-- 급여 상위 5명 페이징
SELECT empno, ename, sal
FROM (SELECT * FROM emp ORDER BY sal DESC)
WHERE rownum <= 5;

-- 인라인뷰 안에 없는 컬럼은 사용 불가
SELECT empno, ename, job, sal
FROM (SELECT ename, sal FROM emp);  -- ❌ empno, job은 사용 불가
```

> - 저장되지 않아 **보안**에 유리
> - **한 번만 사용** 가능 (재사용 필요하면 VIEW로 저장)

---

## 3. VIEW

> 하나 이상의 테이블을 연결해 만든 **가상 테이블**
> 실제 데이터를 저장하지 않고 **SELECT 문장만 저장**

### 장점

| 장점 | 설명 |
|------|------|
| 편리성 | 복잡한 SQL을 간결하게 사용 가능 |
| 재사용성 | 한 번 만들면 여러 곳에서 재사용 |
| 보안성 | 실제 데이터 미노출, 필요한 컬럼만 공개 |
| 단순화 | JOIN·서브쿼리가 복잡할 때 VIEW로 대체 |

### VIEW 종류

| 종류 | 설명 |
|------|------|
| 단순 뷰 | 테이블 1개 참조 — DML 가능 |
| 복합 뷰 | 테이블 2개 이상 (JOIN·서브쿼리) — DML 불가 |
| 인라인 뷰 | `FROM` 뒤 임시 테이블 — 재사용 불가 |

### 생성 / 수정 / 삭제

```sql
-- 생성
CREATE VIEW v_emp AS
SELECT empno, ename, dname
FROM emp JOIN dept ON emp.deptno = dept.deptno;

-- 수정 (DROP 없이 덮어쓰기)
CREATE OR REPLACE VIEW v_emp AS
SELECT empno, ename, sal, dname
FROM emp JOIN dept ON emp.deptno = dept.deptno;

-- 삭제
DROP VIEW v_emp;

-- 저장된 VIEW SQL 확인
SELECT text FROM user_views WHERE view_name = 'V_EMP';  -- 반드시 대문자
```

### VIEW 옵션

```sql
-- DML 허용
CREATE VIEW v_emp AS SELECT ... WITH CHECK OPTION;

-- 읽기 전용 (DML 불가)
CREATE VIEW v_emp AS SELECT ... WITH READ ONLY;
```

> ⚠️ VIEW에서 DML(INSERT·UPDATE·DELETE)을 해도 **실제 변경은 원본 테이블**에 반영됨

---

## 4. 스칼라·인라인뷰 vs VIEW 비교

| 구분 | 스칼라 서브쿼리 | 인라인 뷰 | VIEW |
|------|----------------|-----------|------|
| 위치 | SELECT 뒤 | FROM 뒤 | 독립 객체 |
| 저장 여부 | ❌ | ❌ | ✅ |
| 재사용 | ❌ | ❌ | ✅ |
| 보안 | ✅ | ✅ | ✅ |
| 주요 용도 | 컬럼 1개 계산 | 페이징·임시 테이블 | 복잡한 쿼리 재사용 |
