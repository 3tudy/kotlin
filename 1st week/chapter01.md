# 1장 코틀린이란 무엇이며, 왜 필요한가

## 1.1 코틀린 맛보기

```kotlin
data class Person(val name: String,     // "데이터" 클래스
                 val age: Int? = null)  // null이 될 수 있는 타입(Int?)과 파라미터 디폴트 값

fun main(args: Array<String>) {         // 최상위 함수
    val persons = listOf(Person("영희"),
                        Person("철수", age=29)) // 이름 붙은 파라미터
    val oldest = persons.maxBy { it.age ?: 0}   // lambda expression과 Elvis operator
    println("나이가 가장 많은 사람: $oldest")   // 문자열 템플릿
}

//>> 나이가 가장 많은 사람: Person(name=철수, age=29)   // toString 자동 생성
```

Points

1. 영희는 나이를 지정하지 않았기에 `null`이 대신 쓰인다
2. 리스트에서 나이가 가장 많은 사람을 찾기 위해 `maxBy` 함수를 사용한다
3. `maxBy` 함수에 전달한 **lambda expression**은 파라미터를 하나 받는다
4. `it`이라는 이름을 사용하면 별도로 파라미터 이름을 정의하지 않아도 **lambda expression**의 유일한 인자를 사용할 수 있다
5. **Elvis operator**라고 부르는 `?:`는 `age`가 `null`인 경우 0을 반환, 그렇지 않으면 `age`의 값을 반환한다
6. 영희의 나이는 `null`이지만 **Elvis operator**가 `null`을 0으로 변환해 `age`를 비교할 수 있다

## 1.2 코틀린의 주요 특성

### 1.2.1 대상 플랫폼: 서버, 안드로이드 등 자바가 실행되는 모든 곳

> 코틀린의 주목적은 현재 자바가 사용되고 있는 모든 용도에 적합하면서도 더 간결하고 생산적이며 안전한 대체 언어를 제공하는것

1. 서버상의 코드(e.g. 웹 애플리케이션의 백엔드)
2. 안드로이드 디바이스에서 실행되는 모바일 애플리케이션
3. 인텔의 멀티OS 엔진위의 iOS 디바이스
4. 데스크탑 애플리케이션 (with TornadoFX, JavaFX)
5. 브라우저 (with JavaScript)

### 1.2.2 정적 타입 지정 언어

Kotlin은 정적 타입 지정언어

#### 정적 타입(Statically typed)

1. 모든 프로그램 구성 요소의 *타입*을 **컴파일 시점**에 알 수 있다
2. 프로그램 안에서 객체의 *필드*나 *메소드*를 사용할 때마다 **컴파일러가 타입을 검증해준다**

#### 동적 타입(Dynamically typed)

1. *타입*과 **관계없이** 모든 값을 변수에 넣을 수 있다
2. *메소드*나 *필드* 접근에 대한 검증이 **실행 시점**에 일어남

#### 타입 추론(Type Inference)

컴파일러가 문맥을 고려해 변수 타입을 결정하는 기능

#### Kotlin의 타입 시스템

1. Class
2. Interface
3. Generics
4. 널이 될 수 있는 타입(Nullable type)
    - 컴파일 시점에 *null pointer exception*을 체크할 수 있어 프로그램의 신뢰성을 높일 수 있다
5. **함수 타입**

### 1.2.3 함수형 프로그래밍과 객체지향 프로그래밍

### 함수형 프로그래밍

1. **First-class 함수**
   - 함수를 일반 값처럼
   - as a thing stored in variables, as an argument, as wrapper function
2. **불변성(Immutability)**
   - 불변 객체를 사용
   - 불변 객체: 일단 만들어지고 나면 내부 상태가 절대로 바뀌지 않는 객체
3. **부수 효과(side effect) 없음**
   - 입력이 같으면 항상 같은 출력
   - 다른 객체의 상태를 변경하지 않음
   - 함수 외부나 다른 바깥 환경과 상호작용하지 않는 순수 함수를 사용

## 1.3 코틀린 응용

### 1.3.1 코틀린 서버 프로그래밍

### 1.3.2 코틀린 안드로이드 프로그래밍

## 1.4 코틀린의 철학

### 1.4.1 실용성

### 1.4.2 간결성

### 1.4.3 안정성

### 1.4.4 상호운용성

## 1.5 코틀린 도구 사용

### 1.5.1 코틀린 코드 컴파일

### 1.5.2 인텔리J 아이디어와 안드로이드 스튜디오의 코틀린 플러그인

### 1.5.3 대화형 셸

### 1.5.4 이클립스 플러그인

### 1.5.5 온라인 놀이터

### 1.5.6 자바-코틀린 변환기