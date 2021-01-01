# DBCP(DataBase Connection Pool)

DB와 커넥션을 맺고 **커넥션 객체를 관리**하는 역할

JDBC만을 사용해 DB를 사용할 때의 반복되는 과정

1. DB 접속을 위한 JDBC 드라이버 로드
2. `getConnection` 메소드로 부터 DB 커넥션 객체를 얻음
3. 쿼리 수행을 위한 `PreparedStatement` 객체 생성
4. `executeQuery`를 실행해서 결과를 받음

1, 2 과정이 반복되는 것은 비효율적 -> DBCP를 이용한 커넥션 객체 관리

## JDBC(Java Database Connectivity)란?

자바에서 DB 프로그래밍을 하기 위해 사용되는 API

### JDBC 드라이버

각 DBMS에 알맞는 클라이언트  

Q. JDBC 드라이버는 자바 용 밖에 없는가?
    - python jayDeBeApi 패키지 존재
    - golang database/sql -> go-sql driver 존재(?)


---
[JDBC, DBCP란? 웹 어플리케이션의 DB접속에 대한 고찰 - 알짜배기 프로그래머](https://aljjabaegi.tistory.com/402) \
[DB Connection Pool - 용어 정리와 Single Connection의 문제점 by 버터필드](https://atoz-develop.tistory.com/entry/DB-Connection-Pool-%EC%9A%A9%EC%96%B4-%EC%A0%95%EB%A6%AC%EC%99%80-Single-Connection%EC%9D%98-%EB%AC%B8%EC%A0%9C%EC%A0%90)