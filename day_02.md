# 📘 Day 02 — WHERE / ORDER BY / 내장 함수

## 1. SQL 언어 분류

| 분류 | 이름 | 주요 명령어 |
|------|------|------------|
| DML | 데이터 조작어 | SELECT, INSERT, UPDATE, DELETE |
| DDL | 데이터 정의어 | CREATE, DROP, ALTER, TRUNCATE, RENAME |
| DCL | 권한 제어어 | GRANT(부여), REVOKE(해제) |
| TCL | 트랜잭션 제어 | COMMIT, ROLLBACK, SAVEPOINT |

### DDL 주요 객체
| 객체 | 설명 |
|------|------|
| TABLE | 데이터 저장 공간 |
| VIEW | 가상 테이블 — 보안 / 쿼리 단순화 |
| SEQUENCE | 자동 증가 번호 (PK에 주로 사용) |
| INDEX | 검색·정렬 속도 최적화 |
| FUNCTION | 사용자 정의 함수 |
| PROCEDURE | 사용자 정의 기능 |
| TRIGGER | 자동화 처리 |

---

## 2. SELECT 처리 순서

```sql
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```

---

## 3. WHERE 조건 연산자

### 산술 연산자
| 연산자 | 설명 |
|--------|------|
| `+`, `-`, `*`, `/` | 사칙연산 (나머지는 `MOD()` 함수 사용) |
| `DUAL` | 연습용 가상 테이블 |

> ⚠️ 0으로 나누면 오류 발생 / 정수 ÷ 정수 = 실수

### 비교 연산자
| 연산자 | 설명 |
|--------|------|
| `=` | 같다 |
| `<>` | 같지 않다 (권장) |
| `<`, `>` | 작다, 크다 |

> 숫자, 문자열, 날짜(`'YY/MM/DD'`) 모두 비교 가능

### 논리 연산자
| 연산자 | 설명 |
|--------|------|
| `AND` | 그리고 (`&&` 사용 불가) |
| `OR` | 또는 (`\|\|` 는 문자열 결합) |
| `NOT` | 부정 |

### 범위 / 목록 / NULL
| 연산자 | 형식 | 설명 |
|--------|------|------|
| `BETWEEN` | `컬럼 BETWEEN 값1 AND 값2` | 값1 이상 값2 이하 (경계 포함) |
| `IN` | `컬럼 IN (값1, 값2, ...)` | 목록 중 하나와 일치 |
| `IS NULL` | `컬럼 IS NULL` | NULL 여부 확인 |
| `IS NOT NULL` | `컬럼 IS NOT NULL` | NULL이 아닌 값 |

### LIKE — 패턴 검색
| 패턴 | 예시 | 설명 |
|------|------|------|
| `%검색어%` | `LIKE '%A%'` | 포함 |
| `검색어%` | `LIKE 'A%'` | 시작 |
| `%검색어` | `LIKE '%A'` | 끝 |
| `__C__` | 5글자 중 3번째가 C | `_` = 한 글자 |
| `REGEXP_LIKE()` | `REGEXP_LIKE(ename, 'A\|B\|C')` | 정규식 검색 |

---

## 4. ORDER BY — 정렬

```sql
SELECT 컬럼 FROM 테이블 ORDER BY 컬럼 [ASC | DESC];
```

| 옵션 | 설명 |
|------|------|
| `ASC` | 오름차순 (기본값) |
| `DESC` | 내림차순 |

---

## 5. 내장 함수

### 단일행 함수 — ROW 단위 처리
- 문자 함수 / 숫자 함수 / 날짜 함수 / 변환 함수

### 집계 함수 — Column 단위 처리

| 함수 | 설명 |
|------|------|
| `SUM` | 총합 |
| `AVG` | 평균 |
| `MIN` | 최솟값 |
| `MAX` | 최댓값 |
| `COUNT` | 행 개수 (중복 확인, 검색 결과 수) |
| `RANK()` | 순위 (동점 시 다음 순위 건너뜀 — 1 2 2 4) |
| `DENSE_RANK()` | 순위 (동점 시 순위 유지 — 1 2 2 3) |

> ⚠️ 집계 함수와 일반 컬럼을 함께 쓰려면 반드시 `GROUP BY` 필요

---

## 6. 학습 로드맵

```
WHERE / ORDER BY
→ 내장 함수 (단일 / 집계)
→ GROUP BY / JOIN / SubQuery
→ DDL (CREATE, DROP, ALTER) + INSERT / UPDATE / DELETE
→ VIEW / INDEX / SEQUENCE / PL/SQL
→ DB 설계 (ER-MODEL) / 정규화
→ 트랜잭션 / 보안 / 백업·복원
```
