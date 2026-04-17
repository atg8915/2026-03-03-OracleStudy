# 📘 Day 05 — DDL / DML

## 0. 핵심 문법 빠른 참조

| 구문 | 예시 | 결과 |
|------|------|------|
| `CREATE TABLE` | `CREATE TABLE member (id VARCHAR2(20) PRIMARY KEY)` | 테이블 생성 |
| `ALTER ... ADD` | `ALTER TABLE member ADD age NUMBER` | 컬럼 추가 |
| `ALTER ... MODIFY` | `ALTER TABLE member MODIFY age NUMBER(3)` | 컬럼 수정 |
| `ALTER ... DROP` | `ALTER TABLE member DROP COLUMN age` | 컬럼 삭제 |
| `DROP TABLE` | `DROP TABLE member` | 테이블 전체 삭제 |
| `TRUNCATE` | `TRUNCATE TABLE member` | 데이터만 삭제, 구조 유지 |
| `RENAME` | `RENAME member TO user_info` | 테이블명 변경 |
| `INSERT` 전체 | `INSERT INTO member VALUES (값, 값, ...)` | 전체 컬럼에 값 추가 |
| `INSERT` 지정 | `INSERT INTO member (id, name) VALUES ('abc', '홍길동')` | 지정 컬럼만 추가 |
| `UPDATE` 전체 | `UPDATE member SET name = '홍길동'` | 전체 행 수정 ⚠️ |
| `UPDATE` 조건 | `UPDATE member SET name = '홍길동' WHERE id = 'abc'` | 조건 행만 수정 |
| `DELETE` 전체 | `DELETE FROM member` | 전체 행 삭제 ⚠️ |
| `DELETE` 조건 | `DELETE FROM member WHERE id = 'abc'` | 조건 행만 삭제 |
| `NOT NULL` | `name VARCHAR2(20) NOT NULL` | NULL 입력 불가 |
| `UNIQUE` | `email VARCHAR2(50) UNIQUE` | 중복 불가, NULL 허용 |
| `PRIMARY KEY` | `CONSTRAINT pk PRIMARY KEY(no)` | NOT NULL + UNIQUE |
| `FOREIGN KEY` | `CONSTRAINT fk FOREIGN KEY(bno) REFERENCES board(no)` | 다른 테이블 PK 참조 |
| `CHECK` | `CHECK(sex IN ('남자', '여자'))` | 지정값만 허용 |
| `DEFAULT` | `regdate DATE DEFAULT SYSDATE` | 값 미입력 시 기본값 |

---

## 1. DDL 개요

> 데이터베이스 **구조(객체)를 정의**하는 언어 — 실행 즉시 **AutoCommit**

| 명령어 | 설명 |
|--------|------|
| `CREATE` | 객체 생성 (TABLE, SEQUENCE, INDEX 등) |
| `ALTER` | 객체 수정 (ADD / MODIFY / DROP) |
| `DROP` | 객체 전체 삭제 |
| `TRUNCATE` | 데이터만 삭제, 테이블 구조는 유지 |
| `RENAME` | 객체 이름 변경 |

---

## 2. 테이블 명명 규칙

| 규칙 | 설명 |
|------|------|
| 문자로 시작 | 숫자·특수문자로 시작 불가 |
| 대소문자 구분 없음 | 오라클 내부에서 대문자로 저장 |
| 최대 30byte | 한글 최대 10글자 |
| 유일한 이름 | 같은 DB 내 중복 불가 |
| 키워드 사용 금지 | `SELECT`, `ORDER` 등 예약어 불가 |
| 특수문자 | `_`, `$` 만 허용 (예: `file_name`) |

---

## 3. 오라클 데이터형

| 분류 | 데이터형 | 설명 | Java 매핑 |
|------|----------|------|-----------|
| 문자 | `CHAR(n)` | 고정 바이트 (1~2000byte) — 성별, 등급 등 | `String` |
| 문자 | `VARCHAR2(n)` | 가변 바이트 (1~4000byte) — 가장 많이 사용 | `String` |
| 문자 | `CLOB` | 가변 최대 4GB — 게시판 본문, 자기소개 | `String` |
| 숫자 | `NUMBER` | 기본 (8,2) — 정수 8자리, 소수점 2자리 | `int`, `double` |
| 숫자 | `NUMBER(n)` | 정수 n자리 | `int` |
| 숫자 | `NUMBER(n,d)` | 전체 n자리 중 소수점 d자리 | `double` |
| 날짜 | `DATE` | 날짜+시간 — `SYSDATE`로 현재값 입력 | `java.util.Date` |
| 날짜 | `TIMESTAMP` | DATE보다 정밀한 시간 (기록 경주 등) | `java.util.Date` |
| 기타 | `BLOB` | 이진 데이터 최대 4GB (이미지, 동영상) | `InputStream` |
| 기타 | `BFILE` | 외부 파일 참조 (4GB) | `InputStream` |

> 💡 `CHAR` vs `VARCHAR2` — 고정 크기가 필요한 값(성별, Y/N)은 `CHAR`, 나머지는 `VARCHAR2`

---

## 4. 제약조건

```sql
CREATE TABLE reply (
    no      NUMBER,
    bno     NUMBER,
    id      VARCHAR2(20)  CONSTRAINT reply_id_nn   NOT NULL,
    name    VARCHAR2(50)  CONSTRAINT reply_name_nn  NOT NULL,
    sex     CHAR(6),
    msg     CLOB          CONSTRAINT reply_msg_nn   NOT NULL,
    regdate DATE          DEFAULT SYSDATE,
    CONSTRAINT reply_no_pk  PRIMARY KEY (no),
    CONSTRAINT reply_bno_fk FOREIGN KEY (bno)
        REFERENCES board(no) ON DELETE CASCADE,
    CONSTRAINT reply_sex_ck CHECK (sex IN ('남자', '여자'))
);
```

| 제약조건 | 설명 | 문법 |
|----------|------|------|
| `NOT NULL` | NULL 입력 불가 | `컬럼 데이터형 NOT NULL` |
| `UNIQUE` | 중복 불가, NULL 허용 (후보키 — 이메일, 전화번호) | `컬럼 데이터형 UNIQUE` |
| `PRIMARY KEY` | NOT NULL + UNIQUE — 테이블당 1개 이상 필수 | `CONSTRAINT pk명 PRIMARY KEY(컬럼)` |
| `FOREIGN KEY` | 다른 테이블 PK 참조 — 테이블 간 관계 연결 | `CONSTRAINT fk명 FOREIGN KEY(컬럼) REFERENCES 테이블(컬럼)` |
| `CHECK` | 지정된 값만 허용 | `CHECK(컬럼 IN ('값1', '값2'))` |
| `DEFAULT` | 값 미입력 시 기본값 자동 입력 | `컬럼 데이터형 DEFAULT 값` |

> ⚠️ `ON DELETE CASCADE` : 부모 행 삭제 시 참조하는 자식 행도 함께 삭제

---

## 5. DML — INSERT / UPDATE / DELETE

> 데이터를 **조작**하는 언어 — 반드시 **COMMIT** 으로 확정해야 저장됨

### INSERT
```sql
-- 전체 컬럼 입력 (DEFAULT 무시됨 주의)
INSERT INTO board VALUES (값, 값, ...);

-- 지정 컬럼만 입력 (DEFAULT 적용됨, 나머지 NULL)
INSERT INTO board (no, name, subject, content, pwd)
VALUES (값, 값, 값, 값, 값);
```

### UPDATE
```sql
-- ⚠️ WHERE 없으면 전체 행 수정
UPDATE board SET
    subject = '제목수정',
    content = '내용수정'
WHERE no = 1;
```

### DELETE
```sql
-- ⚠️ WHERE 없으면 전체 행 삭제
DELETE FROM board WHERE no = 1;
```

---

## 6. TRUNCATE vs DELETE vs DROP

| 구분 | 데이터 삭제 | 구조 유지 | 복구 가능 | AutoCommit |
|------|-------------|-----------|-----------|------------|
| `DELETE` | 조건 지정 가능 | ✅ | ✅ (ROLLBACK) | ❌ |
| `TRUNCATE` | 전체만 가능 | ✅ | ❌ | ✅ |
| `DROP` | 전체 | ❌ | ❌ | ✅ |

> 💡 단순히 데이터 전체를 비울 땐 `DELETE`보다 `TRUNCATE`가 빠르지만, 복구가 불가능하므로 주의
