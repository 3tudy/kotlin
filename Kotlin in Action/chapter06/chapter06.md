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

> 코틀린은 원시 타입과 래퍼 타입을 구분하지 않는다.

### 6.2.1 원시 타입: `Int`, `Boolean` 등

> 자바는 원시 타입과 참조 타입을 구분한다. 자바는 참조 타입이 필요한 경우 특별한 래퍼 타입으로 원시 타입 값을 감싸서 사용한다.

- 코틀린은 원시 타입과 래퍼 타입을 구분하지 않으므로 항상 같은 타입을 사용한다.

    ```kotlin
    val i: Int = 1
    val list: List<Int> = listOf(1, 2, 3)
    ```

- 코틀린은 숫자 타입 등 원시 타입의 값에 대해 메소드를 호출할 수 있다.

    ```kotlin
    fun showProgress(progress: Int) {
        val percent = progress.coerceIn(0, 100)
        println("We're ${percent}% done!")
    }
    >>> showProgress(146)
    We're 100% done!
    ```

> 실행 시점에 숫자 타입은 가능한 한 가장 효율적인 방식으로 표현된다. 대부분의 경우 코틀린의 `Int` 타입은 자바 `int` 타입으로 컴파일된다. 하지만 컬렉션과 같은 제네릭 클래스를 사용할 때는 `Int` 타입을 **컬렉션의 타입 파라미터**로 넘기면 그 컬렉션에서는 `Int`의 래퍼 타입인 `java.lang.Integer` 객체가 들어간다.

- 코틀린의 원시 타입은 `null` 참조가 들어갈 수 없기 때문에 그에 상응하는 자바 원시 타입으로 컴파일 할 수 있다.
- 마찬가지로 자바 원시 타입의 값은 결코 `null`이 될 수 없으므로 자바 원시 타입을 코틀린에서 사용할 때도 `null`이 될 수 없는 타입으로 취급할 수 있다.

### 6.2.2 널이 될 수 있는 원시 타입: `Int?`, `Boolean?` 등

> 코틀린에서 `null`이 될 수 있는 원시 타입을 사용하면 그 타입은 자바의 래퍼 타입으로 컴파일된다.

### 6.2.3 숫자 변환

> 코틀린은 한 타입의 숫자를 다른 타입의 숫자로 자동 변환하지 않는다. 결과 타입이 허용하는 숫자의 범위가 원래 타입의 범위보다 얿은 경우조차도 자동 변환은 불가능하다.

```kotlin
val i = 1
val l: Long = i // Error: type mismatch

val i = 1
val l: Long = i.toLong()    // 직접 변환 메소드를 호출하여 형변환이 가능하다.
```

- 코틀린 산술 연산자에서도 자바와 똑같이 숫자 연산 시 오버플로우가 발생할 수 있으나 자동 형변환을 허용하지 않기에 오버플로우를 검사하느라 추가 비용을 들이지는 않는다.

### 6.2.4 `Any`, `Any?`: 최상위 타입

> 자바에서 `Object`가 클래스 계층의 최상위 타입이듯 코틀린에서는 `Any` 타입이 모든 `null`이 될 수 없는 타입의 조상 타입이다. 코틀린에서는 `Any`가 `Int` 등의 원시 타입을 포함한 모든 타입의 조상 타입이다.

```kotlin
val answer: Any = 42    // Any가 참조 타입이기 때문에 42가 박싱된다.
                        // Any는 널이 될 수 없는 타입이다.
                        // 널이 될 수 있는 타입은 Any?를 사용해야 한다.
```

### 6.2.5 `Unit` 타입: 코틀린의 `void`

> 코틀린 `Unit` 타입은 자바 `void`와 같은 기능을 한다.

```kotlin
fun f(): Unit { ... }

fun f() { ... } // 반환 타입을 명시하는 것을 생략할 수 있다.
```

> `Unit`은 모든 기능을 같는 일반적인 타입이며, `void`와 달리 `Unit`을 타입 인자로 쓸 수 있다.`Unit` 타입에 속한 값은 단 하나뿐이며, 그 이름도 `Unit`이다. `Unit` 타입의 함수는 `Unit` 값을 묵시적으로 반환한다.

```kotlin
interface Processor<T> {
    fun process(): T
}
class NoResultProcessor: Processor<Unit> {
    override fun process() {    // Unit을 반환하지만 타입을 지정할 필요는 없다.
        // 업무 처리 코드
        // return을 명시할 필요가 없다.
    }
}
```

### 6.2.6 `Nothing` 타입: 이 함수는 결코 정상적으로 끝나지 않는다

> `Nothing` 타입은 아무 값도 포함하지 않는다. 따라서 `Nothing`은 **함수의 반환 타입**이나 **반환 타입으로 쓰일 타입 파라미터**로만 쓸 수 있다.

```kotlin
fun fail(message: String): Nothing {
    throw IllegalStateException(message)
}
>>> fail("Error occurred")
java.lang.IllegalStateException: Error occurred
```

- `Nothing`을 반환하는 함수를 엘비스 연산자의 우항에 사용해서 전제 조건을 검사할 수 있다.

    ```kotlin
    val address = company.address ?: fail("No address")
    println(address.city)
    ```

## 6.3 컬렉션과 배열

### 6.3.1 널 가능성과 컬렉션

```kotlin
List<Int?>  // 리스트 자체는 항상 null이 아니다.
            // 각 원소는 null이 될 수 있다.

List<Int>?  // 리스트를 가리키는 변수에는 null이 들어갈 수 있다.
            // 각 원소는 null이 아닌 값만 들어갈 수 있다.
```

- `null`이 될 수 있는 원소를 가지는 컬렉션은 각 원소가 `null`인지를 항상 검사해야 한다.

    ```kotlin
    fun addValidNumbers(numbers: List<Int?>) {
        var sumOfValidNumbers = 0
        var invalidNumbers = 0
        for (number in numbers) {   // null이 될 수 있는 값을 읽는다.
            if (number != null) {   // null에 대한 값을 확인한다.
                sumOfValidNumbers += number
            } else {
                invalidNumbers++
            }
        }
        println("Sum of valid numbers: $sumOfValidNumbers)
        println("Invalid numbers: $invalidNumbers")
    }
    >>> val reader = BufferedReader(StringReader("1\nabc\n42"))
    >>> val numbers = readNumbers(reader)
    >>> addValidNumbers(numbers)
    Sum of valid numbers: 43
    Invalid numbers: 1
    ```

### 6.3.2 읽기 전용과 변경 가능한 컬렉션

> 코틀린 컬렉션과 자바 컬렉션을 나누는 가장 중요한 특성 하나는 코틀린에서는 컬렉션 안의 **데이터에 접근하는 인터페이스**와 컬렉션 안의 **데이터를 변경하는 인터페이스**를 **분리**했다는 점이다.

- `kotlin.collections.Collection` 인터페이스를 사용하면 컬렉션 안의 데이터에 **접근**은 가능하지만 변경은 할 수 없다.
- `kotlin.collections.MutableCollection` 인터페이스를 사용해 컬렉션 안의 데이터를 **수정**할 수 있다.
- `MutableCollection`은 일반 인터페이스는 `kotlin.collections.Collection`을 확장하면서 원소를 추가하거나, 삭제하거나, 컬렉션 안의 원소를 모두 지우는 등의 메소드를 더 제공한다.
- 코드에서는 가능하면 항상 **읽기 전용** 인터페이스를 사용하는 것이 좋다.

```kotlin
fun <T> copyElements(source: Collection<T>, target: MutableCollection<T>) {
    for (item in source) {
        target.add(item)
    }
}
>>> val source: Collection<Int> = arrayListOf(3, 5, 7)
>>> val target: MutableCollection<Int> = arrayListOf(1)
>>> copyElements(source, target)
>>> println(target)
[1, 3, 5, 7]

// target에 읽기 전용 컬렉션을 넘기면 컴파일 오류가 발생한다.
>>> val source: Collection<Int> = arrayListOf(3, 5, 7)
>>> val target: Collection<Int> = arrayListOf(1)
>>> copyElements(source, target)
Error: Type mismatch: inferred type is Collection<Int>
    but MutableCollection<Int> was expected
```

- 같은 컬렉션 객체를 다른 타입의 참조들이 가리킬 수 있다.

### 6.3.3 코틀린 컬렉션과 자바

> 코틀린은 모든 자바 컬렉션 인터페이스마다 읽기 전용 인터페이스와 변경 가능한 인터페이스라는 두가지 표현을 제공한다.

| 컬렉션 타입 | 읽기 전용 타입 | 변경 가능 타입 |
|---|---|---|
| `List` | `listOf` | `mutableListOf`, `arrayListOf` |
| `Set` | `setOf` | `mutableSetOf`, `hashSetOf`, `linkedSetOf`, `sortedSetOf` |
| `Map` | `mapOf` | `mutableMapOf`, `hashMapOf`, `linkedMapOf`, `sortedMapOf` |

- 변경 불가능한 컬렉션 타입을 넘겨도 자바 쪽에서 내용을 변경할 수 있다.

    ```kotlin
    /* java code */
    // CollectionUtils.java
    public class CollectionUtils {
        public static List<String> uppercaseAll(List<String> items) {
            for (int i=0; i<items.size(); i++) {
                items.set(i, items.get(i).toUpperCase());
            }
            return items;
        }
    }

    /* kotlin code */
    // collections.kt
    fun printInUppercase(list: List<String>) {
        println(CollectionsUtils.uppercaseAll(list))
        println(list.first())
    }
    >>> val list = listOf("a", "b", "c")
    >>> printInUppercase(list)
    [A, B, C]
    A
    ```

### 6.3.4 컬렉션을 플랫폼 타입으로 다루기

> 자바쪽에서 선언한 컬렉션 타입의 변수를 코틀린에서는 플랫폼 타입으로 본다. 코틀린 코드는 그 타입을 읽기 전용 컬렉션이나 변경 가능한 컬렉션 어느 쪽으로든 다룰 수 있다.

컬렉션의 타입을 결정하기 위한 고려사항

- 컬렉션이 `null`이 될 수 있는가?
- 컬렉션의 원소가 `null`이 될 수 있는가?
- 오버라이드하는 메소드가 컬렉션을 변경할 수 있는가?

### 6.3.5 객체의 배열과 원시 타입의 배열

코틀린에서 배열을 만드는 방법

- `arrayOf` 함수에 원소를 넘긴다.
- `arrayOfNulls` 함수에 정수 값을 인자로 넘기면 모든 원소가 `null`이고 인자로 넘긴 값과 크기가 같은 배열을 만들 수 있다.
- `Array` 생성자는 배열 크기와 람다를 인자로 받아서 람다를 호출해서 각 배열 원소를 초기화해준다.

    ```kotlin
    >>> val letters = Array<String>(26) { i -> ('a' + i).toString() }
    >>> println(letters.joinToString(""))
    abcdefghijklmnopqrstuvwxyz
    ```

- 코틀린은 원시 타입의 배열을 표현하는 별도 클래스를 각 원시 타입마다 하나씩 제공한다.

    | Java | Kotlin |
    |:---:|:---:|
    | `int[]` | `IntArray` |
    | `byte[]` | `ByteArray` |
    | `char[]` | `CharArray` |
