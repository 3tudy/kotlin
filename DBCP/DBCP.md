# DBCP(DataBase Connection Pool)

DB와 커넥션을 맺고 **커넥션 객체를 관리**하는 역할

## JDBC(Java Database Connectivity)란?

자바에서 DB 프로그래밍을 하기 위해 사용되는 API

### JDBC 드라이버

각 DBMS에 알맞는 클라이언트  

Q. JDBC 드라이버는 자바 용 밖에 없는가?
    - python jayDeBeApi 패키지 존재
    - golang database/sql -> go-sql driver 존재(?)

## JDBC만을 사용해 DB를 사용할 때의 반복되는 과정

1. DB 접속을 위한 JDBC 드라이버 로드
2. `getConnection` 메소드로 부터 DB 커넥션 객체를 얻음
3. 쿼리 수행을 위한 `PreparedStatement` 객체 생성
4. `executeQuery`를 실행해서 결과를 받음

### 1, 2 과정이 반복되는 것은 비효율적 -> DBCP를 이용한 커넥션 객체 관리

- 각 요청에 대해 별도의 커넥션 객체를 사용해 다른 DB 작업에 영향을 주지 않는다.
- 사용한 DB 객체는 버리지 않고 풀에 보관해 두었다가 재사용한다.

### DAO가 DBCP로 부터 커넥션 객체를 얻어 작업을 처리하는 과정

1. 클라이언트를 통해 DB 작업이 호출됨
2. DAO는 DBCP에게 커넥션 객체 요청
3. DBCP는 현재 보유하고 있는 커넥션 객체가 없으면
   - 새로 커넥션0 **객체를 생성**해서 반환하거나
   - **보유하고 있는 커넥션 객체**가 있으면 그 객체를 반환한다.
4. DAO는 반환받은 커넥션 객체를 사용해서 SQL문을 실행한다.
5. 작업이 끝나고 DAO는 DBCP에 커넥션 객체를 반환한다.
6. DBCP는 돌려받은 커넥션 객체를 **보관**한다.

#### 위 과정에서 사용자 설정이 필요해 보이는 인자를 알아보자

1. 보유할 수 있는 커넥션 객체의 수
2. DB 정보(ID, DB name) -> 커넥션 풀이 커넥션 객체를 생성하기 위해 필요한 정보일 거 같다.

## 오픈소스 라이브러리로 존재하는 DBCP들

- Commons DBCP
- Tomcat-JDBC
- BoneCP
- Hikari DBCP
- etc..

---
[JDBC, DBCP란? 웹 어플리케이션의 DB접속에 대한 고찰 - 알짜배기 프로그래머](https://aljjabaegi.tistory.com/402) \
[DB Connection Pool - 용어 정리와 Single Connection의 문제점 by 버터필드](https://atoz-develop.tistory.com/entry/DB-Connection-Pool-%EC%9A%A9%EC%96%B4-%EC%A0%95%EB%A6%AC%EC%99%80-Single-Connection%EC%9D%98-%EB%AC%B8%EC%A0%9C%EC%A0%90) \
[Commons DBCP 이해하기 - Naver D2](https://d2.naver.com/helloworld/5102792)
