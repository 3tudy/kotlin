# 10장 애노테이션과 리플렉션

- 애노테이션 적용과 정의
- 리플렉션을 사용해 실행 시점에 객체 내부 관찰
- 코틀린 실전 프로젝트 예제

## 10.1 애노테이션 선언과 적용

### 10.1.1 애노테이션 적용

> 애노테이션을 적용하려면 적용하려는 대상 앞에 애노테이션을 붙이면 된다. 애노테이션은 `@<애노테이션 명>`으로 선언할 수 있다.

### `@Deprecated`

`ReplaceWith` 파라미터를 통해 옛 버전을 대신할 수 있는 패턴을 제사할 수 있다.

```kotlin
@Deprecated("Use removeAt(index) instead.", ReplaceWith("removeAt(index)"))
fun remove(index: Int) {...}
```

#### 애노테이션에 인자를 넘 때는 일반 함수와 마찬가지로 괄호 안에 인자를 넣는다

##### 애노테이션의 인자로는

- 원시 타입의 값
- 문자열
- enum
- 클래스 참조
- 다른 애노테이션 클래스
- 위 요소들로 이뤄진 배열

##### 애노테이션 인자를 지정하는 문법

- 클래스를 애노테이션 인자로 지정할 때: `@MyAnnotation(MyClass::class)`처럼 `::class`를 클래스 이름 뒤에 넣어야 한다.
- 다른 애노테이션을 인자로 지정할 때: 인자로 들어가는 애노테이션의 이름 앞에 `@`를 넣지 않아야 한다.(e.g. `ReplaceWith`)
- 배열을 인자로 지정할 때: `arrayOf` 함수를 사용한다. 자바에서 선언한 애노테이션 클래스를 사용한다면 `value`라는 이름의 파라미터가 필요에 따라 자동으로 **가변 길이 인자로 변환**된다.

#### 애노테이션 인자를 컴파일 시점에 알 수 있어야 한다

> 임의의 **프로퍼티를 인자로 지정할 수 없다.** 프로퍼티를 애노테이션으로 사용하려면 그 앞에 `const` 변경자를 붙여야 한다. 컴파일러는 `const`가 붙은 프로퍼티를 **컴파일 시점 상수**로 취급한다.

```kotlin
const val TEST_TIMEOUT = 100L

@Test(timeout=TEST_TIMEOUT) fun testMethod(){...}
```

##### `const`가 붙은 프로퍼티는 파일의 맨 위나 object 안에 선언해야 하며, 원시 타입이나 `String`으로 초기화해야 한다

### 10.1.2 애노테이션 대상

> **코틀린 소스코드에서 한 선언**을 컴파일한 결과가 **여러 자바 선언과 대응**하는 경우가 자주 있다. 그리고 이때 코틀린 선언과 대응하는 여러 자바 선언에 각각 애노테이션을 붙여야 할 때가 있다.

#### **사용 지점 대상<sup>use-site target</sup>** 선언으로 애노테이션을 붙일 요소를 정할 수 있다

> 사용 지점 대상은 **`@` 기호와 애노테이션 이름 사이**에 붙으며, 애노테이션 이름과는 **콜론(:)**으로 분리된다.

```kotlin
@get:Rule   // @<사용 지점 대상>:<애노테이션 명>
// 프로퍼티가 아니라 게터에 애노테이션을 붙일 때 활용할 수 있다.
```

##### 사용 지점 대상을 지정할 때 지원하는 대상 목록

| 대상 | 설명 |
|---|---|
| `property` | 프로퍼티 전체. 자바에서 선언된 애노테이션에는 이 사용 지점 대상을 사용할 수 없다. |
| `field` | 프로퍼티에 의해 생성되는 (뒷받침하는) 필드 |
| `get` | 프로퍼티 게터 |
| `set` | 프로퍼티 세터 |
| `receiver` | 확장 함수나 프로퍼티의 수신 객체 파라미터 |
| `param` | 생성자 파라미터 |
| `setparam` | 세터 파라미터 |
| `delegate` | 위임 프로퍼티의 위임 인스턴스를 담아둔 필드 |
| `file` | 파일 안에 선언된 최상위 함수와 프로퍼티를 담아두는 클래스 |

- `file` 대상을 사용하는 애노테이션은 `package` 선언 앞에서 파일의 최상위 수준에만 적용할 수 있다.

### 10.1.3 애노테이션을 활용한 JSON 직렬화 제어

> **직렬화**<sup>serialization</sup>는 **객체를** 저장장치에 저장하거나 네트워크를 통해 전송하기 위해 텍스트나 이진 **형식으로 변환하는 것**이다. 반대 과정인 **역직렬화**<sup>deserialization</sup>는 텍스트나 이진 형식으로 저장된 **데이터**로부터 **원래의 객체를 만들어 낸다**.

#### `@JsonExclude`

해당 애노테이션을 사용하면 직렬화나 역직렬화 시 그 프로퍼티를 무시할 수 있다.

#### `@JsonName`

해당 애노테이션을 사용하면 프로퍼티를 표현하는 키/값 쌍의 키로 프로퍼티 이름 대신 애노테이션이 지정한 이름을 쓰게 할 수 있다.

```kotlin
data class Person (
    @JsonName("alias") val firstName: String,   // 직렬화 시 `firstName` 대신 `alias`를 키로 쓰게 된다.
    @JsonExclude val age: Int? = null   // 직렬화나 역직렬화 시에 'age' 프로퍼티를 무시한다.
)
```

### 10.1.4 애노테이션 선언

```kotlin
/* kotlin */
annotation class JsonExclude
annotation class JsonName(val name: String)
```

```java
/* java */
public @interface JsonName {
    String value(); // 자바에서는 name 프로퍼티 대신 value 프로퍼티를 사용한다.
}
```

### 10.1.5 메타애노테이션: 애노테이션을 처리하는 방법 제어

> 자바와 마찬가지로 코틀린 애노테이션 클래스에도 애노테이션을 붙일 수 있다. 애노테이션 클래스에 적용할 수 있는 애노테이션을 **메타애노테이션**<sup>meta-annotation</sup>이라고 부른다.

#### `@Target` 메타애노테이션은 애노테이션을 적용할 수 있는 요소의 유형을 지정한다

```kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class JsonExclude
```

애노테이션 클래스에 대해 구체적인 `@Target`을 지정하지 않으면 모든 선언에 적용할 수 있는 애노테이션이 된다.

#### 애노테이션이 붙을 수 있는 대상이 정의된 이넘<sup>enum</sup>은 `AnnotationTarget`이다

> `AnnotationTarget`안에는 클래스, 파일, 프로퍼티, 프로퍼티 접근자, 타입, 식 등에 대한 이넘 정의가 들어있다. 필요하다면 `@Target(AnnotationTarget.CLASS, AnnotationTarget.METHOD)`처럼 둘 이상의 대상을 한꺼번에 선언할 수도 있다.

메타애노테이션을 직접 만들어야 한다면 `ANNOTATION_CLASS`를 대상으로 지정해야 한다.

```kotlin
@Target(AnnotationTarget.ANNOTATION_CLASS)
annotation class BindingAnnotation

@BindingAnnotation
annotation class MyBinding
```

대상을 `PROPERTY`로 지정한 애노테이션을 자바 코드에서 사용할 수는 없다. 자바에서 그런 애노테이션을 사용하려면 `AnnotationTarget.FIELD`를 두 번째 대상으로 추가해야 한다. 그렇게 하면 애노테이션을 코틀린 프로퍼티와 자바 필드에 적용할 수 있다.

### 10.1.6 애노테이션 파라미터로 클래스 사용

클래스 참조를 파라미터로 하는 애노테이션 클래스를 선언할 수 있다. 이 선언은 어떤 클래스를 선언 메타데이터로 참조할 수 있는 기능을 제공할 수 있다. 그러한 애노테이션의 하나로 제이키드 라이브러리의 `@DeserializeInterface`를 예로 들 수 있다.

#### `@DeserializeInterface`는 인터페이스 타입인 프로퍼티에 대한 역직렬화를 제어할 때 쓰는 애노테이션이다

인터페이스의 인스턴스를 직접 만들 수는 없기 때문에 **역직렬화 시 어떤 클래스를 사용해** 인터페이스를 **구현**할지를 **지정**할 수 있어야 한다.

```kotlin
interface Company {
    val name: String
}

data class CompanyImpl(override val name: String): Company

data class Person(
    val name: String,
    // 역직렬화를 사용할 클래스를 지정하기 위해 인자로 `CompanyImpl::class`를 넘긴다.
    // 일반적으로 클래스를 가리키려면 클래스 이름 뒤에 `::class` 키워드를 붙여야 한다.
    @DeserializeInterface(CompanyImpl::class) val company: Company
)
```

직렬화된 `Person` 인스턴스를 역직렬화하는 과정에서 `company` 프로퍼티를 표현하는 JSON을 읽으면 제이키드는 그 프로퍼티 값에 해당하는 JSON을 역직렬화하면서 **`CompanyImpl`의 인스턴스를 만들어서** `Person` 인스턴스의 `company` 프로퍼티에 설정한다.

### 10.1.7 에노테이션 파라미터로 제네릭 클래스 받기

> `@CustomSerializer` 애노테이션은 커스텀 직렬화 클래스에 대한 참조를 인자로 받는다.

## 10.2 리플렉션: 실행 시점에 코틀린 객체 내부 관찰

### 10.2.1 코틀린 리플렉션 API: KClass, KCallable, KFunction, KProperty

### 10.2.2 리플렉션을 사용한 객체 직렬화 구현

### 10.2.3 애노테이션을 활용한 직렬화 제어

### 10.2.4 JSON 파싱과 객체 역직렬화

### 10.2.5 최종 역질렬화 단계: `callBy()`, 리플렉션을 사용해 객체 만들기
