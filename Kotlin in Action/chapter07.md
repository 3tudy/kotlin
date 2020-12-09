# 7장 연산자 오버로딩과 기타 관례

- 연산자 오버로딩
- 관례: 여러 연산을 지원하기 위해 특별한 이름이 붙은 메소드
- 위임 프로퍼티

## 7.1 산술 연산자 오버로딩

### 7.1.1 이항 산술 연산 오버로딩

> 연산자를 오버로딩하는 함수 앞에는 꼭 `operator` 키워드를 붙여야 한다.

- `plus` 연산자 구현하기

    ```kotlin
    data class Point(val x: Int, val y: Int) {
        operator fin plus(other: Point): Point {    // plus라는 이름의 연산자 함수를 정의한다.
                                                    // plus 함수 앞에 operator 키워드를 붙여야 한다.
            return Point(x + other.x, y + other.y)  // 좌표를 성분별로 더한 새로운 점을 반환한다.
        }
    }
    >>> val p1 = Point(10, 20)
    >>> val p2 = Point(30, 40)
    >>> println(p1 + p2)    // '+'로 계산하면 plus 함수가 호출된다.
    Point(x=40, y=60)
    ```

- 연산자를 확장 함수로 정의할 수 있다.

    ```kotlin
    operator fun Point.plus(other: Point): Point {  // 외부 함수의 클래스에 대한 연산자를 정의할 때는
                                                    // 관례를 따르는 이름의 확장함수로 구현하는게 일반적인 패턴이다.
        return Point(x + other.x, y + other.y)
    }
    ```

- 코틀린에서 정의할 수 있는 이항 연산자와 그에 상응하는 연산자 함수 이름

    | 식 | 함수 이름 |
    |---|---|
    | `a * b` | `times` |
    | `a / b` | `div` |
    | `a % b` | `mod`(1.1부터는 `rem`) |
    | `a + b` | `plus` |
    | `a - b` | `minus` |

- 직접 정의한 함수를 통해 구현하더라고 연산자 우선순위는 언제나 표준 숫자 타입에 대한 연산자 우선순위와 같다.
  - `a + b * c`에서는 곱셈이 먼저 연산된다.
  - '*', '/', '%'는 모두 우선순위가 같다.
  - '+'와 '-'는 위 연산자보다 우선순위가 낮다.

- 두 피연산자의 타입이 같을 필요는 없다.

    ```kotlin
    operator fun Point.times(scale: Double): Point {
        return Point((x * scale).toInt(), (y * scale).toInt())
    }
    >>> val p = Point(10, 20)
    >>> println(p * 1.5)
    Point(x=15, y=30)
    ```

- 코틀린 연산자는 자동으로 교환 법칙을 지원하지는 않는다.
- 연산자 함수의 반환 타입이 꼭 두 피연산자 중 하나와 일치할 필요도 없다.

    ```kotlin
     operator fun Char.times(count: Int): String {
         return toString().repeat(count)
     }
     >>> println('a' * 3)
     aaa
    ```

#### 비트 연산자에 대해 특별한 연산자 함수를 사용하지 않는다

> 코틀린은 표준 숫자 타입에 대해 비트 연산자를 정의하지 않는다. 따라서 커스텀 타입에서 비트 연산자를 정의할 수도 없다.

| Kotlin | Java | description |
|---|---|---|
| `shl` | `<<` | 왼쪽 시프트 |
| `shr` | `>>` | 오른쪽 시프트(부호 비트 유지) |
| `ushr` | `>>>` | 오른쪽 시프트(0으로 부호 비트 설정) |
| `and` | `&` | 비트 곱 |
| `or` | `|` | 비트 합 |
| `xor` | `^` | 비트 배타 합 |
| `inv` | `~` | 비트 반전 |

### 7.1.2 복합 대입 연산자 오버로딩

> `plus`와 같은 연산자를 오버로딩하면 코틀린은 `+` 연산자뿐 아니라 그와 관련 있는 연산자인 `+=`도 자동으로 함께 지원한다. `+=`, `-=` 등의 연산자는 **복합 대입<sup>compound assignment</sup>** 연산자라 불린다.

```kotlin
>>> var point = Point(1, 2)
>>> point += Point(3, 4)
>>> println(point)
Point(x=4, y=6)
```

- 변경 가능한 컬렉션에 원소를 추가하는 것과 같이 연산이 객체에 대한 참조를 다른 참조로 바꾸기 보단 원래 객체의 내부 상태를 변경하게 만들기

    ```kotlin
    // 반환 타입이 Unit인 plusAssign 함수를 통해 구현
    operator fun <T> MutableCollection<T>.plusAssign(element: T): Unit {
        this.add(element)
    }
    >>> val numbers = ArrayList<Int>()
    >>> numbers += 42
    >>> println(numbers[0])
    42
    ```

- `plus`와 `plusAssign`를 같이 정의하면 안된다.

### 7.1.3 단항 연산자 오버로딩

- 단항 연산자를 오버로딩하기 위해 사용하는 함수는 인자를 취하지 않는다.

    ```kotlin
    operator fun Point.unaryMinux(): Point {    // 단한 minus 함수는 파라미터가 없다.
        return Point(-x, -y)    // 좌표에서 각 성분의 음수를 취한 새 점을 반환한다.
    }
    >>> val p = Point(10, 20)
    >>> println(-p)
    Point(x=-10, y=-20)
    ```

- 코틀린에서 오버로딩할 수 있는 단항 연산자

    | 식 | 함수 이름 |
    |---|---|
    | `+a` | `unaryPlus` |
    | `-a` | `unaryMinus` |
    | `!a` | `not` |
    | `++a`, `a++` | `inc` |
    | `--a`, `a--` | `dec` |

- `inc`나 `dec` 함수를 정의해 증가/감소 연산자를 오버로딩하는 경우 컴파일러는 일반적인 값에 대한 전위와 후위 증가/감소 연산자와 같은 의미를 제공한다.

    ```kotlin
    operator fun BigDecimal.inc() = this + BigDecimal.ONE
    >>> var bd = BigDecimal.ZERO
    >>> println(bd++)   // 후위 증가 연산자는 println이 실행된 다음에 값을 증가시킨다.
    0
    >>> println(++bd)   // 전위 증가 연산자는 println이 실행되기 전에 값을 증가시킨다.
    2
    ```

## 7.2 비교 연산자 오버로딩

> 코틀린에서는 산술 연산자와 마찬가지로 원시 타입 값뿐 아니라 모든 객체에 대해 비교 연산을 수행할 수 있다.

### 7.2.1 동등성 연산자: `equals`

> 코틀린은 `==` 연산자 호출을 `equals` 메소드 호출로 컴파일한다. `==`와 `!=`는 내부에서 인자가 `null`인지 검사하므로 다른 연산과 달리 `null`이 될 수 있는 값에도 적용할 수 있다.

```kotlin
class Point(val x: Int, val y: Int) {
    override fun equals(obj: Any?): Boolean {   // Any에 정의된 메소드를 오버라이딩
        if (obj === this) return true
        if (obj !is Point) return false
        return obj.x == x && obj.y == y
    }
}
>>> println(Point(10, 20) == Point(10, 20))
true
>>> println(Point(10, 20) != Point(5, 5))
true
>>> println(null == Point(1, 2))
false
```

- 상위 클래스에서 정의된 `operator` 메소드를 오버라이드할 때는 `operator` 변경자를 붙이지 않아도 자동으로 상위 클래스의 `operator` 지정이 적용된다.
- `Any`에서 상속 받은 `equals`가 확장 함수보다 우선순위가 높기 때문에 `equals`를 확장 함수로 정의할 수 없다.
- `!=` 호출은 `equals` 메소드 호출의 반환 값을 반전시켜 돌려준다.

### 7.2.2 순서 연산자: `compareTo`

> 코틀린은 자바와 똑같이 `Comparable` 인터페이스를 지원한다. 게다가 코틀린은 `Comparable` 인터페이스 안에 있는 `compareTo` 메소드를 호출하는 관례를 제공한다. 따라서 비교 연산자(<, >, <=, >=)는 `compareTo` 호출로 컴파일된다.

```kotlin
class Person(val firstName: String, val lastName: String): Comparable<Person> {
    override fun compareTo(other: Person): Int {
        return compareValuesBy(this, other, Person::lastName, Person::firstName)
    }
}
```

## 7.3 컬렉션과 범위에 대해 쓸 수 있는 관례

### 7.3.1 인덱스로 원소에 접근: `get`과 `set`

- 인덱스 연산자를 사용해 원소를 읽는 연산은 `get` 연산자 메소드로 변환되고, 원소를 쓰는 연산은 `set` 연산자 메소드로 변환된다.

    ```kotlin
    operator fun Point.get(index: Int): Int {
        return when(index) {
            0 -> x
            1 -> y
            else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
        }
    }
    >>> val p = Point(10, 20)
    >>> println(p[1])
    20
    ```

- 인덱스에 해당하는 컬렉션 원소를 쓰고 싶은 때는 `set`이라는 이름의 함수를 정의하면 된다.

    ```kotlin
    data class MutablePoint(var x: Int, var y: Int)
    operator fun MutablePoint.set(index: Int, value: Int) {
        when (index) {
            0 -> x = value
            1 -> y = value
            else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
        }
    }
    >>> val p = MutablePoint(10, 20)
    >>> p[1] = 42   // p[index] = value
    >>> println(p)
    MutablePoint(x=10, y=42)
    ```

### 7.3.2 `in` 관례

> `in`은 객체가 컬렉션에 들어있는지 검사한다. 그런 경우 `in` 연산자와 대응하는 함수는 `contains`다.

```kotlin
data class Rectangle(val upperLeft: Point, val lowerRight: Point)
operator fun Rectangle.contains(p: Point): Boolean {
    return p.x in upperLeft.x until lowerRight.x &&
        p.y in upperLeft.y until lowerRight.y
}
>>> val rect = Rectangle(Point(10, 20), Point(50, 50))
>>> println(Point(20, 30) in rect)
true
>>> println(Point(5, 5) in rect)
false
```

- `in`의 우항에 있는 객체는 `contains` 메소드의 수신 객체가 되고, `in`의 좌항에 있는 객체는 `contains` 메소드에 인자로 전달된다.

### 7.3.3 `rangeTo` 관례

> 범위를 만들려면 `..` 구문을 사용해야 한다. `..` 연산자는 `rangeTo` 함수를 간략하게 표현하는 방법이다.

- 코틀린 표준 라이브러리에는 모드 `Comparable` 객체에 대해 적용 가능한 `rangeTo` 함수가 들어있다.

    ```kotlin
    // 이 함수는 범위를 반환하며,
    // 어떤 원소가 그 범위 안에 들어있는지 in을 통해 검사할 수 있다.
    operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>
    ```

### 7.3.4 `for` 루프를 위한 iterator` 관례

> `for (x in list) { ... }`와 같은 문장은 `list.iterator()`를 호출해서 이터레이터를 얻은 다음, 자바와 마찬가지로 그 이터레이터에 대해 `hasNext`와 `next` 호출을 반복하는 식으로 변환된다. 코틀린에서는 `iterator` 메소드를 확장 함수로 정의할 수 있다.

- 코틀린 표준 라이브러리는 `String`의 상위 클래스인 `CharSequence`에 대한 `iterator` 확장 함수를 제공한다.

    ```kotlin
    operator fun CharSequence.iterator(): CharIterator  // 이 라이브러리 함수는 문자열을 이터레이션할 수 있게 해준다.
    >>> for (c in "abc") {}
    ```

- 클래스 안에 직접 `iterator` 메소드를 구현할 수도 있다.

    ```kotlin
    operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
        object: Iterator<LocalDate> {   // 이 객체는 LocalDate 원소에 대한 Iterator를 구현한다.
            var current = start
            override fun hasNext() = current <= endInclusive
            override fun next() = current.apply {
                current = plusDays(1)
            }
        }
    >>> val newYear = LocalDate.ofYearDay(2017, 1)
    >>> val daysOff = newYear.minusDays(1)..newYear
    >>> for (dayOff in daysOff) { println(dayOff) }
    2016-12-31
    2017-01-01
    ```

## 7.4 구조 분해 선언과 `component` 함수

> 구조 분해를 사용하면 복합적인 값을 분해해서 여러 다른 변수를 한꺼번에 초기화할 수 있다.

```kotlin
>>> val p = Point(10, 20)
>>> val (x, y) = p  // 좌변의 여러 변수를 괄호로 묶었다.
>>> println(x)
10
>>> println(y)
20
```

- 구조 분해 선언의 각 변수를 초기화하기 위해 `componentN`이라는 함수를 호출한다. `N`은 구조 분해 선언에 있는 변수 위치에 따라 붙는 번호다.
- `data` 클래스의 주 생성자에 들어있는 프로퍼티에 대해서는 컴파일러가 자동으로 `componentN` 함수를 만들어준다.
- `data` 타입이 아닌 클래스에서 `componentN`을 구현하는 방법

    ```kotlin
    class Point(val x: Int, val y: Int) {
        operator fun component1() = x
        operator fun component2() = y
    }
    ```

- 코틀린 표준 라이브러리에서는 맨 앞의 다섯 원소에 대한 `componentN`을 제공한다. (6개 이상은 구조분해 불가능)

### 7.4.1 구조 분해 선언과 루프

> 함수 본문 내의 선언문뿐 아니라 변수 선언이 들어갈 수 있는 장소라면 어디든 구조 분해 선언을 사용할 수 있다.

```kotlin
fun printEntries(map: Map<String, String>) {
    for ((key, value) in map) {
        println("$key -> $value")
    }
}
>>> val map = mapOf("Oracle" to "Java", "JetBrains" to "Kotlin")
>>> printEntries(map)
Oracle -> Java
JetBrains -> Kotlin
```

## 7.5 프로퍼티 접근자 로직 재활용: 위임 프로퍼티

> 위임 프로퍼티<sup>delegated property</sup>를 사용하면 값을 뒷받침하는 필드에 단순히 저장하는 것보다 더 복잡한 방식으로 작동하는 프로퍼티를 쉽게 구현할 수 있다. 또한 그 과정에서 접근자 로직을 매번 재구현할 필요도 없다.

> **위임**은 객체가 직접 작업을 수행하지 않고 다른 도우미 객체가 그 작업을 처리하게 맡기는 디자인 패턴을 말한다. 이때 작업을 처리하는 도우미 객체를 **위임 객체**라 부른다.

### 7.5.1 위임 프로퍼티 소개

```kotlin
class Foo {
    var p: Type by Delegate()
}
```

1. `p` 프로퍼티는 접근자 로직을 다른 객체에게 위임한다.
2. 위에서는 `Delegate` 클래스의 인스턴스를 위임 객체로 사용한다.
3. `by` 뒤에 있는 식을 계산해서 위임에 쓰일 객체를 얻는다.
4. 프로퍼티 위임 객체가 따라야 하는 관례를 따르는 모든 객체를 위임에 사용할 수 있다.

- 위 선언은 다음과 같은 내용으로 컴파일된다.

    ```kotlin
    class Foo {
        private val delegate = Delegate()   // 컴파일러가 생성한 도우미 프로퍼티
        var p: Type                                         // p 프로퍼티를 위해 컴파일러가 생성한 접근자는
        set(value: Type) = delegate.setValue(..., value)    // delegate의 getValue와 setValue 메소드를 호출한다.
        get() = delegate.getValue(...)
    }
    ```

- `Delegate` 클래스를 단순화하면 다음과 같다.

    ```kotlin
    class Delegate {
        operator fun getValue(...) { ... }
        operator fun setValue(..., value: Type) { ... } 
    }
    class Foo {
        var p: Type by Delegate()
    }
    >>> val foo = Foo()
    >>> val oldValue = foo.p    // delegate.getValue() 호출
    >>> foo.p = newValue        // delegate.setValue() 호출
    ```

### 7.5.2 위임 프로퍼티 사용: `by lazy()`를 사용한 프로퍼티 초기화 지연

> 지연 초기화<sup>lazy initialization</sup>는 객체의 일부분을 초기화하지 않고 남겨뒀다가 실제로 그 부분의 값을 필요할 경우 초기화할 때 흔히 쓰이는 패턴이다.

- 지연 초기화를 뒷받침하는 프로퍼티를 통해 구현하기

    ```kotlin
    class Person(val name: String) {
        private var _emails: List<Email>? = null
        val emails: List<Email>
            get() {
                if (_emails == null) {
                    _emails = loadEmails(this)
                }
                return _emails!!
            }
    }
    >>> val p = Person("Alice")
    >>> p.emails    // 최초로 emails를 읽을 때 단 한 번만 이메일을 가져온다.
    Load emails for Alice
    >>> p.emails
    ```

- 지연 초기화를 위임 프로퍼티를 통해 구현하기

    ```kotlin
    class Person(val name: String) {
        val emails by lazy { loadEmails(this) }
    }
    ```

    - `lazy` 함수는 코틀린 관례에 맞는 시그니처의 `getValue` 메소드가 들어있는 객체를 반환한다.
    - `lazy` 함수의 인자는 값을 초기화할 때 호출할 람다다.

### 7.5.3 위임 프로퍼티 구현

- `PropertyChangeSupport` 클래스
  - 리스너의 목록을 관리
  - `PropertyChangeEvent` 이벤트가 들어오면 목록의 모든 리스너에게 이벤트를 통지
- `ObservableProperty` 클래스
  - 위임을 받는 클래스

```kotlin
class ObservableProperty(var propValue: Int, val chnageSupport: PropertyChangeSupport) {
    operator fun getValue(p: Person, prop: KProperty<*>): Int = propValue
    operator fun setValue(p: Person, prop: KProperty<*>, newValue: Int) {
        val oldValue = propValue
        propValue = newValue
        changeSupport.firPropertyChange(prop.name, oldValue, newValue)
    }
}
```

- `getValue`와 `setValue`는 프로퍼티가 포함된 객체와 프로퍼티를 표현하는 객체를 파라미터로 받는다.
- 코틀린은 `KProperty` 타입의 객체를 사용해 프로퍼티를 표현한다.
- `KProperty` 인자를 통해 프로퍼티 이름을 전달받으므로 주 생성자에서는  `name` 프로퍼티를 없앤다.

```kotlin
class Person(val name: String, age: Int, salary: Int): PropertyChangeAware() {
    var age: Int by ObservableProperty(age, changeSupport)
    var salary: Int by ObservableProperty(salary, changeSupport)
}
```

- `by` 오른쪽에 오는 객체를 **위임 객체**라고 부른다.
- 표준 라이브러리를 사용해 값의 변경을 통지할 수 있다. 이 경우, `PropertyChangeSupport`와 표준 라이브러리의 클래스를 연결하기 위해 `PropertyChangeSupport`를 사용하는 방법을 알려주는 람다를 그 표준 라이브러리 클래스에게 넘겨야 한다.

    ```kotlin
    class Person(val name: String, age: Int, salary: Int): PropertyChangeAware() {
        private val observer = {
            prop: KProperty<*>, oldValue: Int, newValue: Int ->
            changeSupport.firePropertyChange(prop.name, oldVaue, newValue)
        }
        var age: Int by Delegates.observable(age, observer)
        var salary: Int by Delegates.observable(salary, observer)
    }
    ```

### 7.5.4 위임 프로퍼티 컴파일 규칙

```kotlin
class C {
    var prop: Type by MyDelegate()
}
val c = C()
```

1. 컴파일러는 `MyDelegate` 클래스의 인스턴스를 감춰진 프로퍼티에 저장하며 그 감춰진 프로퍼티를 `<delegate>`라는 이름으로 부른다.
2. 컴파일러는 프로퍼티를 표현하기 위해 `KProperty` 타입의 객체를 사용한다. 이 객체를 `<property>`라고 부른다.
3. 컴파일러는 다음 코드를 생성한다.

    ```kotlin
    class C {
        private val <delegate> = MyDelegate()
        var prop: Type
            get() = <delegate>.getValue(this, <property>)
            set(value: Type) = <delegate>.setValue(this, <property>, value)
    }
    ```

### 7.5.5 프로퍼티 값을 맵에 저장

- 값을 맵에 저장하는 프로퍼티 정의

    ```kotlin
    class Person {
        private val _attributes = hashMapOf<String, String>()
        fun setAttribute(attrName: String, value: String) {
            _attributes[attrName] = value
        }
        val name: String
            get() = _attributes["name"]!!
    }
    >>> val p = Person()
    >>> val data = mapOf("name" to "Dmitry", "company" to "JetBrains")
    >>> for ((attrName, value) in data)
    ...     p.setAttribute(attrName, value)
    >>> println(p.name)
    Dmitry
    ```

- 값을 맵에 저장하는 위임 프로퍼티 사용

    ```kotlin
    class Person {
        private val _attributes = hashMapOf<String, String>()
        fun setAttribute(attrName: String, value: String) {
            _attributes[attrName] = value
            val name: String by _attributes
        }
    }
    ```

- 표준 라이브러리가 `Map`과 `MutableMap` 인터페이스에 대해 `getValue`와 `setValue` 확장 함수를 제공한다.
- `getValue`에서 맵에 프로퍼티를 값을 저장할 때는 자동으로 프로퍼티 이름을 키로 활용한다.

### 7.5.6 프레임워크에서 위임 프로퍼티 활용

- 위임 프로퍼티를 사용해 데이터베이스 칼럼 접근하기

    ```kotlin
    object Users: IdTable() {   // 객체는 데이터베이스 테이블에 해당한다.
        val name = varchar("name", length=50).index()
        val age = integer("age")
    }
    class User(id: EntityID): Entity(id) {  // 각 User 인스턴스는 테이블에 들어있는 구체적인 엔티티에 해당한다.
        var name: String by Users.name
        var age: Int by Users.age
    }
    ```

- 프레임워크는 `Column` 클래스 안에 `getValue`와 `setValue` 메소드를 정의한다.

    ```kotlin
    operator fun <T> Column<T>.getValue(o: Entity, desc: KProperty<*>): T {
        // 데이터베이스에서 칼럼 값 가져오기
    }
    operator fun <T> Column<T>.setValue(o: Entity, desc: KProperty<*>, value: T) {
        // 데이터베이스의 값 변경하기
    }
    ```
