# 📘 Day 08 — PL/SQL

## 0. 핵심 문법 빠른 참조

| 구문 | 예시 | 설명 |
|------|------|------|
| 일반 변수 선언 | `vename VARCHAR2(20)` | 직접 데이터형 지정 |
| `%TYPE` | `vename emp.ename%TYPE` | 컬럼 데이터형 자동 참조 |
| `%ROWTYPE` | `vemp emp%ROWTYPE` | 테이블 전체 컬럼 구조 참조 |
| `RECORD` | `TYPE empDept IS RECORD(...)` | 사용자 정의 구조체 |
| 값 대입 | `vname := '홍길동'` | `:=` 으로 대입 |
| 초기값 설정 | `vno NUMBER := 1` | 선언과 동시에 초기값 |
| SELECT INTO | `SELECT ename INTO vename FROM emp WHERE empno=7788` | 변수에 쿼리 결과 저장 |
| 출력 | `DBMS_OUTPUT.PUT_LINE('결과: ' \|\| vename)` | 콘솔 출력 |
| IF 단일 | `IF 조건 THEN 처리 END IF` | 단일 조건 |
| IF~ELSIF | `IF 조건 THEN ... ELSIF 조건 THEN ... ELSE ... END IF` | 다중 조건 |
| IF~ELSE | `IF 조건 THEN 처리 ELSE 처리 END IF` | 선택 조건 |
| CASE | `CASE 값 WHEN 조건 THEN 값 ELSE 값 END` | 선택문 |
| LOOP | `LOOP 처리 EXIT WHEN 조건 END LOOP` | do~while 방식 |
| WHILE | `WHILE 조건 LOOP 처리 END LOOP` | 조건 반복 |
| FOR | `FOR i IN 1..10 LOOP 처리 END LOOP` | 범위 반복 |
| FOR 역순 | `FOR i IN REVERSE 1..10 LOOP 처리 END LOOP` | 역순 반복 |
| CURSOR 선언 | `CURSOR cur IS SELECT * FROM emp` | 여러 ROW 저장 |
| CURSOR FOR | `FOR vemp IN cur LOOP 처리 END LOOP` | 커서 간편 반복 |

---

## 1. PL/SQL 기본 구조

```sql
SET SERVEROUTPUT ON;  -- 출력 활성화 (필수)

DECLARE
    -- 변수 선언부
BEGIN
    -- 실행부 (SQL + 제어문)
END;
/
```

> - `DECLARE` : 변수 선언 (생략 가능)
> - `BEGIN ~ END` : 실제 실행 코드
> - `/` : PL/SQL 블록 실행 명령
> - Java의 `System.out.println()` → `DBMS_OUTPUT.PUT_LINE()`

---

## 2. 변수 종류

### 일반 변수 (스칼라)
```sql
DECLARE
    vempno   NUMBER(4);
    vename   VARCHAR2(20);
    vhiredate DATE;
BEGIN
    SELECT empno, ename, hiredate INTO vempno, vename, vhiredate
    FROM emp WHERE empno = 7788;

    DBMS_OUTPUT.PUT_LINE('사번: ' || vempno);
    DBMS_OUTPUT.PUT_LINE('이름: ' || vename);
END;
/
```

### %TYPE — 컬럼 데이터형 자동 참조 ⭐
```sql
DECLARE
    vname customer.name%TYPE;      -- customer.name의 데이터형과 동일하게 선언
    vphone customer.phone%TYPE;
BEGIN
    SELECT name, phone INTO vname, vphone
    FROM customer WHERE custid = 1;

    DBMS_OUTPUT.PUT_LINE('이름: ' || vname);
    DBMS_OUTPUT.PUT_LINE('전화: ' || vphone);
END;
/
```
> 컬럼 데이터형이 바뀌어도 자동 반영 → 유지보수에 유리

### %ROWTYPE — 테이블 전체 행 구조 참조
```sql
DECLARE
    vemp emp%ROWTYPE;   -- emp 테이블의 모든 컬럼을 한번에 선언
BEGIN
    SELECT * INTO vemp FROM emp WHERE empno = &사번;

    DBMS_OUTPUT.PUT_LINE('사번: ' || vemp.empno);
    DBMS_OUTPUT.PUT_LINE('이름: ' || vemp.ename);
    DBMS_OUTPUT.PUT_LINE('급여: ' || vemp.sal);
END;
/
```

### RECORD — 사용자 정의 구조체 (JOIN 결과 저장)
```sql
DECLARE
    TYPE empDept IS RECORD (
        empno    emp.empno%TYPE,
        ename    emp.ename%TYPE,
        dname    dept.dname%TYPE,
        grade    salgrade.grade%TYPE
    );
    ed empDept;
BEGIN
    SELECT empno, ename, dname, grade INTO ed
    FROM emp
    JOIN dept ON emp.deptno = dept.deptno
    JOIN salgrade ON sal BETWEEN losal AND hisal
    WHERE empno = &사번;

    DBMS_OUTPUT.PUT_LINE('이름: ' || ed.ename);
    DBMS_OUTPUT.PUT_LINE('부서: ' || ed.dname);
    DBMS_OUTPUT.PUT_LINE('등급: ' || ed.grade);
END;
/
```

---

## 3. 제어문

### IF 단일 조건
```sql
IF vdeptno = 10 THEN
    vdname := '개발부';
END IF;
```

### IF ~ ELSIF ~ ELSE 다중 조건
```sql
IF vdeptno = 10 THEN
    vdname := '개발부';
ELSIF vdeptno = 20 THEN
    vdname := '영업부';
ELSIF vdeptno = 30 THEN
    vdname := '기획부';
ELSE
    vdname := '신입';
END IF;
```

### IF ~ ELSE 선택 조건
```sql
IF vcomm > 0 THEN
    DBMS_OUTPUT.PUT_LINE(vename || ' 성과급: ' || vcomm);
ELSE
    DBMS_OUTPUT.PUT_LINE(vename || ' 성과급 없음');
END IF;
```

### CASE 선택문
```sql
vdname := CASE vdeptno
    WHEN 10 THEN '개발부'
    WHEN 20 THEN '영업부'
    WHEN 30 THEN '기획부'
    ELSE '신입'
END;
```

### IF vs CASE 비교
| 구분 | IF ~ ELSIF | CASE |
|------|-----------|------|
| 조건 범위 | 모든 연산자 사용 가능 | 단순 값 비교 |
| 가독성 | 복잡한 조건에 유리 | 단순 비교에 유리 |

---

## 4. 반복문

### LOOP (do~while 방식)
```sql
DECLARE
    sno NUMBER := 1;
BEGIN
    LOOP
        DBMS_OUTPUT.PUT_LINE(sno);
        sno := sno + 1;
        EXIT WHEN sno > 10;   -- 종료 조건
    END LOOP;
END;
/
```

### WHILE
```sql
DECLARE
    no NUMBER := 1;
BEGIN
    WHILE no <= 10 LOOP
        DBMS_OUTPUT.PUT(no || ' ');
        no := no + 1;
    END LOOP;
    DBMS_OUTPUT.NEW_LINE;
END;
/
```

### FOR ⭐ (가장 많이 사용)
```sql
-- 오름차순
FOR i IN 1..10 LOOP
    DBMS_OUTPUT.PUT(i || ' ');
END LOOP;

-- 내림차순
FOR i IN REVERSE 1..10 LOOP
    DBMS_OUTPUT.PUT(i || ' ');
END LOOP;
```

### FOR 활용 예 — 1~100 짝수합/홀수합
```sql
DECLARE
    total NUMBER := 0;
    even  NUMBER := 0;
    odd   NUMBER := 0;
BEGIN
    FOR i IN 1..100 LOOP
        total := total + i;
        IF MOD(i, 2) = 0 THEN
            even := even + i;
        ELSE
            odd := odd + i;
        END IF;
    END LOOP;
    DBMS_OUTPUT.PUT_LINE('총합: '   || total);
    DBMS_OUTPUT.PUT_LINE('짝수합: ' || even);
    DBMS_OUTPUT.PUT_LINE('홀수합: ' || odd);
END;
/
```

---

## 5. CURSOR — 여러 ROW 처리

> `SELECT INTO`는 1개 ROW만 받을 수 있음
> 여러 ROW를 처리하려면 **CURSOR** 사용

### 기본 커서 (OPEN → FETCH → CLOSE)
```sql
DECLARE
    vemp emp%ROWTYPE;
    CURSOR cur IS SELECT * FROM emp;
BEGIN
    OPEN cur;
    LOOP
        FETCH cur INTO vemp;
        EXIT WHEN cur%NOTFOUND;   -- 데이터 없으면 종료
        DBMS_OUTPUT.PUT_LINE(vemp.empno || ' ' || vemp.ename);
    END LOOP;
    CLOSE cur;
END;
/
```

### FOR 커서 (간편 방식) ⭐
```sql
DECLARE
    CURSOR cur IS SELECT * FROM emp;
BEGIN
    FOR vemp IN cur LOOP   -- OPEN/FETCH/CLOSE 자동 처리
        DBMS_OUTPUT.PUT_LINE(vemp.empno || ' ' || vemp.ename);
    END LOOP;
END;
/
```

---

## 6. PL/SQL 전체 구조 요약

```
PL/SQL
├── 변수
│   ├── 스칼라         NUMBER, VARCHAR2, DATE ...
│   ├── %TYPE          테이블.컬럼%TYPE
│   ├── %ROWTYPE       테이블%ROWTYPE
│   └── RECORD         TYPE 이름 IS RECORD(...)
├── 제어문
│   ├── IF / ELSIF / ELSE / END IF
│   └── CASE ~ WHEN ~ ELSE ~ END
├── 반복문
│   ├── LOOP ~ EXIT WHEN ~ END LOOP
│   ├── WHILE ~ LOOP ~ END LOOP
│   └── FOR 변수 IN 범위 LOOP ~ END LOOP  ⭐
└── CURSOR
    ├── 기본: OPEN → FETCH → CLOSE
    └── FOR 커서: 자동 처리 ⭐
```

> ⚠️ PL/SQL에서 `SELECT`는 반드시 `INTO`로 변수에 저장해야 함
> ⚠️ 반복문 증가는 `i++` 대신 `i := i + 1`
> ⚠️ `FUNCTION / PROCEDURE / TRIGGER` 제작 시 `DECLARE` 대신 `CREATE` 사용
