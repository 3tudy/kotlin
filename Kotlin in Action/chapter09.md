# 9장 제네릭스

- 제네릭 함수와 클래스를 정의하는 방법
- 타입 소거와 실체화한 타입 파라미터
- 선언 지점과 사용 지점 변성

## 9.1 제네릭 타입 파라미터

> 제네릭스를 사용하면 **타입 파라미터**를 받는 타입을 정의할 수 있다. 제네릭 타입의 인스턴스를 만들려면 타입 파라미터를 구체적인 **타입 인자**로 치환해야 한다.

- 코틀린 컴파일러는 보통 타입과 마찬가지로 타입 인자도 추론할 수 있다.

    ```kotlin
    val authors = listOf("Dmitry", "Svetlana")
    ```

- 빈 리스트를 만들어야 한다면 타입 인자를 직접 명시해야 한다.

    ```kotlin
    // 1. 변수의 타입을 지정
    val readers: MutableList<String> = mutableListOf()

    // 2. 변수를 만드는 함수의 타입 인자를 지정
    val readers = mutableListOf<String>()
    ```

### 9.1.1 제네릭 함수와 프로퍼티

> 모든 리스트(제네릭 리스트)를 다룰 수 있는 제네릭 함수. 제네릭 함수를 호출할 때는 반드시 구체적인 타입으로 타입 인자를 넘겨야 한다.

```kotlin
fun <T> List<T>.slice(indices: IntRange): List<T>
```

- 함수의 타입 파라미터 `T`가 수신 객체와 반환 타입에 쓰인다.
- 수신 객체와 반환 타입 모두 `List<T>`다.

```kotlin
>>> val letters = ('a'..'z').toList()
>>> println(letters.slice<Char>(0..2))  // 타입 인자를 명시적으로 지정
[a, b, c]
>>> println(letters.slice(10..13))  // 컴파일러는 여기서 T가 Char라는 사실을 추론
[k, l, m, n]
```

- 제네릭 고차 함수 호출하기

    ```kotlin
    val authors = listOf("Dmitry", "Svetlana")
    val readers = mutableListOf<String>(/*...*/)
    fun <T> List<T>.filter(predicate: (T) -> Boolean): List<T>
    >>> readers.filter(it !in authors)
    ```

    - 람다 파라미터에 대해 자동으로 만들어지 변수 `it`의 타입은 `T`라는 제네릭 타입이다.
    - 컴파일러는 `filter`가 `List<T>` 타입의 리스트에 대해 호출될 수 있다는 사실과
    - `filter`의 수신 객체인 `reader`의 타입이 `List<String>`이라는 사실을 알고
    - 그로부터 `T`가 `String`이라는 사실을 추론한다.

- 클래스나 인터페이스 안에 정의된 메소드, 확장 함수 또는 최상위 함수에서 타입 파라미터를 선언 할 수 있다.
- 제네릭 함수를 정의할 때와 마찬가지 방법으로 제네릭 확장 프로퍼티를 선언할 수 있다.

    ```kotlin
    val <T> List<T>.penultimate: T  // 모든 리스트 타입에 제네릭 확장 프로퍼티를 사용할 수 있다.
        get() = this[size - 2]
    >>> println(listOf(1, 2, 3, 4).penultimate) // 타입 파라미터 T는 Int로 추론된다.
    3
    ```

### 9.1.2 제네릭 클래스 선언

> 자바와 마찬가지로 코틀린에서도 타입 파라미터를 넣은 꺽쇠 기호(<>)를 클래스 이름 뒤에 붙이면 클래스를 제네릭하게 만들 수 있다. 타입 파라미터를 이름 뒤에 붙이고 나면 클래스 본문 안에서 타입 파라미터를 다른 일반 타입처럼 사용할 수 있다.

```kotlin
interface List<T> { // List 인터페이스에 T라는 타입 파라미터를 정의한다.
    operator fun get(index: Int): T // 인터페이스 안에서 T를 일반 타입처럼 사용할 수 있다.
}
```

- 제네릭 클래스를 확장하는 클래스를 정의하려면 기반 타입의 제네릭 파라미터에 대해 타입 인자를 지정해야 한다.

    ```kotlin
    class StringList: List<String> {
        override fun get(index: Int): String = ...
    }
    class ArrayList<T>: List<T> {
        override fun get(index: Int): T = ...
    }
    ```

    - 하위 클래스에서 상위 클래스에 정의된 함수를 오버라이드하거나 사용하려면 타입 인자 `T`를 구체적 타입으로 치환해야 한다. 따라서  `fun get(Int): T`가 아니라 `fun get(Int): String`이라는 시그니처를 사용한다.

- 클래스가 자기 자신을 타입 인자로 참조할 수도 있다.

    ```kotlin
    interface Comparable<T> {
        fun compareTo(other: T): Int
    }
    class String: Comparable<String> {
        override fun compareTo(other: String): Int = /*...*/
    }
    ```

    - `String` 클래스는 제네릭 `Comparable` 인터페이스를 구현하면서 그 인터페이스 타입 파라미터 `T`로 `String` 자신을 지정한다.

### 9.1.3 타입 파라미터 제약

> **타입 파라미터 제약**은 클래스나 함수에 사용할 수 있는 타입 인자를 제한하는 기능이다.

- 어떤 타입을 제네릭 타입의 타입 파라미터에 대한 **상한**으로 지정하면 그 제네릭 타입을 인스턴스화할 때 사용하는 타입 인자는 반드시 그 상한 타입이거나 그 상한 타입의 하위 타입이어야 한다.
- 제약을 가하려면 타입 파라미터 이름 뒤에 콜론(:)을 표시하고 그 뒤에 상한 타입을 적으면 된다.

    ```kotlin
    // fun <타입 파라미터: 상한> List<T>.sum(): T
    fun <T: Number> List<T>.sum(): T

    /* java */
    <T extends Number> T sum(List<T> list)
    ```

- 타입 파라미터 `T`에 대한 상한을 정하고나면 `T` 타입의 값을 그 상한 타입의 값으로 취급할 수 있다.

    ```kotlin
    fun <T: Number> oneHalf(value: T): Double {
        return value.toDouble() / 2.0   // Number 클래스에 정의된 메소드를 호출할 수 있다.
    }
    >>> println(oneHalf(3))
    1.5
    ```

- 타입 파라미터를 제약하는 함수 선언하기

    ```kotlin
    fun <T: Comparable<T>> max(first: T, second: T): T {    // 이 함수들의 인자들은 비교 가능해야 한다.
        return if (first > second) first else second
    }
    >>> println(max("kotlin", "java"))
    kotlin
    ```

    - `T`의 상한 타입은 `Comparable<T>`다.
    - `String`이 `Comparable<String>`을 확장하므로 `String`은 `max` 함수에 적합한 타입 인자다.

- 타입 파라미터에 대해 둘 이상의 제약을 가해야 하는 경우

    ```kotlin
    fun <T> ensureTrailingPeriod(seq: T) where T: CharSequence, T: Appendable {
        if (!seq.endsWith('.')) {
            seq.append('.')
        }
    }
    >>> val helloWorld = StringBuilder("Hello World")
    >>> ensureTrailingPeriod(helloWorld)
    >>> println(helloWorld)
    Hello World.
    ```

    - 위 예제는 타입 인자가 `CharSequence`와 `Appendable` 인터페이스를 반드시 구현해야 한다는 사실을 표현한다.
    - 이는 데이터에 접근하는 연산(`endsWith`)과 테이터를 변환하는 연산(`append`)을 `T` 타입의 값에게 수행할 수 있다는 뜻이다.

### 9.1.4 타입 파라미터를 널이 될 수 없는 타입으로 한정

> 제네릭 클래스나 함수를 정의하고 그 타입을 인스턴스화할 때는 `null`이 될 수 있는 타입을 포함하는 어떤 타입으로 타입 인자를 지정해도 타입 파라미터를 치환할 수 있다. 아무런 상한을 정하지 않은 타입 파라미터는 결과적으로 `Any?`를 상한으로 정한 파라미터와 같다.

- `null` 가능성을 제외한 아무런 제약도 필요 없다면 `Any?`대신 `Any`를 상한으로 사용하면 된다.

## 9.2 실행 시 제네릭스의 동작: 소거된 타입 파라미터와 실체화된 타입 파라미터

> JVM의 제네릭스는 보통 **타입 소거**를 사용해 구현된다. 이는 실행 시점에 제네릭 클래스의 인스턴스에 타입 인자 정보가 들어있지 않다는 뜻이다.

### 9.2.1 실행 시점의 제네릭: 타입 검사와 캐스트

> 자바와 마찬가지로 코틀린 제네릭 타입 인자 정보는 런타임에 지워진다. 이는 제네릭 클래스 인스턴스가 그 인스턴스를 생성할 때 쓰인 타입 인자에 대한 정보를 유지하지 않는다는 뜻이다.

```kotlin
val list1: List<String> = listOf("a", "b")
val list2: List<Int> = listOf(1, 2, 3)
```

- **컴파일러**는 두 리스트를 서로 다른 타입으로 인식하지만 **실행 시점에 그 둘은 완전히 같은 타입의 객체**다.
- 컴파일러가 타입 인자를 알고 올바른 타입의 값만 각 리스트에 넣도록 보장해주기 때문에 `List`의 원소 타입을 추론할 수 있다.
- 타입 인자를 따로 저장하지 않기 때문에 실행 시점에 타입 인자를 검사할 수 없다. (**타입 소거**의 단점)

    ```kotlin
    >>> if (value is List<String>) {...}
    ERROR: Cannot check for instance of erased type
    ```

    - 실행 시점에 어떤 값이 `List`인지 여부는 확실히 알 수 있다.
    - 하지만, 그 리스트가 `String` 리스트인지, `Person` 리스트인지 혹은 다른 어떤 타입의 리스트인지는 알 수 없다.

- 어떤 값이 집합이나 맵이 아닌 리스트라는 사실을 **스타 프로젝션**을 이용해 체크할 수 있다.

    ```kotlin
    if (value is List<*>) {...}
    ```

    - 타입 파라미터가 2개 이상이라면 모든 타입 파라미터에 `*`를 포함시켜야 한다.

- `as`나 `as?` 캐스팅에도 여전히 제네릭 타입을 사용할 수 있다. 하지만 기저 클래스는 같지만 타입 인자가 다른 타입으로 캐스팅해도 여전히 캐스팅에 성공한다.

    ```kotlin
    fun printSum(c: Collection<*>) {
        val intList = c as? List<Int> ?: throw IllegalArgumentException("List is expected") // 컴파일러가 Unchecked cast: List<*> to List<Int> 경고를 발생시킴
        println(inList.sum())
    }
    >>> printSum(listOf(1, 2, 3))   // 동작은 함
    6
    >>> printSum(setOf(1, 2, 3))    // 집합은 리스트가 아니므로 예외 발생
    IllegalArgumentException: List is expected
    >>> printSum(listOf("a", "b", "c")) // as? 캐스팅은 성공하지만 나중에 다른 예외가 발생 - case ClassCastException
    ClassCastException: String cannot be cast to Number
    ```

    - case `ClassCastException`
      - 어떤 값이 `List<Int>`인지 검사할 수는 없으므로 `IllegalArgumentException`이 발생하지는 않는다.
      - `sum`은 `Number` 타입의 값을 리스트에서 가져와 더하려고 시도하지만 `String`을 `Number`로 사용하려 하기에 `ClassCastException`이 발생한다.

- 타입 정보가 주어진 경우에는 `is` 검사를 수행할 수 있다.

    ```kotlin
    fun printSum(c: Collection<Int>) {
        if (c is List<Int>) {
            println(c.sum())
        }
    }
    >>> printSum(listOf(1, 2, 3))
    6
    ```

    - 컴파일 시점에 `c` 컬렉션이 `Int` 값을 저장한다는 사실이 알려져 있으므로 `c`가 `List<Int>`인지 검사할 수 있다.

### 9.2.2 실체화한 타입 파라미터를 사용한 함수 선언

> 코틀린의 제네릭 타입의 타입 인자 정보는 실행 시점에 지워진다. 따라서 제네릭 클래스의 인스턴스가 있어도 그 인스턴스를 만들 때 사용한 타입 인자를 알아낼 수 없다. **제네릭 함수의 타입 인자도 제네릭 함수가 호출되도 그 함수의 본문에서는 호출시 쓰인 타입 인자를 알 수 없다.**

```kotlin
>>> fun <T> isA(value: Any) = value is T
Error: Cannot check for instance of erased type: T
```

- 인라인 함수의 타입 파라미터는 실체화되므로 실행 시점에 인라인 함수의 타입 인자를 알 수 있다.

    ```kotlin
    inline fun <refied T> isA(value: Any) = value is T
    >>> println(isA<String>("abc"))
    true
    >>> println(isA<String>(123))
    false
    ```

    - `isA` 함수를 인라인 함수로 만들고 타입 파라미터를 `reified`로 지정하면 `value`의 타입이 `T`의 인스턴스인지를 실행 시점에 검사할 수 있다.

- 실체화한 타입 파라미터 활용

    ```kotlin
    // 인자로 받은 컬렉션의 원소 중에서
    // 타입 인자로 지정한 클래스의 인스턴스만을 모아서 만든
    // 리스트를 반환하는 filterIsInstance
    >>> val items = listOf("one", 2, "three")
    >>> println(items.filterIsInstance<String>())
    [one, three]
    ```

    - `filterIsInstance`의 타입 인자로 `String`을 지정해 함수의 반환 타입을 `List<String>`으로 추론할 수 있게한다.
    - 따라서 타입 인자를 실행 시점에 알 수 있고 `filterIsInstance`는 그 타입 인자를 사용해 리스트의 원소 중에 인자와 타입이 일치하는 원소만을 추려낼 수 있다.

### 9.2.3 실체화한 타입 파라미터로 클래스 참조 대신

> `java.lang.Class` 타입 인자를 파라미터로 받는 API에 대한 코틀린 어댑터를 구축하는 경우 실체화한 타입 파라미터를 자주 사용한다.

- `ServiceLoader`는 어떤 추상 클래스나 인터페이스를 표현하는 `java.lang.Class`를 받아서 그 클래스나 인스턴스를 구현한 인스턴스를 반환한다.

    ```kotlin
    val serviceImpl = ServiceLoader.load(Service::class.java)
    ```

    - `Service::class.java`라는 코드는 `Service.class`라는 자바 코드와 완전히 같다.

솔직히 모르겠다. 자바에서 `ServiceLoader`를 사용하는 예를 좀 더 찾아봐야겠다.

### 9.2.4 실체화한 타입 파라미터의 제약

실체화한 타입 파라미터를 사용할 수 있는 경우

- 타입 검사와 캐스팅(`is`, `!is`, `as`, `as?`)
- 10장에서 설명할 코틀린 리플렉션 API(`::class`)
- 코틀린 타입에 대응하는 `java.lang.Class`를 얻기(`::class.java`)
- 다른 함수를 호출할 때 타입 인자로 사용

실체화한 타입 파라미터가 하지 못하는 잡업

- 타입 파라미터 클래스의 인스턴스 생성하기
- 타입 파라미터 클래스의 동반 객체 메소드 호출하기
- 실체화한 타입 파라미터를 요구하는 함수를 호출하면서 실체화하지 않은 타입 파라미터로 받은 타입을 타입 인자로 넘기기
- 클래스, 프로퍼티, 인라인 함수가 아닌 함수의 타입 파라미터를 `reified`로 지정하기

> 실체화한 타입 파라미터를 인라인 함수에만 사용할 수 있으므로 실체화한 타입 파라미터를 사용하는 함수는 자신에게 전달되는 모든 람다와 함께 인라이닝된다.

- 람다 내부에서 타입 파라미터를 사용하는 방식에 따라서는 `noinline` 변경자를 함수 타입 파라미터에 붙여 인라이닝을 금지할 수 있다.

## 9.3 변성: 제네릭과 하위 타입

> **변성<sup>variance</sup>** 개념은 `List<String>`와 `List<Any>`와 같이 기저 타입이 같고 인자가 다른 여러 타입이 서로 어떤 관계가 있는지 설명하는 개념이다.

### 9.3.1 변성이 있는 이유: 인자를 함수에 넘기기

- `MutableList<Any>`가 필요한 곳에 `MutableList<String>`을 넘기면 안된다.

    ```kotlin
    fun addAnswer(list: MutableList<Any>) {
        list.add(42)
    }
    >>> val strings = mutableListOf("abc", "bac")
    >>> addAnswer(strings)
    >>> println(strings.maxBy { it.length })    // 실행 시점에 예외가 발생한다.
    ClassCastException: Integer cannot be cast to String
    ```

### 9.3.2 클래스, 타입, 하위 타입

1. 제네릭 클래스가 아닌 클래스에서는 클래스 이름을 바로 타입으로 쓸 수 있다.
2. 제네릭 클래스에서는 올바른 타입을 얻으려면 제네릭 타입의 타입 파라미터를 구체적인 타입 인자로 바꿔줘야 한다.
3. 어떤 타입 `A`의 값이 필요한 모든 장소에 어떤 타입 `B`의 값을 넣어도 아무 문제가 없다면 `B`는 타입 `A`의 **하위 타입**이다. (`B`가 `A`보다 구체적)
4. **상위 타입**은 하위 타입의 반대다.
5. **하위 타입**은 **하위 클래스**와 근본적으로 같다.
6. 제네릭 타입을 인스턴스화 할 때 타입 인자로 서로 다른 타입이 들어가면 인스턴스 타입 사이의 하위 타입 관계가 성립하지 않으면 그 제네릭 타입을 **무공변**<sup>invariant</sup>이라고 한다.
7. 코틀린의 `List` 인터페이스는 일기 전용 컬렉션을 표현한다. `A`가 `B`의 하위 타입이면 `List<A>`는 `List<B>`의 하위 타입이다. 그런 클래스나 인터페이스를 **공변적**<sup>covariant</sup>이라고 한다.

### 9.3.3 공변성: 하위 타입 관계를 유지

> `Producer<T>`를 예로 공변성 클래스를 설명하자. `A`가 `B`의 하위 타입일 때 `Producer<A>`가 `Producer<B>`의 하위 타입이면 `Producer`는 공변적이다. 이를 하위 타입 관계가 유지된다고 말한다.

- 코틀린에서 제네릭 클래스가 타입 파라미터에 대해 공변적임을 표시하려면 타입 파라미터 이름 앞에 `out`을 넣어야 한다.

    ```kotlin
    interface Producer<out T> { // 클래스가 T에 대해 공변적이라고 선언한다.
        fun produce(): T
    }
    ```

    - 클래스의 타입 파라미터를 공변적으로 만들면 함수 정의에 사용한 파라미터 타입과 타입 인자의 타입이 정확히 일치하지 않더라고 그 클래스의 인스턴스를 함수 인자나 반환 값으로 사용할 수 있다.

- `T`가 함수 반환 타입에 쓰인다면 `T`는 아웃<sub>out</sub> 위치에 있다.
- `T`가 함수 파라미터 타입에 쓰인다면 `T`는 인<sub>in</sub> 위치에 있다. 파라미터 타입 `T`의 값은 소비<sub>consume</sub>된다.
- 클래스 타입 파라미터 `T` 앞에 `out` 키워드를 붙이면 클래스 안에서 `T`를 사용하는 메소드가 **아웃** 위치에서만 `T`를 사용하게 허용하고 **인** 위치에서는 `T`를 사용하지 못하게 막는다.

    ```kotlin
    class Herd<out T: Animal> {
        val size: Int get() = ...
        operator fun get(i: Int): T {...}   // T를 반환 타입으로 사용한다.
    }
    ```

- 타입 파라미터 `T`에 붙은 `out` 키워드의 의미
  1. **공변성**: 하위 타입 관계가 유지된다.
  2. **사용 제한**: `T`를 아웃 위치에서만 사용할 수 있다.

### 9.3.4 반공변성: 뒤집힌 하위 타입 관계

> 반공변 클래스의 하위 타입 관계는 공변 클래스의 경우와 반대다.

```kotlin
interface Comparator<in T> {
    fun compare(e1: T, e2: T): Int {...}    // T를 in 위치에 사용
}
```

- 위 인터페이스의 메소드는 `T` 타입의 값을 **소비**하기만 한다.

> `Consumer<T>`를 예로 들어 설명하자. 타입 `B`가 타입 `A`의 하위 타입인 경우 `Consumer<A>`가 `Consumer<B>`의 하위 타입인 관계가 성립하면 제네릭 클래스 `Consumer<T>`는 타입 인자 `T`에 대해 반공변이다.

### 9.3.5 사용 지점 변성: 타입이 언급되는 지점에서 변성 지정

1. 클래스를 선언하면서 변성을 지정하는 방식을 **선언 지점 변성**이라고 한다.
2. 타입 파라미터가 있는 타입을 사용할 때마다 해당 타입 파라미터를 하위 타입이나 상위 타입 중 어떤 타입으로 대치할 수 있는지 명시하는 방식을 **사용 지점 변성**이라고 한다.

- 클래스 안에서 어떤 타입 파라미터가 공변적이거나 반공변적인지 선언할 수 없는 경우에도 특정 타입 파라미터가 나타나는 지점에서 변성을 정할 수 있다.

    ```kotlin
    fun <T> copyData(source: MutableList<T>, destination: MutableList<T>) {
        for (item in source) {
            destination.add(item)
        }
    }
    ```

    - 이 함수는 컬렉션의 원소를 다른 컬렉션으로 복사한다.
    - 두 컬렉션 모두 무공변 타입이지만 원본 컬렉션에서는 읽기만 하고 대상 컬렉션에는 쓰기만 한다.
    - 이 경우 두 컬렉션의 원소 타입이 정확하게 일치할 필요가 없다.

- 위 함수를 여러 다른 리스트 타입에 대해 작동하게 만들기

    ```kotlin
    fun <T: R, R> copyData(source: MutableList<T>, // source 원소 타입은 destination 원소 타입의 하위 타입이어야 한다.
                           destination: MutableList<R>) {
        for (item in source) {
            destination.add(item)
        }
    }
    >>> val ints = mutableListOf(1, 2, 3)
    >>> val anyItems = mutableListOf<Any>()
    >>> copyData(ints, anyItems)    // Int가 Any의 하위 타입이므로 이 함수를 호출할 수 있다.
    >>> println(anyItems)
    [1, 2, 3]
    ```

    - 두 타입 파라미터는 원본과 대상 리스트의 원소 타입을 표현한다.

- 함수의 구현이 아웃 위치(or 인 위치)에 있는 타입 파라미터를 사용하는 메소드만 호출한다면 그런 정보를 바탕으로 함수 정의 시 타입 파라미터에 변성 변경자를 추가할 수 있다.

    ```kotlin
    // 아웃-프로젝션 타입 파라미터를 사용하는 데이터 복사 함수
    fun <T> copyData(source: MutableList<out T>,    // out 키워드를 타입을 사용하는 위치 앞에
                     destination: MutableList<T>) { // 붙이면 T 타입을 in 위치에 사용하는
                                                    // 메소드를 호출하지 않는다는 뜻이다.
        for (item in source) {
            destination.add(item)
        }
    }
    ```

    ```kotlin
    // in 프로젝션 타입 파라미터를 사용하는 데이터 복사 함수
    fun <T> copyData(source: MutableList<T>,
                     destination: MutableList<in T>) {  // 원본 리스트 원소 타입의 상위 타입을
                                                        // 대상 리스트 원소 타입으로 허용한다.
        for (item in source) {
            destination.add(item)
        }                     
    }
    ```

### 9.3.6 스타 프로젝션: 타입 인자 대신 `*` 사용

> 제네릭 타입 인자 정보가 없을 표현하기 위해 **스타 프로젝션**을 사용한다. 예를 들어 원소 타입이 알려지지 않은 리스트는 `List<*>`라는 구문으로 표현할 수 있다.

1. `MutableList<*>`는 `MutableList<Any?>`와 같지 않다.
   - `MutableList<*>`는 어떤 정해진 구체적인 타입의 원소만을 담는다.
   - `MutableList<Any?>`는 모든 타입의 원소를 담을 수 있다.
2. 컴파일러는 `MutableList<*>`를 **아웃 프로젝션 타입**으로 인식한다.

    ```kotlin
    >>> val list: MutableList<Any?> = MutableListOf('a', 1. "qwe")
    >>> val chars = mutableListOf('a', 'b', 'c')
    >>> val unknownElements: MutableList<*> = if (Random().nextBoolean()) list else chars
    >>> unknownElements.add(42)
    Error: Out-projected type 'MutableList<*>` prohibits the use of 'fun add(element: E): Boolean'
    >>> println(unknownElements.first())
    a
    ```

3. 타입 파라미터를 시그니처에서 전혀 언급하지 않거나 데이터를 읽기는 하지만 그 타입에는 관심이 없는 경우와 같이 타입 인자 정보가 중요하지 않을 때도 스타 프로젝션 구문을 사용할 수 있다.

    ```kotlin
    fun printFirst(list: List<*>) { // 모든 리스트를 인자로 받을 수 있다.
        if (list.isNotEmpty()) {    // isNotEmpty()에서는 제네릭 타입 파라미터를 사용하지 않는다.
            println(list.first())   // first()는 이제 Any?를 반환하지만 여기서는 그 타입만으로 충분하다.
        }
    }
    >>> printFirst(listOf("Svetlana", "Dmitry"))
    Svetlana
    ```
