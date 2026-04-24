# 📘 Day 10 — Java + Oracle 연동 (JDBC)

## 0. 핵심 문법 빠른 참조

| 구문 | 예시 | 설명 |
|------|------|------|
| 드라이버 등록 | `Class.forName("oracle.jdbc.driver.OracleDriver")` | Oracle 드라이버 로드 |
| MySQL 드라이버 | `Class.forName("com.mysql.cj.jdbc.Driver")` | MySQL/MariaDB |
| DB 연결 | `DriverManager.getConnection(URL, "hr", "happy")` | Connection 객체 생성 |
| Oracle URL | `jdbc:oracle:thin:@localhost:1521:XE` | Oracle 접속 URL |
| SQL 전송 | `conn.prepareStatement(sql)` | PreparedStatement 생성 |
| SELECT 실행 | `ps.executeQuery()` → `ResultSet` | 조회 결과 반환 |
| DML 실행 | `ps.executeUpdate()` → `int` | 영향받은 행 수 반환 |
| 닫기 | `ps.close()` / `conn.close()` | 자원 해제 필수 |

---

## 1. JDBC 연동 순서

```java
// 1. 드라이버 등록
Class.forName("oracle.jdbc.driver.OracleDriver");

// 2. DB 연결
String url = "jdbc:oracle:thin:@localhost:1521:XE";
Connection conn = DriverManager.getConnection(url, "hr", "happy");
// ⚠️ URL, 계정정보는 GIT에 올리지 말 것

// 3. SQL 작성
String sql = "SELECT * FROM movie WHERE mno = ?";

// 4. SQL 전송
PreparedStatement ps = conn.prepareStatement(sql);
ps.setInt(1, mno);

// 5. 실행 & 결과 받기
ResultSet rs = ps.executeQuery();       // SELECT
// int result = ps.executeUpdate();     // INSERT / UPDATE / DELETE

// 6. VO에 값 채우기
List<MovieVO> list = new ArrayList<>();
while (rs.next()) {
    MovieVO vo = new MovieVO();
    vo.setMno(rs.getInt("mno"));
    vo.setTitle(rs.getString("title"));
    list.add(vo);
}

// 7. 닫기
ps.close();
conn.close();
```

---

## 2. 주요 라이브러리

| 분류 | 클래스 | 용도 |
|------|--------|------|
| JDBC | `Connection` | DB 연결 |
| JDBC | `PreparedStatement` | SQL 실행 |
| JDBC | `ResultSet` | SELECT 결과 저장 |
| 컬렉션 | `List<VO>` | 여러 ROW 저장 |
| 컬렉션 | `Map` | Key-Value 데이터 저장 |
| 문자열 | `String`, `StringTokenizer` | 문자열 처리 |
| 날짜 | `Date` | 날짜 처리 |
| 수학 | `Math.ceil()` | 올림 (페이징 총 페이지 계산) |

---

## 3. 기능별 설계 패턴

| 기능 | 리턴형 | 매개변수 | SQL |
|------|--------|----------|-----|
| 목록 | `List<MovieVO>` | `int page` | `SELECT` + `ROWNUM` 페이징 |
| 상세보기 | `MovieVO` | `int mno` | `SELECT WHERE mno = ?` |
| 검색 | `List<MovieVO>` | `String fd, String column` | `SELECT WHERE column LIKE ?` |
| 등록 | `int` | `MovieVO vo` | `INSERT` |
| 수정 | `int` | `MovieVO vo` | `UPDATE WHERE mno = ?` |
| 삭제 | `int` | `int mno` | `DELETE WHERE mno = ?` |

---

## 4. Java 핵심 개념 복습

```
Java 핵심
├── 변수 / VO (캡슐화)        → DB 1 ROW = Java 객체 1개
├── 연산자 / 제어문           → if / for / while
├── 배열 / List              → 여러 VO 저장 후 전송
├── 객체지향
│   ├── 캡슐화               → VO (getter/setter)
│   ├── 포함                 → Connection, PreparedStatement
│   └── 오버라이딩           → 메소드 재정의
├── 예외처리                  → try ~ catch (DB 연결 필수)
└── J2EE                    → 브라우저 요청값 받기 / 전송
```

> ⚠️ DB URL, 계정정보(ID/PW)는 절대 GitHub에 올리지 말 것
> ⚠️ `ps.close()` / `conn.close()` 자원 해제 필수 — `finally` 블록에서 처리
