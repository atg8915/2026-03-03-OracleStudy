# 📘 Day 03 — 단일행 함수 / GROUP BY / JOIN

## 1. 내장 함수 분류

```
내장 함수
├── 집계 함수     → 컬럼(Column) 단위
├── 단일행 함수   → 행(Row) 단위
└── 사용자 정의   → CREATE FUNCTION
```

---

## 2. 단일행 함수

### 📝 문자 함수

| 함수 | 예시 | 결과 |
|------|------|------|
| `LENGTH(str)` | `LENGTH('홍길동')` | `3` |
| `LENGTHB(str)` | `LENGTHB('홍길동')` | `9` (한글 1자 = 3byte) |
| `UPPER(str)` | `UPPER('abc')` | `ABC` |
| `LOWER(str)` | `LOWER('ABC')` | `abc` |
| `INITCAP(str)` | `INITCAP('king')` | `King` |
| `REPLACE(str, 찾을문자, 바꿀문자)` | `REPLACE('Hello','l','x')` | `Hexxo` |
| `LPAD(str, 길이, 채울문자)` | `LPAD('KING',5,'#')` | `#KING` |
| `RPAD(str, 길이, 채울문자)` | `RPAD('KING',5,'#')` | `KING#` |
| `LTRIM(str)` | 왼쪽 공백/문자 제거 | |
| `RTRIM(str)` | 오른쪽 공백/문자 제거 | |
| `TRIM(str)` | 양쪽 공백 제거 | |
| `SUBSTR(str, 시작, 개수)` | `SUBSTR('Hello',2,3)` | `ell` |
| `INSTR(str, 찾을문자, 시작, N번째)` | 양수→indexOf / 음수→lastIndexOf | |
| `CONCAT(str1, str2)` | `CONCAT('Hello',' Java')` | `Hello Java` |
| `ASCII(str)` | 문자 → 숫자 | |
| `CHR(num)` | 숫자 → 문자 | |

---

### 🔢 숫자 함수

| 함수 | 설명 | 예시 | 결과 |
|------|------|------|------|
| `MOD(a, b)` | 나머지 (`%`) | `MOD(10,2)` | `0` |
| `CEIL(n)` | 올림 (총 페이지 계산에 활용) | `CEIL(9.1)` | `10` |
| `ROUND(n)` | 반올림 | `ROUND(9.5)` | `10` |
| `TRUNC(n)` | 버림 | `TRUNC(9.8)` | `9` |

---

### 📅 날짜 함수

| 함수 | 설명 |
|------|------|
| `SYSDATE` | 현재 시스템 날짜/시간 (`SYSDATE+1` = 내일) |
| `MONTHS_BETWEEN(현재, 과거)` | 두 날짜 사이의 개월 수 (근무 개월 계산) |
| `ADD_MONTHS(날짜, n)` | n개월 후 날짜 (적금·보험 만기 계산) |
| `LAST_DAY(날짜)` | 해당 월의 마지막 날 |
| `NEXT_DAY(날짜, '요일')` | 돌아오는 특정 요일 날짜 |

---

### 🔄 변환 함수

| 함수 | 설명 |
|------|------|
| `TO_CHAR(값, 형식)` | 숫자·날짜 → 문자열 |
| `TO_NUMBER(str)` | 문자 → 숫자 |
| `TO_DATE(str, 형식)` | 문자열 → DATE |

**날짜 형식 코드**

| 코드 | 의미 |
|------|------|
| `YYYY` / `YY` | 년도 |
| `MM` | 월 |
| `DD` | 일 |
| `HH` / `HH24` | 시 |
| `MI` | 분 |
| `SS` | 초 |
| `DAY` | 요일 |

---

### 🔧 기타 함수

**NVL** — NULL을 다른 값으로 대체
```sql
NVL(comm, 0)         -- NULL이면 0으로
NVL(bunji, ' ')      -- NULL이면 공백으로
SELECT NVL(MAX(no)+1, 1) FROM board  -- 자동 증가 번호
```

**DECODE** vs **CASE**

| 구분 | DECODE | CASE |
|------|--------|------|
| 조건 | 단순 값 비교 (`=`) | 모든 연산자 사용 가능 |
| 가독성 | 낮음 | 높음 |
| 활용 | 단순 치환 | 범위·복합 조건 |

```sql
-- CASE 형식
CASE
  WHEN 조건1 THEN 값1
  WHEN 조건2 THEN 값2
  ELSE 기본값
END AS 별칭
```

---

## 3. 집계 함수 요약

| 함수 | 설명 | 활용 예 |
|------|------|---------|
| `COUNT` | 행 개수 | 로그인 중복 확인, 검색 결과 수 |
| `SUM` | 합계 | 장바구니 총액 |
| `AVG` | 평균 | 구매 평균 금액 통계 |
| `MAX` / `MIN` | 최대·최솟값 | |
| `RANK()` | 순위 (동점 시 건너뜀) | 1 2 2 **4** |
| `DENSE_RANK()` | 순위 (동점 시 유지) | 1 2 2 **3** |

> ⚠️ 집계 함수 + 일반 컬럼 함께 사용 시 반드시 `GROUP BY` 필요

---

## 4. SELECT 전체 구조 정리

```sql
SELECT 컬럼 | *
FROM   테이블
WHERE  조건            -- 연산자 (true/false)
GROUP BY 컬럼          -- 집계 함수 그룹화
HAVING 집계조건        -- 집계 함수 조건
ORDER BY 컬럼 ASC|DESC -- 정렬
```

**실제 처리 순서** : `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY`
