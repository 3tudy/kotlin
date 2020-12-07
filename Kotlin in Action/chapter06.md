# 6장 코틀린 타입 시스템

- 널이 될 수 있는 타입과 널을 처리하는 구문의 문법
- 코틀린 원시 타입 소개와 자바 타입과 코틀린 원시 타입의 관계
- 코틀린 컬렉션 소개와 자바 컬렉션과 코틀린 컬렉션의 관계

## 6.1 널 가능성

> 널 가능성<sup>nullability</sup>은 `NullPointerException` 오류를 피할 수 있게 돕기 위한 코틀린 타입 시스템의 특성이다.

### 6.1.1 널이 될 수 있는 타입

타입 이름 뒤에 **물음표(?)**를 붙여 그 타입의 변수나 프로퍼티에 `null` 참조를 저장할 수 있음을 알릴 수 있다. 따라서 기본적으로 모든 타입은 `null`이 될 수 없는 타입이다.

```kotlin
// 함수가 null을 인자로 받을 수 없으면
fun strlen(s: String) = s.length

// 함수가 null을 인자로 받을 수 있으면
fun strlen(s: String?) = s.length
```

- `null`이 될 수 잇는 값을 널이 될 수 없는 타입의 변수에 대입할 수 없다.

    ```kotlin
    >>> fun strLenSafe(s: String?) = s.length() // 인스턴스 s는 null 될 수 있기에 length 메소드를 직접 호출할 수 없다.
    ERROR: only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type kotlin.String?
    ```

- `null`이 될 수 있는 타입의 값을 `null`이 될 수 없는 타입의 파라미터를 받는 함수에 전달할 수 없다.

    ```kotlin
    >>> val x: String? = null
    >>> val y: String = x
    ERROR: Type mismatch: inferred type is String? but String was expected
    >>> strLen(x)
    ERROR: Type mismatch: inferred type is String? but String was expected
    ```

- `null`이 될 수 있는 타입의 값과 `null`을 비교하고 나면 컴파일이 가능하다.

    ```kotlin
    fun strLenSafe(s: String?): Int =
        if (s != null) s.length else 0  // null 검사를 추가하면 코드가 컴파일된다.
    >>> val x: String? = null
    >>> println(strLenSafe(x))
    0
    >>> println(strLenSafe("abc"))
    3
    ```

### 6.1.2 타입의 의미

> 타입은 분류<sup>classification</sup>로 타입은 어떤 값들이 가능한지와 그 타입에 대해 수행할 수 있는 연산의 종류를 결정한다.

### 6.1.3 안전한 호출 연산자: `?.`

```kotlin
/* in kotlin */
foo?.bar()

/* in java */
if (foo != null) foo.bar() else null
```

```kotlin
fun printAllCaps(s: String?) {
    val allCaps: String? = s?.toUpperCase() // s?.toUpperCase()은 반환값 역시 String? 타입이다.
    println(allCaps)
}
>>> printAllCaps("abc")
ABC
>>> printAllCaps(null)
null
```

- `null`이 될 수 있는 프로퍼티를 다루기 위해 안전한 호출 사용하기

    ```kotlin
    class Employee(val name: String, val manager: Employee?)
    fun managerName(employee: Employee): String? = employee.manager?.name
    >>> val ceo = Employee("Da Boss", null)
    >>> val developer = Employee("Bob Smith", ceo)
    >>> println(managerName(developer))
    Da Boss
    >>> println(managerName(ceo))
    null
    ```

- 안전한 호출 연쇄시키기

    ```kotlin
    class Address(val streetAddress: String, val zipCode: Int, val city: String, val country: String)

    class Company(val name: String, val address: Address?)
    class Person(val name: String, val company: Company?)
    fun Person.countryName(): String {
        val country = this.company?.address?.country
        return if (country != null) country else "Unknown"
    }
    >>> val person = Person("Dmitry", null)
    >>> println(person.countryName())
    Unknown
    ```

### 6.1.4 엘비스 연산자: `?:`

> 코틀린은 `null` 대신 사용할 디폴트 값을 지정할 때 편리하게 사용할 수 있는 연산자를 제공한다. 다음은 엘비스<sup>Elvis</sup> 연산자를 사용하는 방법이다.

```kotlin
fun foo(s: String?) {
    val t: String = s ?: "" // "s"가 null이면 결과는 빈 문자열("")이다.
}
```

#### 엘비스 연산자

- 이항 연산자
- 연산 과정
  1. 좌항을 계산한 값이 `null`인지 검사한다.
  2. 좌항 값이 `null`이 아니면 좌항 값을 결과로 한다.
  3. 좌항 값이 `null`이면 우항 값을 결과로 한다.
- 엘비스 연산자를 활용해 null 값 다루기

    ```kotlin
    fun strLenSafe(s: String?): Int = s?.length ?: 0
    >>> println(strLenSafe("abc"))
    3
    >>> println(strLenSafe(null))
    0
    ```

- `throw`와 엘비스 연산자 함께 사용하기

    ```kotlin
    class Address(val streetAddress: String, val zipCode: Int, val city: String, val country: String)
    class Company(val name: String, val address: Address?)
    class Person(val name: String, val company: Company?)
    fun printShippingLabel(person: Person) {
        val address = person.company?.address
            ?: throw IllegalArgumentException("No address") // 주소가 없으면 예외를 발생시킨다.
            with (address) {    // 엘비스 연산자 연산의 결과로 address는 null이 아님을 알 수 있다.
                println(streetAddress)
                println("$zipCode $city, $country")
            }
    }
    >>> val address = Address("Elsestr. 47", 80687, "Munich", "Germany")
    >>> val jetbrains = Company("JetBrains", address)
    >>> val person = Person("Dmitry", jetbrains)
    >>> printShippingLabel(person)
    Elsestr. 47
    80687 Munich, Germany
    >>> printShippingLabel(Person("Alexey", null))
    java.lang.IllegalArgumentException: No address
    ```

### 6.1.5 안전한 캐스트: `as?`

> `as?` 연산자는 어떤 값을 지정한 타입으로 캐스트한다. `as?`는 값을 대상 타입으로 변환할 수 없으면 `null`을 반환한다.

- 안전한 연산자를 사용해 `equals` 구현하기

    ```kotlin
    class Person(val firstName: String, val lastName: String) {
        override fun equals(o: Any?): Boolean {
            val otherPerson = o as? Person ?: return false
            return otherPerson.firstName == firstName &&
                    otherPerson.lastName == lastName
        }
        override fun hashCode(): Int =
            firstName.hashCode() * 37 + lastName.hashCode()
    }
    >>> val p1 = Person("Dmitry", "Jemerov")
    >>> val p2 = Person("Dmitry", "Jemerov")
    >>> println(p1 == p2)
    true
    >>> println(p1.equals(42))
    false
    ```

### 6.1.6 널 아님 단언: `!!`

> **널 아님 단언<sup>not-null assertion</sup>** 은 코틀린에서 널이 될 수 있는 타입의 값을 다룰 때 사용할 수 있는 도구 중에서 가장 단순하면서도 무딘 도구다. 느낌표를 이중(!!)으로 사용하면 어떤 값이든 `null`이 될 수 없는 타입으로 바꿀 수 있다.

```kotlin
fun ignoreNulls(s: String?) {
    val sNotNull: String = s!!  // 예외가 가리키는 지점
    println(sNotNull.length)
}
>>> ignoreNulls(null)
Exception in thread "main" kotlin.KotlinNullPointerException at <...>.ignoreNulls(07_NotnullAssertions.kt:2)
```

되도록 쓰지 않는 편이 좋다.

### 6.1.7 `let` 함수

> `let` 함수는 **자신의 수신 객체를** 인자로 전달받은 **람다에게** 넘긴다. `null`이 될 수 있는 값에 대해 안전한 호출 구문을 사용해 `let`을 호출하되 `null`이 될 수 없는 타입을 인자로 받는 람다를 `let`에 전달한다. 이렇게 하면 `null`이 될 수 있는 타입의 값을 널이 될 수 없는 타입의 값으로 바꿔서 람다에 전달하게 된다.

- `let`을 사용해 `null`이 아닌 인자로 함수 호출하기

    ```kotlin
    fun sendEmailTo(email: String) {
        println("Sending email to $email")
    }
    >>> var email: String? = "yole@example.com"
    >>> email?.let { sendEmailTo(it) }  // 안전한 호출 구문을 사용해 
    Sending email to yole@example.com   // null이 될 수 없는 타입의 값을 let으로 넘긴다.
    >>> email = null
    >>> email?.let { sendEmailTo(it) }  // null 값을 넘기면 아무일도 하지 않는다.
    ```

### 6.1.8 나중에 초기화할 프로퍼티

> 코틀린에서는 클래스 안의 `null`이 될 수 없는 프로퍼티를 생성자 안에서 초기화하지 않고 특별한 메소드 안에서 초기화할 수는 없다. 코틀린에서는 일반적으로 생성자에서 모든 프로퍼티를 초기화해야 한다.

- 널 아님 단언을 사용해 널이 될 수 있는 프로퍼티 접근하기

    ```kotlin
    class MyService {
        fun performAction(): String = "foo"
    }
    class MyTest {
        private var myService: myService? = null    // null로 초기화하기 위해 널이 될 수 있는 타입인 프로퍼티를 선언
        @Before fun setUp() {
            myService = MyService() // setUp 메소드 안에서 진짜 초깃값을 지정
        }
        @Test fun testAction() {
            Assert.assertEquals("foo", myService!!.performAction()) // 반드시 널 가능성에 신경 써야 한다.
                                                                    // !!나 ?을 꼭 써야 한다.
        }
    }
    ```

- 코드를 간결하게 하기 위해 `myService` 프로퍼티를 **나중에 초기화<sup>late-initialized</sup>** 할 수 있다. `lateinit` 변경자를 붙이면 프로퍼티를 나중에 초기화할 수 있다.

    ```kotlin
    class MyService {
        fun performAction(): String = "foo"
    }
    class MyTest {
        private lateinit var myService: MyService   // 초기화하지 않고 널이 될 수 없는 프로퍼티를 선언
        @Before fun setUp() {
            myService = MyService() // 프로퍼티 초기화
        }
        @Test fun testAction() {
            Assert.assertEquals("foo", myService.performAction())   // 널 검사를 수행하지 않고 프로퍼티 사용
        }
    }
    ```

- 나중에 초기화하는 프로퍼티는 항상 `var`여야 한다.

### 6.1.9 널이 될 수 있는 타입 확장

- `null`이 될 수 있는 타입의 확장 함수는 안전한 호출 없이도 호출이 가능하다.

    ```kotlin
    fun verifyUserInput(input: String?) {
        if (input.isNullOrBlank()) {    // 안전한 호출을 하지 않아도 된다
            println("Please fill in the required fields")
        }
    }
    >>> verifyUserInput(" ")
    Please fill in the required fields
    >>> verifyUserInput(null)   // null을 수신 객체로 전달해도 아무런 예외가 발생하지 않는다.
    Please fill in the required fields
    ```

- `null`이 될 수 있는 타입의 값에 대해 안전한 호출을 사용하지 않고 `let`을 호출하면 람다의 인자는 `null`이 될 수 있는 타입으로 추론된다.

    ```kotlin
    >>> val person: Person? = ...
    >>> person.let { sendEmailTo(it) }
    ERROR: Type mismatch: inferred type is Person? but Person was expected
    ```

### 6.1.10 타입 파라미터의 널 가능성

> 코틀린에서는 함수나 클래스의 모든 타입 파라미터는 기본적으로 `null`이 될 수 있다.

타입 파라미터가 `null`이 아님을 확실히 하려면 `null`일 될 수 없는 타입 상한<sup>upper bound</sup>를 지정해야 한다.

```kotlin
fun <T: Any> printHashCode(t: T) {  // T는 null이 될 수 없는 타입이다.
    println(t.hashCode())
}
>>> printHashCode(null)
Error: Type parameter bound for 'T' is not satisfied
>>> printHashCode(42)
42
```

### 6.1.11 널 가능성과 자바

자바와 널 가능성과의 호환성은 자바에서의 어노테이션으로 어느정도 구분할 수 있으나 어노테이션으로 표시되지 않은 소스 코드에 대해서는 **플랫폼 타입**을 만들어 대응한다.

#### 플랫폼 타입

> 플랫폼 타입은 코틀린이 `null` 관련 정보를 알 수 없는 타입을 말한다. 그 타입을 널이 될 수 있는 타입으로 처리해도 되고 널이 될 수 없는 타입으로 처리해도 된다.

코틀린으로 가져오기전에 자바에서 코딩할 때 어노테이션 잘 붙여서 만들어라.

## 6.2 코틀린의 원시 타입

### 6.2.1 원시 타입: `Int`, `Boolean` 등

### 6.2.2 널이 될 수 있는 원시 타입: `Int?`, `Boolean?` 등

### 6.2.3 숫자 변환

### 6.2.4 `Any`, `Any?`: 최상위 타입

### 6.2.5 `Unit` 타입: 코틀린의 `void`

### 6.2.6 `Nothing` 타입: 이 함수는 결코 정상적으로 끝나지 않는다

## 6.3 컬렉션과 배열

### 6.3.1 널 가능성과 컬렉션

### 6.3.2 읽기 전용과 변경 가능한 컬렉션

### 6.3.3 코틀린 컬렉션과 자바

### 6.3.4 컬렉션을 플랫폼 타입으로 다루기

### 6.3.5 객체의 배열과 원시 타입의 배열
