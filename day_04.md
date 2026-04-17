# 📘 Day 04 — JOIN / SubQuery

## 1. JOIN 개요

> 정규화로 나뉜 테이블을 연결해 필요한 데이터를 추출하는 과정
> **SELECT 문에서만 사용 가능**

```
JOIN 종류
├── INNER JOIN   → 교집합 (가장 많이 사용, NULL 처리 불가)
├── OUTER JOIN   → INNER JOIN + NULL 처리
│   ├── LEFT OUTER JOIN
│   ├── RIGHT OUTER JOIN
│   └── FULL OUTER JOIN
├── SELF JOIN    → 같은 테이블끼리 조인
├── NATURAL JOIN → 동일 컬럼명 자동 연결
└── JOIN ~ USING → 동일 컬럼명 지정 연결
```

---

## 2. INNER JOIN

> 두 테이블에서 **공통으로 존재하는 값**만 조회 (교집합)

| 구분 | 설명 |
|------|------|
| EQUI JOIN | `=` 연산자로 연결 |
| NON-EQUI JOIN | `BETWEEN`, 비교 연산자로 범위 연결 |

```sql
-- 오라클 방식
SELECT a.col, b.col
FROM A a, B b
WHERE a.col = b.col;

-- ANSI 방식 (권장)
SELECT a.col, b.col
FROM A a JOIN B b
ON a.col = b.col;

-- NATURAL JOIN (컬럼명 동일할 때)
SELECT col FROM A NATURAL JOIN B;

-- JOIN ~ USING (컬럼명 동일할 때)
SELECT col FROM A JOIN B USING(공통컬럼);
```

> ⚠️ 컬럼명이 동일하면 반드시 `테이블.컬럼` 또는 `별칭.컬럼`으로 구분
> ⚠️ NATURAL JOIN / JOIN~USING 은 NON-EQUI JOIN에서 사용 불가

---

## 3. OUTER JOIN

> **한쪽 테이블 기준**으로 NULL인 데이터도 함께 출력

| 종류 | 설명 | 오라클 문법 | ANSI 문법 |
|------|------|-------------|-----------|
| LEFT OUTER | 왼쪽 테이블 기준 전체 출력 | `WHERE A.col = B.col(+)` | `A LEFT OUTER JOIN B ON ...` |
| RIGHT OUTER | 오른쪽 테이블 기준 전체 출력 | `WHERE A.col(+) = B.col` | `A RIGHT OUTER JOIN B ON ...` |
| FULL OUTER | 양쪽 전체 출력 | 오라클 문법 없음 | `A FULL OUTER JOIN B ON ...` |

---

## 4. SELF JOIN

> **같은 테이블**을 별칭으로 구분해서 조인
> 계층 구조 데이터에 활용 (사원-상사, 게시물-댓글 등)

```sql
SELECT e1.ename "본인", e2.ename "상사"
FROM emp e1, emp e2
WHERE e1.mgr = e2.empno(+);  -- (+) : 상사가 없는 경우도 출력
```

---

## 5. 서브쿼리(SubQuery) 개요

> **SQL 안에 또 다른 SQL**을 넣어 처리
> JOIN이 테이블+테이블이라면, 서브쿼리는 SQL+SQL

```
서브쿼리 위치별 종류
├── WHERE 뒤  → 일반 서브쿼리  (조건값으로 사용)
├── SELECT 뒤 → 스칼라 서브쿼리 (JOIN 대체, 컬럼으로 사용)
└── FROM 뒤   → 인라인 뷰      (테이블 대체, 페이징에 활용)
```

---

## 6. 서브쿼리 종류

### 단일행 서브쿼리
> 결과가 **1개**인 경우 → 비교 연산자 사용 (`=`, `!=`, `<`, `>`)

```sql
-- 평균 급여보다 적게 받는 사원
SELECT * FROM emp
WHERE sal < (SELECT AVG(sal) FROM emp);

-- 중첩 서브쿼리 (최고 급여자와 같은 부서 사원)
SELECT * FROM emp
WHERE deptno = (SELECT deptno FROM emp
                WHERE sal = (SELECT MAX(sal) FROM emp));
```

### 다중행 서브쿼리
> 결과가 **여러 개**인 경우 → `IN`, `ANY`, `ALL` 사용

| 연산자 | 설명 | 예시 |
|--------|------|------|
| `IN` | 목록 중 일치하는 값 | `IN (10, 20, 30)` |
| `ANY` / `SOME` | 최솟값 초과 or 최댓값 미만 | `> ANY` → 최솟값 초과 |
| `ALL` | 최댓값 초과 or 최솟값 미만 | `> ALL` → 최댓값 초과 |

```sql
-- IN 예시
SELECT * FROM emp
WHERE deptno IN (SELECT DISTINCT deptno FROM emp);

-- ANY 예시 (deptno > 10)
SELECT * FROM emp
WHERE deptno > ANY (SELECT DISTINCT deptno FROM emp);
```

### 인라인 뷰 (FROM 뒤)
> 서브쿼리 결과를 **테이블처럼** 사용 → 페이징 처리에 활용

```sql
-- 급여 상위 5명
SELECT empno, ename, sal
FROM (SELECT * FROM emp ORDER BY sal DESC)
WHERE rownum <= 5;
```

### EXISTS
> 서브쿼리 결과가 **존재하는지 여부**만 체크 → IN보다 속도 빠름

```sql
-- 주문 이력이 있는 고객
SELECT name, address FROM customer cs
WHERE EXISTS (
    SELECT * FROM orders os
    WHERE cs.custid = os.custid
);
```

---

## 7. JOIN vs 서브쿼리

| 구분 | JOIN | 서브쿼리 |
|------|------|----------|
| 목적 | 테이블 + 테이블 → 컬럼 확장 | SQL + SQL → 결과값 활용 |
| 사용 위치 | SELECT 전용 | DML 전체 (SELECT/INSERT/UPDATE/DELETE) |
| NULL 처리 | OUTER JOIN 필요 | 서브쿼리 내 처리 가능 |
| 네트워크 | 여러 번 통신 | 한 번에 처리 → 효율적 |

---

## 8. 핵심 정리

```sql
-- JOIN 기본 패턴
SELECT a.col, b.col
FROM A a JOIN B b ON a.key = b.key
WHERE 추가조건;

-- 서브쿼리 기본 패턴
SELECT * FROM 테이블
WHERE col = (SELECT col FROM 테이블 WHERE 조건);

-- 인라인 뷰 + ROWNUM (페이징)
SELECT * FROM (SELECT * FROM 테이블 ORDER BY col DESC)
WHERE rownum <= N;
```
