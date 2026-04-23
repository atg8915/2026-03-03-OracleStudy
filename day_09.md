# 📘 Day 09 — FUNCTION / PROCEDURE / TRIGGER

## 0. 핵심 문법 빠른 참조

| 구문 | 예시 | 설명 |
|------|------|------|
| FUNCTION 생성 | `CREATE OR REPLACE FUNCTION func(p IN NUMBER) RETURN NUMBER IS BEGIN RETURN 값; END;` | 리턴값 있음 |
| PROCEDURE 생성 | `CREATE OR REPLACE PROCEDURE pro(p IN NUMBER) IS BEGIN DML; END;` | 리턴값 없음 |
| OUT 매개변수 | `pName OUT student.name%TYPE` | 값을 밖으로 반환 |
| PROCEDURE 호출 | `CALL pro_name(값, ...)` | 프로시저 실행 |
| TRIGGER 생성 | `CREATE OR REPLACE TRIGGER tri AFTER INSERT ON 테이블 FOR EACH ROW BEGIN ... END;` | 이벤트 자동 실행 |
| `:NEW.컬럼` | `:NEW.수량` | 트리거에서 새로 입력된 값 |
| `:OLD.컬럼` | `:OLD.수량` | 트리거에서 변경 전 값 |
| 삭제 | `DROP FUNCTION func_name` / `DROP PROCEDURE pro_name` / `DROP TRIGGER tri_name` | 삭제 |

---

## 1. FUNCTION vs PROCEDURE vs TRIGGER

| 구분 | FUNCTION | PROCEDURE | TRIGGER |
|------|----------|-----------|---------|
| 리턴값 | ✅ 있음 | ❌ 없음 | ❌ 없음 |
| 주요 용도 | ROW 단위 계산, 변환, 조회 | INSERT/UPDATE/DELETE 처리 | 이벤트 자동 처리 |
| 호출 방식 | `SELECT` 뒤에서 호출 | `CALL pro_name()` | 자동 실행 (직접 호출 불가) |
| DML 사용 | ❌ (SELECT만) | ✅ | ✅ |

---

## 2. FUNCTION

> 리턴값이 있는 사용자 정의 함수 — `SELECT` 뒤에서 호출

```sql
-- 생성
CREATE OR REPLACE FUNCTION studentTotal(pHakbun student.hakbun%TYPE)
RETURN NUMBER
IS
    pSum NUMBER;
BEGIN
    SELECT kor + eng + math INTO pSum
    FROM student WHERE hakbun = pHakbun;
    RETURN pSum;
END;
/

-- 호출 (SELECT 뒤에서)
SELECT hakbun, name, studentTotal(hakbun) FROM student;
```

### 주요 활용 패턴
```sql
-- ROW 단위 계산
SELECT hakbun, name, studentAvg(hakbun) FROM student;

-- JOIN 대체 (스칼라 서브쿼리 대신)
SELECT empno, ename, deptGetDname(deptno) FROM emp;

-- 날짜 변환
SELECT empno, ename, dateChange(empno) FROM emp;
```

---

## 3. PROCEDURE

> 리턴값 없는 작업 수행용 — `INSERT/UPDATE/DELETE` 처리에 사용

```sql
-- INSERT 프로시저
CREATE OR REPLACE PROCEDURE studentInsert(
    pName  student.name%TYPE,
    pKor   student.kor%TYPE,
    pEng   student.eng%TYPE,
    pMath  student.math%TYPE
)
IS
BEGIN
    INSERT INTO student VALUES (std_seq.nextval, pName, pKor, pEng, pMath);
    COMMIT;
END;
/
CALL studentInsert('홍길동', 90, 87, 67);

-- SELECT 프로시저 (OUT 매개변수)
CREATE OR REPLACE PROCEDURE studentSelect(
    pHakbun student.hakbun%TYPE,
    pName   OUT student.name%TYPE,
    pKor    OUT student.kor%TYPE
)
IS
BEGIN
    SELECT name, kor INTO pName, pKor
    FROM student WHERE hakbun = pHakbun;
END;
/
```

### 매개변수 종류
| 종류 | 설명 | 사용 |
|------|------|------|
| `IN` | 값 입력 (기본값, 생략 가능) | INSERT/UPDATE/DELETE 조건 |
| `OUT` | 값 반환 | SELECT 결과 밖으로 전달 |
| `IN OUT` | 입력 + 반환 | 값 수정 후 반환 |

---

## 4. TRIGGER

> 테이블에 `INSERT/UPDATE/DELETE` 발생 시 **자동 실행**
> 반드시 **다른 테이블**에 적용 (같은 테이블 불가)

```sql
-- 입고 → 재고 자동 처리
CREATE OR REPLACE TRIGGER 입고_Trigger
AFTER INSERT ON 입고
FOR EACH ROW
DECLARE
    v_cnt NUMBER;
BEGIN
    SELECT COUNT(*) INTO v_cnt FROM 재고 WHERE 품번 = :NEW.품번;

    IF v_cnt = 0 THEN  -- 새 상품
        INSERT INTO 재고 VALUES (:NEW.품번, :NEW.수량, :NEW.금액, :NEW.수량 * :NEW.금액);
    ELSE               -- 기존 상품
        UPDATE 재고 SET
            수량 = 수량 + :NEW.수량,
            누적금액 = 누적금액 + (:NEW.수량 * :NEW.금액)
        WHERE 품번 = :NEW.품번;
    END IF;
END;
/
```

### 트리거 활용 예
| 이벤트 | 처리 |
|--------|------|
| 입고 INSERT | 재고 INSERT 또는 UPDATE (수량 증가) |
| 출고 INSERT | 재고 UPDATE (수량 감소) 또는 DELETE (재고 0) |
| 게시글 삭제 | 댓글 자동 삭제 |
| 좋아요 INSERT | 맛집 좋아요 수 UPDATE |

---

## 5. TCL — 트랜잭션 처리

```sql
COMMIT;              -- 변경사항 확정
ROLLBACK;            -- 전체 취소
SAVEPOINT sp1;       -- 저장 지점 설정
ROLLBACK TO sp1;     -- sp1 이후만 취소
```

> Java에서는 `conn.setAutoCommit(false)` → DML 수행 → `conn.commit()` / `conn.rollback()`

---

## 6. SQL 튜닝 포인트

| 나쁜 예 ❌ | 좋은 예 ✅ | 이유 |
|-----------|-----------|------|
| `SELECT *` | `SELECT empno, ename` | 필요한 컬럼만 조회 → I/O 감소 |
| JOIN 남발 | FUNCTION / 스칼라 서브쿼리 | SQL 단순화, 보안 |
| 인덱스 없음 | 자주 검색되는 컬럼에 INDEX | 검색 속도 향상 |
| DML에 인덱스 과다 | 인덱스 최소화 | DML 속도 저하 방지 |
