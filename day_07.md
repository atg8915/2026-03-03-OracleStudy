# 📘 Day 07 — INDEX / SQL 전체 문법 총정리

## 0. 핵심 문법 빠른 참조

| 구문 | 예시 | 결과 |
|------|------|------|
| 인덱스 생성 | `CREATE INDEX idx_emp_name ON emp(ename)` | 단일 인덱스 생성 |
| 복합 인덱스 | `CREATE INDEX idx_food ON food(type, score DESC)` | 여러 컬럼 인덱스 |
| 함수 기반 인덱스 | `CREATE INDEX idx_addr ON food(SUBSTR(address,1,2))` | 함수 결과에 인덱스 |
| 인덱스 삭제 | `DROP INDEX idx_emp_name` | 인덱스 삭제 |
| 인덱스 리빌드 | `ALTER INDEX idx_emp_name REBUILD` | 인덱스 재구성 |
| 힌트 오름차순 | `SELECT /*+ INDEX_ASC(emp pk_emp) */ * FROM emp` | 인덱스 오름차순 정렬 |
| 힌트 내림차순 | `SELECT /*+ INDEX_DESC(emp pk_emp) */ * FROM emp` | 인덱스 내림차순 정렬 |
| 인덱스 목록 확인 | `SELECT * FROM user_indexes` | 생성된 인덱스 목록 |
| 시퀀스 생성 | `CREATE SEQUENCE seq_name START WITH 1 INCREMENT BY 1 NOCACHE NOCYCLE` | 시퀀스 생성 |
| 시퀀스 삭제 | `DROP SEQUENCE seq_name` | 시퀀스 삭제 |
| 컬럼 추가 | `ALTER TABLE emp ADD phone VARCHAR2(20)` | 컬럼 추가 |
| 컬럼 수정 | `ALTER TABLE emp MODIFY phone VARCHAR2(30)` | 컬럼 수정 |
| 컬럼 삭제 | `ALTER TABLE emp DROP COLUMN phone` | 컬럼 삭제 |
| 컬럼명 변경 | `ALTER TABLE emp RENAME COLUMN phone TO tel` | 컬럼명 변경 |

---

## 1. INDEX 개념

> 데이터를 **빠르게 찾기 위한 자료구조** — 책의 색인(목차)과 같은 역할
> 인덱스가 없으면 **Full Scan** (전체 탐색), 인덱스가 있으면 **필요한 위치만 탐색**

### 자동 생성되는 인덱스
- `PRIMARY KEY` 설정 시 자동 생성
- `UNIQUE` 설정 시 자동 생성

### 인덱스 생성이 유리한 경우 ✅
| 상황 | 예시 |
|------|------|
| 데이터가 많은 테이블 (10만 건 이상) | 회원, 주문, 상품 테이블 |
| `WHERE`에서 자주 검색되는 컬럼 | 이름, 지역, 카테고리 |
| `JOIN`에서 주로 사용되는 컬럼 | `deptno`, 곡명, 가수명 |
| `ORDER BY`, `GROUP BY` 컬럼 | 평점, 날짜 |
| `NULL`값을 포함하는 컬럼 | `comm`, `bunji` |

### 인덱스를 쓰면 안 되는 경우 ❌
| 상황 | 이유 |
|------|------|
| 데이터가 적은 테이블 | Full Scan이 더 빠름 |
| `INSERT/UPDATE/DELETE`가 많은 곳 | 변경 시마다 인덱스 재구성 → 속도 저하 |
| 값이 단조로운 컬럼 (`남자/여자`, `Y/N`) | 구별 효과 없음 |
| `WHERE UPPER(name) = ''` 등 함수 사용 | 인덱스 미적용 |
| `LIKE '%검색어'` — `%`가 앞에 오는 경우 | 인덱스 미적용 |
| `WHERE score + 1 > 5` 등 계산식 | 인덱스 미적용 |
| 조건 없이 전체 조회 | 인덱스 의미 없음 |

---

## 2. INDEX 종류

| 종류 | 설명 | 사용 시기 |
|------|------|-----------|
| B-Tree Index | 기본 인덱스, 정렬된 트리 구조 | 일반 검색·범위·정렬 |
| Bitmap Index | 비트로 저장, `AND/OR` 연산 최적화 | 값 종류가 적은 컬럼 (Y/N) — 데이터웨어하우스·빅데이터 |
| Function Based Index | 함수 결과에 인덱스 적용 | `SUBSTR`, `NVL` 등 함수 사용 컬럼 |
| 복합 Index | 여러 컬럼을 묶어서 인덱스 | 여러 조건을 동시에 검색 |

### B-Tree 구조
```
        [4]
       /   \
     [2]   [6]
    /  \   /  \
  [1] [3] [5] [7]
```
> - 루트에서 시작해 값을 비교하며 절반씩 범위를 좁혀가며 탐색
> - 100만 건도 약 20번 비교면 탐색 가능
> - 정확한 값(`=`), 범위(`BETWEEN`, `<`, `>`), 정렬에 모두 유효

### ROWID
> 각 ROW의 **실제 물리적 메모리 주소값** — 인덱스가 최종적으로 ROWID를 찾아 데이터 접근
```
AAAS+K AAH AAAANH AAA
  │     │     │    └─ 행 번호
  │     │     └─ 블록 번호
  │     └─ 파일 번호
  └─ 테이블 번호
```

---

## 3. INDEX 생성 / 삭제 / 관리

```sql
-- 단일 인덱스
CREATE INDEX idx_emp_ename ON emp(ename);

-- 복합 인덱스 (정렬 방향 지정 가능)
CREATE INDEX idx_food ON food(type, score DESC);

-- 함수 기반 인덱스
CREATE INDEX idx_addr ON food(SUBSTR(address, 1, 2), score DESC);

-- 삭제
DROP INDEX idx_emp_ename;

-- 리빌드 (DML이 많아 인덱스가 느려졌을 때)
ALTER INDEX idx_emp_ename REBUILD;

-- 힌트로 인덱스 정렬 지정
SELECT /*+ INDEX_ASC(emp pk_emp) */ * FROM emp;
SELECT /*+ INDEX_DESC(emp pk_emp) */ * FROM emp;
```

### 인덱스 적용 조건
| 적용 ✅ | 미적용 ❌ |
|---------|----------|
| `WHERE name = '홍길동'` | `WHERE UPPER(name) = 'KIM'` |
| `WHERE score BETWEEN 70 AND 90` | `WHERE score + 1 > 5` |
| `ORDER BY score DESC` (인덱스 방향 일치) | `ORDER BY name \|\| type` |
| `JOIN ON emp.deptno = dept.deptno` | `LIKE '%마포%'` |

---

## 4. SQL 전체 문법 총정리

### DML
```sql
-- SELECT
SELECT * | 컬럼
FROM 테이블 | 뷰 | (SELECT ~)
[WHERE 조건]
[GROUP BY 컬럼]
[HAVING 집계조건]
[ORDER BY 컬럼 ASC|DESC]

-- INSERT
INSERT INTO 테이블 VALUES (값, 값, ...);              -- 전체 컬럼
INSERT INTO 테이블 (컬럼, ...) VALUES (값, ...);      -- 지정 컬럼

-- UPDATE
UPDATE 테이블 SET 컬럼=값, 컬럼=값
[WHERE 조건];   -- ⚠️ WHERE 없으면 전체 수정

-- DELETE
DELETE FROM 테이블
[WHERE 조건];   -- ⚠️ WHERE 없으면 전체 삭제
```

### DDL
```sql
-- 테이블 생성
CREATE TABLE 테이블 (
    컬럼 데이터형 [제약조건],
    [CONSTRAINT 제약조건명 PRIMARY KEY(컬럼)],
    [CONSTRAINT 제약조건명 FOREIGN KEY(컬럼) REFERENCES 테이블(컬럼)]
);

-- 테이블 복사 생성
CREATE TABLE 새테이블 AS SELECT * FROM 원본테이블;

-- 뷰 생성/수정
CREATE [OR REPLACE] VIEW 뷰이름 AS SELECT ~;

-- 시퀀스 생성
CREATE SEQUENCE seq_name
    START WITH 1
    INCREMENT BY 1
    NOCACHE
    NOCYCLE;

-- 인덱스 생성
CREATE INDEX idx_name ON 테이블(컬럼 ASC|DESC);

-- ALTER
ALTER TABLE 테이블 ADD 컬럼 데이터형;
ALTER TABLE 테이블 MODIFY 컬럼 데이터형;
ALTER TABLE 테이블 DROP COLUMN 컬럼;
ALTER TABLE 테이블 RENAME COLUMN 기존명 TO 새이름;

-- DROP
DROP TABLE 테이블;
DROP INDEX idx_name;
DROP SEQUENCE seq_name;
DROP VIEW 뷰이름;

-- 기타
RENAME 기존테이블 TO 새테이블;
TRUNCATE TABLE 테이블;
```

### DCL / TCL
```sql
-- DCL
GRANT CREATE VIEW TO hr;       -- 권한 부여
REVOKE CREATE VIEW FROM hr;    -- 권한 해제

-- TCL
COMMIT;                        -- 변경사항 확정 저장
ROLLBACK;                      -- 변경사항 전체 취소
SAVEPOINT sp1;                 -- 저장 지점 설정
ROLLBACK TO sp1;               -- 해당 지점으로 취소
```

### 데이터형 / 제약조건 한눈에 보기
| 데이터형 | 설명 |
|----------|------|
| `VARCHAR2(n)` | 가변 문자열 — 가장 많이 사용 |
| `CHAR(n)` | 고정 문자열 — 성별, Y/N |
| `CLOB` | 대용량 문자 — 게시판 본문 |
| `NUMBER(n,d)` | 숫자 — 정수/실수 |
| `DATE` | 날짜+시간 |
| `TIMESTAMP` | 정밀 시간 |

| 제약조건 | 설명 |
|----------|------|
| `NOT NULL` | NULL 불가 |
| `UNIQUE` | 중복 불가, NULL 허용 |
| `PRIMARY KEY` | NOT NULL + UNIQUE |
| `FOREIGN KEY` | 다른 테이블 PK 참조 |
| `CHECK` | 지정값만 허용 |
| `DEFAULT` | 기본값 설정 |
