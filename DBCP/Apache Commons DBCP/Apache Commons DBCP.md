# Apache Commons DBCP

## 커넥션의 개수

### 커넥션 풀의 저장 구조

commons-pool에서 제공하는 리소스 풀의 기능을 이용한다.

#### commons-pool

object pooling을 제공하는 아파치의 패키지이다.

object pooling이란 데이터베이스 커넥션 객체와 같이 자주 생성되는 객체를 미리 생성해두고 필요할 때마다 객체를 생성하거나 **이미 생성된 객체를 반환시켜 사용**하도록 만드는 방식이라고 보면 될거 같다. 또한, 사용이 끝난 객체를 버리지 않고 다시 오브젝트 풀로 **반납**하여 다시 사용할 수 있어 자원을 좀 더 효율적으로 쓸 수 있게 한다.

DBCP인 Apache Commons가 commons-pool을 사용한다는 말은 데이터베이스에 대한 커넥션 객체들을 commons-pool에 미리 생성해두고 이를 **대여, 반납, 생성**하는 형식으로 커넥션 관리를 수행하고 있다고 생각하면 되겠다.

#### 그렇다면 커넥션의 관리는 어떻게 이루어지는가

![commons-pool](./commons-pool의%20GenericObjectPool.png)

[그림 출처 - Commons DBCP 이해하기](https://d2.naver.com/helloworld/5102792)

1. Commons DBCP는 `PoolableConnection` 타입의 커넥션 객체를 생성하고
2. 생성한 커넥션 객체에 `ConnectionEventListener`를 등록한다.
3. `ConnectionEventListener`에는 애플리케이션이 사용한 커넥션 풀로 반환하기 위해 **JDBC 드라이버가 호출할 수 있는 콜백 메소드**가 있다.
4. 이렇게 생성된 커넥션 객체는 commons-pool의 `addObject()` 메소드로 커넥션 풀에 추가된다.
5. 이때, commons-pool은 내부적으로 **현재 시간을 담고 있는 타임 스탬프**와 **추가된 커넥션 객체의 레퍼런스**를 한 쌍으로 하는 `ObjectTimestampPair` 자료구조를 생성한다.
6. 그리고 이들은 **LIFO 형태의 `CursorableLinkedList`로 관리**한다.

#### 커넥션 개수 관련 속성

> 커넥션의 개수는 `BasicDataSource` 클래스의 다음 속성으로 지정할 수 있다.

| 속성 이름 | 설명 |
|---|---|
| `initialSize` | `BasicDataSource` 클래스 생성 후 최초로 `getConnection()` 메소드를 호출할 때 커넥션 풀에 채워 넣을 커넥션 개수 |
| `maxActive` | 동시에 사용할 수 있는 최대 커넥션의 개수 |
| `maxIdle` | 커넥션 풀에 반납할 때 최대로 유지될 수 있는 커넥션 개수 |
| `minIdle` | 최소한으로 유지할 커넥션 개수 |

##### 커넥션 개수와 관련된 속성은 다음 조건을 만족시켜야 한다

- maxActive >= initialSize
- maxIdle >= minIdle
- maxActive = maxIdle

##### 커넥션 개수 속성들을 설정할 때 고려 사항들

1. `maxActive` 값은 DBMS의 설정과 애플리케이션 서버의 개수, 애플리케이션 서버(Apache, Tomcat)에서 동시에 처리할 수 있는 사용자 수 등을 고려해야 한다.
2. Commons DBCP에서는 DBMS에 로그인을 시도하고 있는 커넥션도 사용중인 것으로 간주한다.
    - LoginTimeOut과 같은 속성을 설정하는 것도 고려해 장애의 확산을 방지할 수 있어야 한다.

#### `maxWait` 속성

TPS와 서버의 처리 가능한 스레드 개수, 실제 유저의 대기 가능 시간 등을 고려해 최적점을 찾는 것이 이상적이다.

---
[Commons DBCP 이해하기 - Naver D2](https://d2.naver.com/helloworld/5102792) \
[[JAVA] Object Pooling - 곰팡이 먼지연구소 블로그](https://gompangs.tistory.com/entry/JAVA-Object-Pooling)
