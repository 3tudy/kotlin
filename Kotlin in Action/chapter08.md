# 8장 고차 함수: 파라미터와 반환 값으로 람다 사용

- 함수 타입
- 고차 함수와 코드를 구조화할 때 고차 함수를 사용하는 방법
- 인라인 함수
- 비로컬 `return`과 레이블
- 무명 함수

## 8.1 고차 함수 정의

> **고차 함수**는 다른 함수를 인자로 받거나 함수를 반환하는 함수다. 고차 함수는 람다나 함수 참조를 인자로 넘길 수 있거나 람다나 함수 참조를 반환하는 함수다. 함수를 인자로 받는 동시에 함수를 반환하는 함수도 고차 함수다.

*고차 함수를 이용해서 python의 데코레이터 문법 비슷한 걸 만들 수 있을거 같다.*

### 8.1.1 함수 타입

- 함수의 타입을 정의하려면 **함수 파라미터의 타입을 괄호 안에** 넣고, 그 뒤에 **화살표(->)** 를 추가한 다음, 함수의 **반환 타입을 지정**하면 된다.

    ```kotlin
    val sum = { x: Int, y: Int -> x + y }
    val sum: (Int, Int) -> Int = { x, y -> x + y }

    val action = { println(42) }
    val action: () -> Unit = { println(42) }    // 함수 타입을 선언할 때는 
                                                // 타입을 반드시 명시해야 하므로
                                                // Unit을 필수로 넣어줘야 한다.
    ```

- 다른 함수와 마찬가지로 함수 타입에서도 반환 타입을 `null`이 될 수 있는 타입으로 지정할 수 있다.

    ```kotlin
    var canReturnNull: (Int, Int) -> Int? = {x, y -> null}
    ```

- 함수 자체를 `null`이 될 수 있는 타입으로 선언할 수 있다.

    ```kotlin
    var funOrNull: ((Int, Int) -> Int)? = null  // 함수 타입을 괄호로 감싸고
                                                // 그 뒤에 물음표를 붙인다.
    ```

#### 파라미터 이름과 함수 타입

- 함수 타입에서 파라미터 이름을 지정할 수도 있다.

    ```kotlin
    fun performRequest(url: String, callback: (code: Int, content: String) -> Unit) {
        /*...*/
    }
    >>> val url = "http://kotl.in"
    >>> performRequest(url) {code, content -> /*...*/}  // API에서 제공하는 이름을 람다에 사용할 수 있다.
    >>> performRequest(url) {code, page -> /*...*/} // 원하는 이름을 붙일 수도 있다.
    ```

- 파라미터 이름은 타입 검사 시 무시된다.
- 이 함수 타입의 람다를 정의할 때 파라미터 이름이 꼭 함수 타입 선언의 파라미터 이름과 일치하지 않아도 된다.

### 8.1.2 인자로 받은 함수 호출

```kotlin
fun twoAndThree(operation: (Int, Int) -> Int) { // 함수 타입은 파라미터 선언
    val result = operation(2, 3)    // 함수 타입인 파라미터 호출
    println("The result is $result")
}
>>> twoAndThree {a, b -> a + b}
The result is 5
>>> twoAndThree {a, b -> a * b}
The result is 6
```

### 8.1.3 자바에서 코틀린 함수 타입 사용

> 컴파일된 코드 안에서 함수 타입은 일반 인터페이스로 바뀐다. 즉, 함수 타입의 변수는 `FunctionN` 인터페이스를 구현하는 객체를 저장한다.

- 코틀린 표준 라이브러리는 함수 인자의 개수에 따라 `Function0<R>`(인자가 없는 함수), `Function1<P1, R>`(인자가 하나인 함수) 등의 인터페이스를 제공한다.
  - 각 인터페이스에는 `invoke` 메소드 정의가 하나 들어있다.
  - `invoke`를 호출하면 함수를 실행할 수 있다.
  - 함수 타입인 변수는 인자 개수에 따라 적당한 `FunctionN` 인터페이스를 구현하는 클래스의 인스턴스를 저장한다.
  - 그 클래스의 `invoke` 메소드 본문에는 람다의 본문이 들어간다.

- 함수 타입을 사용하는 코틀린 함수를 자바에서도 쉽게 호출할 수 있다. 자바 8 람다를 넘기면 자동으로 함수 타입의 값으로 변환된다.

    ```kotlin
    /* 코틀린 선언 */
    fun processTheAnswer(f: (Int) -> Int) {
        println(f(42))
    }

    /* 자바 */
    >>> processTheAnswer(number -> number + 1);
    43
    ```

- 자바 8 이전의 자바에서는 필요한 `FunctionN` 인터페이스의 `invoke` 메소드를 구현하는 무명 클래스를 넘기면 된다.

    ```kotlin
    >>> processTheAnswer(
    ...     new Function1<Integer, Integer>() {
    ...         @override
    ...         public Integer invoke(Integer, number) {
    ...             System.out.println(number);
    ...             return number + 1;
    ...         }
    ...     }
    ...);
    43
    ```

- 자바에서 코틀린 표준 라이브러리가 제공하는 람다를 인자로 받는 확장 함수를 쉽게 호출할 수 있다. 하지만 수신 객체를 확장 함수의 첫 번째 인자로 명시적으로 넘겨야 한다.

    ```kotlin
    /* 자바 */
    >>> List<String> strings = new ArrayList();
    >>> strings.add("42");
    >>> CollectionsKt.forEach(strings, s -> {
    ...    System.out.println(s);
    ...    return Unit.INSTANCE;    // Unit 타입의 값을 명시적으로 반환해야 한다.
    ...});
    ```

### 8.1.4 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터

> 파라미터를 함수 타입으로 선언할 때도 디폴트 값을 정할 수 있다.

```kotlin
fun <T> Collection<T>.joinToString(
    seperator: String = ", ",
    prefix: String = "",
    postfix: String  "",
    transform: (T) -> String = { it.toString() }    // 함수 타입 파라미터를 선언하면서
                                                    // 람다를 디폴트 값으로 지정한다.
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(transform(element))
    }
    result.append(postfix)
    return result.toString()
}
>>> val letters = listOf("Alpha", "Beta")
>>> println(letters.joinToString())
Alpha, Beta
>>> println(letters.joinToString { it.toLowerCase() })  // 람다를 인자로 전달
alpha, beta
>>> println(letters.joinToString(separator="! ", postfix="! ", transform={it.toUpperCase()}))
ALPHA! BETA!
```

- `null`이 될 수 있는 함수 타입을 사용할 수도 있다. 하지만 `null`이 될 수 있는 타입의 함수는 직접 호출될 수 없다. 따라서 `null` 여부를 **명시적으로 검사**해주거나 **안전 호출**을 이용해야 한다.

    ```kotlin
    fun <T> Collection<T>.joinToString(
        separator: String=", ",
        prefix: String="",
        postfix: String="",
        transform: ((T) -> String)? = null
    ): String {
        val result = StringBuilder(prefix)
        for ((index, element) in this.withIndex()) {
            if (index > 0) result.append(separator)
            val str = transform?.invoke(element)    // 안전 호출을 사용해 함수를 호출한다.
                ?: element.toString()   // 엘비스 연산자를 사용해 람다를 인자로 받지 않은 경우를 처리한다.
            result.append(str)
        }
        result.append(postfix)
        return result.toString()
    }
    ```

### 8.1.5 함수를 함수에서 반환

- 다른 함수를 반환하는 함수를 정의하려면 **함수의 반환 타입으로 함수 타입을 지정해야 한다**.

    ```kotlin
    enum class Delivery { STANDARD, EXPECITED }
    class Order(val itemCount: Int)
    fun getShippingCostCalculator(delivery: Delivery): (Order) -> Double {  // 함수를 반환하는 함수를 선언
        if (delivery == Delivery.EXPEDITED) {
            return { order -> 6 + 2.1 * order.itemCount }   // 람다 반환
        }
        return { order -> 1.2 * order.itemCount }   // 람다 반환
    }
    // val calculator = {order: Order -> 6 + 2.1 * order.itemCount}
    >>> val calculator = getShippingCostCalculator(Delivery.EXPEDITED)
    >>> println("Shipping costs ${calculator(Order(3))}")
    Shipping costs 12.3
    ```

### 8.1.6 람다를 활용한 중복 제거

> 람다를 사용하면 코드 중복을 줄일 수 있다.

## 8.2 인라인 함수: 람다의 부가 비용 없애기

> `inline` 변경자를 어떤 함수에 붙이면 컴파일러는 그 함수를 호출하는 모든 문장을 함수 본문에 해당하는 바이트코드로 바꿔치기 해준다.

### 8.2.1 인라이닝이 작동하는 방식

> 어떤 함수를 `inline`으로 선언하면 그 함수의 본문이 인라인<sup>inline</sup>된다. **함수를 호출하는 코드**를 **함수를 호출하는 바이트코드 대신에** 함수 본문을 **번역한 바이트코드로 컴파일한다**는 뜻이다.

```kotlin
inline fun <T> synchronized(lock: Lock, action: () -> T): T {
    lock.lock()
    try {
        return action()
    }
    finally {
        lock.unlock()
    }
}
val l = Lock()
synchronized(l) {
    // ...
}
```

- `synchronized` 함수를 `inline`으로 선언했으므로 `synchronized`를 호출하는 코드는 모두 자바의 `synchronized`문과 같아진다.

    ```kotlin
    fun foo(l: Lock) {
        println("Before sync")
        synchronized(l) {   // inline된 함수와 람다
            println("Action")
        }
        println("After sync")
    }
    ```

- 인라인 함수를 호출하면서 람다를 넘기는 대신에 함수 타입의 변수를 넘길 수도 있다.

    ```kotlin
    class LockOwner(val lock: Lock) {
        fun runUnderLock(body: () -> Unit) {
            synchronized(lock, body)    // 람다 대신 넘겨진 함수 타입의 변수
                                        // 이 경우, 변수에 저장되는 함수를 알 수 없기 때문에 
                                        // 변수에 저장된 함수는 인라인되지 않는다.
        }
    }
    ```

- 한 인라인 함수를 두 곳에서 각각 다른 람다를 사용해 호출한다면 그 두 호출은 각각 따로 인라이닝된다.

### 8.2.2 인라인 함수의 한계

> 람다가 본문에 직접 펼쳐지기 때문에 함수가 파라미터로 전달받은 람다를 본문에 사용하는 방식이 한정될 수밖에 없다.

- 인라이닝하면 안 되는 람다를 파라미터로 받는다면 `noinline` 변경자를 파라미터 이름 앞에 붙여서 인라이닝을 금지할 수 있다.

    ```kotlin
    inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {
        // ...
    }
    ```

- 코틀린에서는 어떤 모듈이나 서드파티 라이브러리 안에서 인라인 함수를 정의하고 그 모듈이나 라이브러리 밖에서 해당 인라인 함수를 사용할 수 있다.
- 자바에서도 코틀린에서 정의한 인라인 함수를 호출할 수 있다. 이 경우, 컴파일러는 인라인 함수를 인라이닝하지 않고 일반 함수 호출로 컴파일한다.

### 8.2.3 컬렉션 연산 인라이닝

#### 표준 라이브러리 함수를 사용하지 않고 직접 이런 연산을 구현한다면 더 효율적이지 않을까?

- 람다를 사용해 컬렉션 거르기

    ```kotlin
    data class Person(val name: String, val age: Int)
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    >>> println(people.filter {it.age < 30})
    [Person(name=Alice, age=29)]

    ```

- 컬렉션을 직접 거르기

    ```kotlin
    >>> val result = mutableListOf<Person>()
    >>> for (person in people) {
            if (person.age < 30) result.add(person)
        }
    >>> println(result)
    [Person(name=Alice, age=29)]
    ```

- 코틀린의 `filter` 함수는 인라인 함수다.
- `filter` 함수의 바이트코드는 그 함수에 전달된 람다 본문의 바이트코드와 함께 `filter`를 호출한 위치에 들어간다.
- 그 결과, 위 예제에서 `filter`를 써서 생긴 바이트코드와 뒤 예제에서 생긴 바이트코드는 거의 같다.
- 결과적으로 코틀린이 제공하는 함수 인라이닝을 믿고 성능에 신경 쓰지 않아도 된다.

#### `filter`와 `map`을 연쇄해서 사용하면 어떻게 될까?

```kotlin
>>> println(people.filter {it.age > 30}
...               .map(Person::name))
[Bob]
```

- 위 예제는 람다 식과 멤버 참조를 사용한다.
- 여기서 사용한 `filter`와 `map`은 인라인 함수다.
- 따라서 그 두 함수의 본문은 인라이닝되며, 추가 객체나 클래스 생성은 없다.
- 하지만 이 코드는 리스트를 걸러낸 결과를 저장하는 중간 리스트를 만든다.

#### `asSequence`를 통해 리스트 대신 시퀀스를 사용하면 중간 리스트로 인한 부가 비용은 줄어든다

- 시퀀스는 람다를 인라인하지 않는다.
- 따라서 지연 계산을 통해 성능을 향상시키려는 이유로 모든 컬렉션 연산에 `asSequence`를 붙이면 안 된다.
- 시퀀스를 통해 성능을 향상시킬 수 있는 경우는 컬렉션 크기가 큰 경우뿐이다.

### 8.2.4 함수를 인라인으로 선언해야 하는 경우

> `inline` 키워드를 사용해도 람다를 인자로 받는 함수만 성능이 좋아질 가능성이 높다.

- 일반 함수 호출의 경우 JVM은 이미 강력하게 인라이닝을 지원한다.
- JVM은 코드 실행을 분석해서 가장 이익이 되는 방향으로 호출을 인라이닝한다.
- 이런 과정은 바이트코드를 실제 기계어 코드로 번역하는 과정(JIT)에서 일어난다.
- 이런 JVM 최적화를 활용하면 바이트코드에서는 각 함수 구현이 정확히 한 번만 있으면 된다.
- 코틀린 인라인 함수는 바이트코드에서 각 함수 호출 지점을 함수 본문으로 대치하기 때문에 코드 중복이 생긴다.

람다를 인자로 받는 함수를 인라이닝할 때의 장점

1. 인라이닝을 없앨 수 있는 부가 비용이 상당하다.
   - 함수 호출 비용을 줄일 수 있다.
   - 람다를 표현하는 클래스와 람다 인스턴스에 해당하는 객체를 만들 필요가 없다.
2. 현재의 JVM은 함수 호출과 람다를 인라이닝해 줄 정도로 똑똑하지 못하다.
3. 인라이닝을 사용하면 일반 람다에서는 사용할 수 없는 몇 가지 기능을 사용할 수 있다.

- 인라인 함수의 크기는 작아야 한다.

### 8.2.5 자원 관리를 위해 인라인된 람다 사용

> 람다로 중복을 없앨 수 있는 일반적인 패턴 중 한 가지는 어떤 작업을 하기 전에 자원을 획득하고 작업을 마친 후 자원을 해제하는 자원 관리다. 여기서 **자원**은 **파일**, **락**, **데이터베이스 트랜잭션** 등 여러 다른 대상을 가리킬 수 있다.

- 자원 관리 패턴을 만들 때 보통 사용하는 방법은 `try/finally`문을 사용하되 `try` 블록을 시작하기 직전에 자원을 획득하고 `finally` 블록에서 자원을 해제하는 것이다.

    ```kotlin
    // 코틀린 라이브러리에 있는 withLock 함수 정의
    fun <T> Lock.withLock(action: () -> T): T {
        lock()
        try {
            return action()
        } finally {
            unlock()
        }
    }
    ```

- 자바 `try-with-resource`과 같은 기능을 제공하는 `use` 함수

    ```kotlin
    /* in java */
    static String readFirstLineFromFile(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            return br.readLine();
        }
    }

    /* in kotlin */
    fun readFirstLineFromFile(path: String): String {
        BufferedReader(FileReader(path)).use {  // BufferedReader 객체를 만들고
                                                // use 함수를 호출하면서
                                                // 파일에 대한 연산을 실행할 람다를 넘긴다.
            br -> return br.readLine()  // 자원(파일)에서 맨 처음 가져온 한 줄을 람다가 아닌
                                        // readFirstLineFromFile에서 반환한다.
        }
    }
    ```

    - `use` 함수는 닫을 수 있는<sup>closeable</sup> 자원에 대한 확장 함수이다.
    - `use` 함수는 람다를 인자로 받는다.

## 8.3 고차 함수 안에서 흐름 제어

### 8.3.1 람다 안의 `return`문: 람다를 둘러싼 함수로부터 반환

- 일반 루프 안에서 `return` 사용하기

    ```kotlin
    data class Person(val name: String, val age: Int)
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    fun lookForAlice(people: List<Person>) {
        for (person in people) {
            if (person.name == "Alice") {
                println("Fount!")
                return
            }
        }
        println("Alice is not found")
    }
    >>> lookForAlice(people)
    Found!
    ```

- `forEach`로 바꿔보기

    ```kotlin
    fun lookForAlice(people: List<Person>) {
        people.forEach {
            if (it.name == "Alice") {
                println("Found!")
                return  // 위 함수와 마찬가지로 lookForAlice 함수에서 반환된다.
            }
        }
        println("Alice is not found")
    }
    ```

    - 람다 안에서 `return`을 사용하면 람다로부터만 반환되는 게 아니라 그 람다를 호출하는 함수가 실행을 끝내고 반환된다.

> 자신을 둘러싸고 있는 블록보다 더 바깥에 있는 다른 블록을 반환하게 만드는 `return`문을 넌로컬<sup>non-local</sup> return이라 부른다.

- 자바의 경우 `return`은 `for` 루프나 `synchronized` 블록을 끝내지 않고 메소드를 반환시킨다.
- 코틀린에서는 언어가 제공하는 기본 구성 요소가 아니라 람다를 받는 함수로 `for`나 `synchronized`와 같은 기능을 구현한다. 코틀린은 그런 함수 안에서 쓰이는 `return`이 자바의 `return`과 같은 의미를 갖게 허용한다.
- 위와 같이 `return`이 바깥쪽 함수를 반환시킬 수 있는 때는 람다를 인자로 받는 함수가 **인라인 함수**인 경우뿐이다.
- **인라이닝이 되지 않는 함수**에 전달되는 람다안에서 `return`을 **사용할 수는 없다.**

### 8.3.2 람다로부터 반환: 레이블을 사용한 `return`

> 람다 식에서도 **로컬 return**을 사용할 수 있다. 람다 안에서 로컬 return은 `for`루프의 `break`와 비슷한 역할을 한다.

- 로컬 return과 넌로컬 return을 구분하기 위해 **레이블**을 사용해야 한다. `return`으로 실행을 끝내고 싶은 람다 식 앞에 레이블을 붙이고, `return` 키워드 뒤에 그 레이블을 추가하면 된다.

    ```kotlin
    fun lookForAlice(people: List<Person>) {
        people.forEach label@{  // 람다 식 앞에 붙은 label
            if (it.name == "Alice") return@label    // 앞에서 정의한 label을 참조한다.
        }
        println("Alice might be somewhere") // 이 줄은 항상 출력된다.
    }
    >>> lookForAlice(people)
    Alice might be somewhere
    ```

- 람다에 레이블을 붙여서 사용하는 대신 람다를 인자로 받는 인라인 함수의 이름을 `return` 뒤에 레이블로 사용해도 된다.

    ```kotlin
    fun lookForAlice(people: List<Person>) {
        people.forEach {
            if (it.name == "Alice") return@forEach  // forEach를 끝낸다.
        }
        println("Alice might be somewhere")
    }
    ```

### 8.3.3 무명 함수: 기본적으로 로컬 `return`

> 무명 함수는 코드 블록을 함수에 넘길 때 사용할 수 있는 다른 방법이다.

- 무명 함수 안에서 `return` 사용하기

    ```kotlin
    fun lookForAlice(people: List<Person>) {
        people.forEach(fun (person) {   // 람다 식 대신 무명 함수를 사용
            if (person.name == "Alice") return  // return은 가장 가까운 함수를 가리킨다.
                                                // 이 경우, 그건 무명 함수이다.
            println("${person.name} is not Alice")
        })
    }
    >>> lookForAlice(people)
    Bob is not Alice
    ```

- `filter`에 무명 함수 넘기기

    ```kotlin
    people.filter(fun (person): Boolean {
        return person.age < 30
    })
    ```

    - 식을 본문으로 하는 무명 함수의 반환 타입은 생략할 수 있다.

#### `return`에 적용되는 규칙은 단순히 "`return`은 `fun` 키워드를 사용해 정의된 가장 안쪽 함수를 반환시킨다"는 점이다

- 람다 식은 `fun`을 사용해 정의되지 않으므로 람다 본문의 `return`은 람다 밖에 함수를 반환시킨다.
