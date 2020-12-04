# 5장 람다 프로그래밍

- 람다 식과 멤버 참조
- 함수형 스타일로 컬렉션 다루기
- 시퀀스: 지연 컬렉션 연산
- 자바 함수형 인터페이스를 코틀린에서 사용
- 수신 객체 지정 람다 사용

> **람다 식<sup>lambda expression</sup>** 또는 **람다**는 기본적으로 다른 함수에 넘길 수 있는 **작은 코드 조각**을 뜻한다.

## 5.1 람다 식과 멤버 참조

### 5.1.1 람다 소개: 코드 블록을 함수 인자로 넘기기

함수를 자체를 다른 함수로 전달한다.

```java
// onClickListener in java
// 무명 내부 클래스로 리스너 구현하기
button.setOnClickListener(new OnClickListener() {   // onClick method를 구현하기 위해
    @Override                                       // 무명 클래스의 인스턴스를 선언해야 한다.
    public void onClick(View view) { /* 클릭 시 수행할 동작 */ }
});

// 람다로 리스너 구현하기
button.setOnClickListener { /* 클릭 시 수행할 동작 */ }
```

### 5.1.2 람다와 컬렉션

```kotlin
// 람다를 사용해 컬렉션 검색
>>> val people = listOf(Person("Alice", 29), Person("Bob", 31))
>>> println(people.maxBy{ it.age }) // 람다가 멤버를 참조해 people에서 age가 가장 큰 멤버를 반환한다.
Person(name=Bob, age=31)
```

### 5.1.3 람다 식의 문법

> 람다는 값처럼 여기저기 전달할 수 있는 동작의 모음

#### 람다 식 문법

```kotlin
{ parameters -> body }  // 항상 중괄호 사이에 위치함

{ x: Int, y: Int -> x + y }
```

1. 람다를 변수에 저장하고 변수를 통해 람다를 호출할 수 있다.

    ```kotlin
    >>> val sum = { x: Int, y: Int -> x + y }
    >>> println(sum(1, 2))  // 변수에 저장된 람다를 호출
    3
    ```

2. 람다 식을 직접도 호출할 수는 있다.

    ```kotlin
    >>> { println(42) }()   // 그닥 유용하지는 않다.
    42

    // 굳이 코드 블럭을 쓸거라면
    // 'run'을 사용하면 그나마 가독성은 약간 향상되는 듯하다.
    >>> run { println(42) } // 람다 본문에 있는 코드를 실행한다.
    42
    ```

3. 코틀린에서는 람다도 줄일 수 있다.

    ```kotlin
    >>> val people = listOf(Person("Alice", 29), Person("Bob", 31))
    >>> println(people.maxBy{ it.age }) // 앞서 봤던 'maxBy'가 기존 람다를 줄인 함수이다.
    Person(name=Bob, age=31)

    // 정식으로 람다를 작성하면
    people.maxBy({ p: Person -> p.age })    // 소괄호 안에 있는 람다 식으로 'maxBy' 함수의 인자로 넘기고 있다.
    ```

4. 람다를 변수에 저장할 때는 파라미터의 타입을 추론할 문맥이 존재하지 않는다. 따라서 파라미터 타입을 명시해야 한다.

    ```kotlin
    >>> val getAge = { p: Person -> p.age }
    >>> people.maxBy(getAge)
    ```

#### 다시 처음 썼던 'maxBy'로 되돌아가 보자

1. 코틀린에서는 함수 호출 시 맨 뒤에 있는 인자가 람다 식이면 그 람다를 괄호 밖으로 빼낼 수 있다.

    ```kotlin
    people.maxBy() { p: Person -> p.age }
    ```

2. 람다가 어떤 함수의 유일한 인자이고 괄호 뒤에 람다를 썼다면 호출 시 빈 괄호를 없애도 된다.

    ```kotlin
    people.maxBy { p: Person -> p.age }
    ```

3. 로컬 변수처럼 컴파일러는 람다 파라미터의 타입도 추론할 수 있다. 따라서 파라미터 타입을 명시할 필요가 없다.

    ```kotlin
    people.maxBy { p -> p.age }
    ```

    > `maxBy` 함수의 경우 파라미터의 타입은 항상 컬렉션 원소 타입과 같다. 컴파일러는 `Person` 타입의 객체가 들어있는 컬렉션에 대해 `maxBy`를 호출한다는 사실을 알고 있으므로 람다의 파라미터도 `Person`이라는 사실을 이해할 수 있다. \
    다만, 컴파일러가 항상 람다 파라미터의 타입을 추론할 수 있는 것은 아니다.

4. 람다의 파라미터가 하나뿐이고 그 타입을 컴파일러가 추론할 수 있는 경우 `it`을 바로 쓸 수 있다. (하지만 파라미터 타입을 명시해주는 게 더 좋다.)

    ```kotlin
    people.maxBy { it.age }
    ```

### 5.1.4 현재 영역에 있는 변수에 접근

> 자바 메소드 안에서 무명 내부 클래스를 정의할 때 **메소드의 로컬 변수**를 **무명 내부 클래스**에서 사용할 수 있다. 람다 안에서도 같은 일을 할 수 있다. 람다를 함수 안에서 정의하면 함수의 파라미터뿐 아니라 **람다 정의의 앞에 선언된 로컬 변수 까지** 람다에서 모두 사용할 수 있다.

```kotlin
// `foreach` 표준 함수는 컬렉션의 모든 원소에 대해 람다를 호출해준다.
// python의 'map'과 유사한거 같다.
fun printMessagesWithPrefix(messages: Collection<String>, prefix: String) {
    messages.forEach {          // 각 원소에 대해 수행할 작업을 람다로 받는다.
        println("$prefix $it")  // 람다 아네서 함수의 'prefix' 파라미터를 사용한다.
    }
}
>>> val errors = listOf("403 Forbidden", "404 Not Found")
>>> printMessagesWithPrefix(errors, "Error:")
Error: 403 Forbidden
Error: 404 Not Found
```

자바와 다른 점

1. 코틀린 람다 안에서는 파이널 변수가 아닌 변수에 접근할 수 있다.
2. 람다 안에서 바깥의 변수를 변경할 수 있다.

```kotlin
fun printProblemCounts(responses: Collections<String>) {
    var clientErrors = 0
    var serverErrors = 0
    response.forEach {
        if (it.startsWith("4")) {
            clientErrors++      // 람다 안에서 람다 밖의 변수를 변경
        } else if (it.startsWith("5")) {
            serverErrors++      // 람다 안에서 람다 밖의 변수를 변경
        }
    }
    println("$clientErrors client errors, $serverErrors server errors")
}
>>> val responses = listOf("200 OK", "418 I'm a teapot", "500 Internal Server Error")
>>> printProblemCounts(responses)
1 client errors, 1 server errors
```

위 예제에서 `prefix`, `clientErrors`, `serverErrors`와 같이 람다 안에서 사용하는 외부 변수를 **람다가 포획<sup>capture</sub>한 변수**라고 부른다.

- 기본적으로 함수 안에 정의된 로컬 변수의 생명주기는 함수가 반환되면 끝난다.
- 어떤 함수가 자신의 로컬 변수를 포획한 람다를 반환하거나 다른 변수에 저장한다면 로컬 변수의 생명주기와 함수의 생명주기가 달라질 수 있다.

#### 포획한 변수가 있는 람다를 저장해서 함수가 끝난 뒤에 실행해도 람다의 본문 코드는 여전히 포획한 변수를 읽거나 쓸 수 있다

1. `final` 변수를 포획한 경우
   - 람다 코드를 변수 값과 함께 저장한다.
2. `final`이 아닌 변수를 포획한 경우
   - 변수를 특별한 래퍼로 감싸서 나중에 변경하거나 읽을 수 있게 한 다음
   - 래퍼에 대한 참조를 람다 코드와 함께 저장한다.

#### 람다를 이벤트 핸들러나 다른 비동기적으로 실행되는 코드로 활용하는 경우 함수 호출이 끝난 다음에 로컬 변수가 변경될 수도 있다

```kotlin
fun tryToCountButtonClicks(button: Button): Int {
    var clicks = 0
    button.onClick { clicks++ }
    return clicks
}
```

`onClick` 핸들러는 호출될 때마다 `clicks`의 값을 증가시키지만, 호출의 순서가 `tryToCountButtonClicks`가 `clicks`를 반환한 다음에 핸들러가 호출되기 때문에 **이 함수는 항상 0을 반환한다.**

#### 클로저

> 람다를 실행 시점에 표현하는 데이터 구조

1. 람다에서 시작하는 모든 참조가 포함된 닫힌 객체 그래프를 람다 코드와 함께 저장해야 한다.
2. 포획한 변수를 제대로 처리하기 위해 클로저가 필요하다.

### 5.1.5 멤버 참조

함수를 값으로 바꿔서 인자로 직접 넘기기

```kotlin
val getAge = Person::age    // 멤버 참조를 뜻하는 이중콜론(::)
```

> 멤버 참조<sup>member reference</sup>는 프로퍼티나 메소드를 단 하나만 호출하는 **함수 값**을 만들어준다.

```kotlin
class::member

Peron::age
{ person: Person -> person.age }

val getAge = Person::age
val getAge = { person: Person -> person.age }
```

- 최상위에 선언된(그리고 다른 클래스의 멤버가 아닌) 함수나 프로퍼티를 참조할 수도 있다.

```kotlin
$ cat salute.kt
fun salute() = println("Salute!")

fun main(args:Array<String>) {
    run(::salute)   // 최상위 함수를 참조한다.
                    // 클래스 이름을 생략하고 이중콜론(::)으로 참조를 바로 시작한다.
}
$ kotlinc salute.kt
$ kotlin SaluteKt
Salute!
```

- 람다가 인자가 여럿인 다른 함수한테 작업을 위임하는 경우 람다를 정의하지 않고 직접 위임 함수에 대한 참조를 제공할 수 있다.

```kotlin
// 'sendEmail' 함수에게 작업을 위임
val action = { person: Person, message: String -> sendEmail(person, message)}
// 람다 대신 멤버 참조를 쓸 수 있다.
val nextAction = ::sendEmail
```

- 생성자 참조<sup>constructor reference</sup>를 사용하면 클래스 생성 작업을 연기하거나 저장해둘 수 있다. 이중콜론(::) 뒤에 클래스 이름을 넣으면 생성자 참조를 만들 수 있다.

```kotlin
data class Person(val name: String, val age: Int)
>>> val createPerson = ::Person // "Person"의 인스턴스를 만드는 동작을 값으로 저장한다.
>>> val p = createPerson("Alice", 29)
>>> println(p)
Person(name=Alice, age=29)
```

- 확장 함수도 맴버 함수와 똑같은 방식으로 참조할 수 있다.

```kotlin
fun Person.isAdult() = age >= 21
val predicate = Person::isAdult
```

#### 바운드 멤버 참조

```kotlin
>>> val p = Person("Dmitry", 34)
>>> val personAgeFunction = Person::age
>>> println(personAgeFunction(p))
34
>>> val dmitrysAgeFunction = p::age // p 객체의 age 프로퍼티를 리턴하는 람다
>>> println(dmitrysAgeFunction())   // dmitrysAgeFunction은 인자가 필요없다.
34
```

## 5.2 컬렉션 함수형 API

### 5.2.1 필수적인 함수: `filter`와 `map`

> `filter`와 `map`은 컬렉션을 활용할 때 기반이 되는 함수다.

`filter` 컬렉션을 이터레이션하면서 주어진 람다에 각 원소를 넘겨서 람다가 `true`를 반환하는 원소만 모은다.

```kotlin
data class Person(val name: String, val age: Int)
>>> val list = listOf(1, 2, 3, 4)
>>> println(list.filter { it % 2 == 0 })    // 짝수만 남는다.
[2, 4]
// python의 리스트 컴프리헨션을 약간 풀어 쓴 느낌?
```

`map` 주어진 **람다**를 컬렉션의 각 원소에 적용한 결과를 모아서 **새 컬렉션을 만든다.**

```kotlin
>>> val list = listOf(1, 2, 3, 4)
>>> println(list.map { it * it })
[1, 4, 9, 16]
```

#### map 자료구조 (python의 딕셔너리)

> 맵의 경우 키와 값을 처리하는 함수가 따로 존재한다.

1. `filterKeys`와 `mapKeys`는 **키**를 걸러 내거나 변환한다.
2. `filterValues`와 `mapValues`는 **값**을 걸러 내거나 변환한다.

### 5.2.2 `all`, `any`, `count`, `find`: 컬렉션에 술어 적용

1. `all`, `any` 모든 원소가 어떤 조건을 만족하는지 판단하는 연산

    ```kotlin
    val canBeInClub27 = { p: Person -> p.age <= 27 }

    >>> val people = listOf(Person("Alice", 27), Person("Bob", 31))
    >>> println(people.all(canBeInClub27))  // 모든 원소가 람다를 만족하는 지
    false
    >>> println(people.any(canBeInClub27))  // 하나라도 람다를 만족하는 원소가 있는지
    true

    // 아래 둘은 같은 조건을 나타낸다.
    // 부정(!)을 all이나 any에 붙이기보단 술어(조건문)에서 사용하는게 가독성에 좋다.
    >>> val list = listOf(1, 2, 3)
    >>> println(!list.all { it == 3})
    true
    >>> println(list.any { it != 3 })
    true
    ```

2. `count` 조건을 만족하는 **원소의 개수**를 반환

    ```kotlin
    >>> val people = listOf(Person("Alice", 27), Person("Bob", 31))
    >>> println(people.count(canBeInClub27))
    1
    >>> println(people.filter(canBeInClub27).size)  // 위와 같은 결과를 낸다.
                                                    // 하지만 size 프로퍼티를 갖는 중간 컬렉션이 생성된다.
                                                    // 성능을 위해 count를 사용하자.
    1
    ```

3. `find` 조건을 만족하는 **첫 번째 원소**를 반환

    ```kotlin
    >>> val people = listOf(Person("Alice", 27), Person("Bob", 31))
    >>> println(people.find(canBeInClub27))
    Person(name=Alice, age=27)      // 원소가 있는 경우 해당 원소를 반환
                                    // 원소가 없으면 null을 반환한다.
                                    // firstOrNull과 같다.
    ```

### 5.2.3 `groupBy`: 리스트를 여러 그룹으로 이뤄진 맵으로 변경

> 특성을 파라미터로 전달하면 컬렉션을 자동으로 구분해주는 함수

```kotlin
>>> val people = listOf(Person("Alice", 31), Person("Bob", 29), Person("Carol", 31))
>>> println(people.groupBy { it.age })  // 같은 나이끼리 묶인다.
{29=[Person(name=Bob, age=29)],
 31=[Person(name=Alice, age=31), Person(name=Carol, age=31)]}
```

각 그룹은 리스트다. 위 `groupBy`의 결과 타입은 `Map<Int, List<Person>>`이다.

### 5.2.4 `flatMap`과 `flatten`: 중첩된 컬렉션 안의 원소 처리

`flatMap`

1. 인자로 주어진 람다를 컬레션의 모든 객체에 적용하고
2. 람다를 적용한 결과 얻어지는 여러 리스트를 한 리스트로 한데 모은다.

```kotlin
>>> val strings = listOf("abc", "def")
>>> println(strings.flatMap { it.toList() })
[a, b, c, d, e, f]
```

```kotlin
>>> val books = listOf(Book("Thursday Next", listOf("Jasper Fforde")),
                       Book("Mort", listOf("Terry Pratchett")),
                       Book("Good Omens", listOf("Terry Pratchett",
                                                 "Neil Gaiman")))
>>> println(books.flatMap{ it.authors }.toSet())
[Jasper Fforde, Terry Pratchett, Neil Gaiman]
```

1. 주어진 람다를 `map`
2. `map`으로 연산된 결과를 `flatten`으로 합친다.

## 5.3 지연 계산(lazy) 컬렉션 연산

> 앞 절에서는 `map`이나 `filter` 같은 몇 가지 컬렉션 함수를 살펴봤다. 그런 함수는 컬렉션을 **즉시** 생성한다. 이는 컬렉션 함수를 연쇄하면 매 단계마다 계산 중간 결과를 새로운 컬렉션에 임시로 담는다는 말이다.

```kotlin
people.map(Person::name).filter { it.startsWith("A") }
// 1. map에 의한 중간 컬렉션이 생성됨
// 2. filter에 의한 컬렉션이 생성됨
```

각 연산이 매번 컬렉션을 직접 사용하는 대신 **시퀀스<sup>sequence</sup>를 사용해 연산을 좀더 효율적으로 만들 수 있다.

```kotlin
people.asSequence()     // 원본 컬렉션을 sequence로 변환한다.
    .map(Person::name)
    .filter { it.startsWith("A") }
    .toList()   // 결과 sequence를 다시 리스트로 변환한다.
```

코틀린 지연 계산 시퀀스는 `Sequence` 인터페이스에서 시작한다. `Sequence` 안에는 iterator라는 단 하나의 메소드가 있다. 그 메소드를 통해 시퀀스로부터 원소 값을 얻을 수 있다.(Q. python의 generator와 같은 역할인가?)

원소를 차례로 이터레이션해야할 때는 시퀀스를 직접 써도 괜찮지만 시퀀스를 참조할 때 리스트처럼 인덱스로 참조하는 것은 불가능하다.

### 5.3.1 시퀀스 연산 실행: 중간 연산과 최종 연산

> 시퀀스에 대한 연산은 **중간<sup>intermediate</sup> 연산**과 **최종<sup>terminal</sup> 연산**으로 나뉜다.

- **중간 연산** 최초 시퀀스의 원소를 변환하는 방법을 안다.
- **최종 연산** 결과를 반환한다.

```kotlin
people.asSequence()
    .map(Person::name)              // | 중간 연산
    .filter { it.startsWith("A") }  // |
    .toList()   // 최종 연산
```

각 원소 하나씩 전체 연산을 한번에 통과한다. 즉, `filter`와 `map`의 순서에 따라서도 성능의 차이가 생길 수 있다.

### 5.3.2 시퀀스 만들기

컬렉션에 대해 `asSequence()`를 호출해 시퀀스를 만드는 방법 외에 다른 방법은 `generateSequence` 함수를 사용하는 것이다.

```kotlin
// 자연수의 시퀀스를 생성하고 사용하기
>>> val naturalNumbers = generateSequence(0) { it + 1 }
>>> val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
>>> println(numbersTo100.sum()) // 모든 지연 연산은 'sum'의 결과를 계산할 때 수행된다.
5050
```

## 5.4 자바 함수형 인터페이스 활용

### 5.4.1 자바 메소드에 람다를 인자로 전달

함수형 인터페이스를 인자로 원하는 자바 메소드에 코틀린 람다를 전달할 수 있다.

```kotlin
/* java */
void postponeComputation(int delay, Runnable computation);

/* kotlin */
postponeComputation(1000) { println(42) }
// 코틀린에서 람다를 Runnable 인스턴스로 변환해준다.
```

**`Runnable` 인스턴스**는 실제로는 **`Runnable`을 구현한 무명 클래스의 인스턴스**이다. 컴파일러는 자동으로 그런 무명 클래스와 인스턴스를 만들어준다. 이때 그 무명 클래스에 있는 **유일한 추상 메소드**를 구현할 때 **람다 본문**을 메소드 본문으로 사용한다.

```kotlin
// 람다 대신 무명 객체를 명시적으로 만들어서 사용할 수도 있다.
postponeComputation(1000, object: Runnable {
    override fun run() {
        println(42)
    }
})
```

1. 무명 객체를 명시적으로 선언하는 경우
   - 메소드를 호출할 때마다 새로운 객체가 생성된다.
2. 람다를 사용하는 경우
   - 정의가 들어있는 함수의 변수에 접근하지 않는 람다에 대응하는 무명 객체를 메소드를 호출할 때마다 반복 사용한다.

```kotlin
// 람다와 동일한 코드
val runnable = Runnable { println(42) } // 전역 변수로 컴파일되므로 프로그램 안에
                                        // 단 하나의 인스턴스만 존재한다.
fun handleComputation() {
    postponeComputation(1000, runnable) // 모든 handleComputation 호출에 같은 객체를 사용한다.
}
```

- 람다가 주변 영역의 변수를 포획한다면 매 호출마다 같은 인스턴스를 사용할 수 없다

```kotlin
// 컴파일러는 매번 주변 영역의 변수를 포획한 새로운 인스턴스를 생성해준다.
fun handelComputation(id: String) { // 람다 안에서 'id' 변수를 포획한다.
    postponeComputation(1000) { println(id) }   // handleComputation을 호출할 때 마다
}                                               // 새로 Runnable 인스턴스를 만든다.
```

### 5.4.2 SAM<sup>Single Abstract Method</sup> 생성자: 람다를 함수형 인터페이스로 명시적으로 변경

> SAM 생성자는 람다를 함수형 인터페이스의 인스턴스로 변환할 수 있게 컴파일러가 자동으로 생성한 함수다. **컴파일러가 자동으로 람다를 함수형 인터페이스 무명 클래스로 바꾸지 못하는 경우** SAM 생성자를 사용할 수 있다.

```kotlin
// SAM 생성자를 사용해 값 반환하기
fun createAllDoneRunnable(): Runnable {
    return Runnable { println("All done!") }
}
>>> createAllDoneRunnable().run()
All done!
```

- SAM 생성자의 이름은 사용하려는 함수형 인터페이스의 이름과 같다.
- SAM 생성자는 그 함수형 인터페이스의 유일한 추상 메소드의 본문에 사용할 람다만을 인자로 받아서 함수형 인터페이스를 구현하는 클래스의 인스턴스를 반환한다.

```kotlin
// SAM 생성자를 사용해 listener 인스턴스 재사용하기
val listener = OnClickListener { view ->
    val text = when (view.id) {     // view.id를 사용해 어떤 버튼이 클릭됐는지 판단한다.
        R.id.button1 -> "First button"
        R.id.button2 -> "Second button"
        else -> "Unknown button"
    }
    toast(text)         // 'text'의 값을 사용자에게 보여준다.
}
button1.setOnClickListener(listener)
button2.setOnClickListener(listener)
```

## 5.5 수신 객체 지정 람다: `with`와 `apply

수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메소드를 호출할 수 있게 한다.

### 5.5.1 `with` 함수

**`with`** 어떤 객체의 이름을 반복하지 않고도 그 객체에 대해 다양한 연산을 수행할 수 있도록 해주는 함수

```kotlin
// 'with' 함수 없이 알파벳 출력하기
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in 'A'..'Z') {
        result.append(letter)
    }
    result.append("\nNow I know the alphabet!")
    return result.toString()
}
>>> println(alphabet())
ABCDEFGHIJKLMNOPQRSTUVWXYZ
Now I know the alphabet!

// `with` 함수를 사용하여 알파벳 출력하기
fun alphabet(): String {
    val stringBuilder = StringBuilder()
    return with(stringBuilder) {    // 메소드를 호출하려는 수신 객체를 지정한다.
        for (letter in 'A'..'Z') {
            this.append(letter)     // 'this'를 명시해서 앞에서 지정한
                                    // 수신 객체의 메소드를 호출한다.
        }
        append("\nNow I know the alphabet!")    // 'this'를 생략하고 메소드를 호출한다.
        this.toString() // 람다에서 값을 반환한다.
    }
}
```

파라미터가 두 개인 함수 `with`

```kotlin
with(stringBuilder, { /* alphabet things */ })
```

- `with`는 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다.
- 인자로 받은 람다 본문에서는 `this`를 사용해 그 수신 객체에 접근할 수 있다.

#### 메소드 이름 충돌

> `with`에게 인자로 넘긴 객체의 클래스와 `with`를 상굥하는 코드가 들어있는 클래스 안에 이름이 같은 메소드가 있을 땐 `this` 참조 앞에 레이블을 붙이면 호출하고 싶은 메소드를 명확하게 정할 수 있다.

`alphabet` 함수가 `OuterClass`의 메소드라고 할 때, `StringBuilder`가 아닌 바깥쪽 클래스(`OuterClass`)에 정의된 `toString`을 호출하고 싶다면

```kotlin
this@OuterClass.toString()
```

### 5.5.2 `apply` 함수

`with`와 거의 같은 기능을 하지만 반환하는 것이 람다의 결과가 아닌 **자신에게 전달된 객체**를 반환한다.

```kotlin
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
}.toString()
```

`apply`는 **확장 함수**로 정의되어 있다. 위 함수에서 `apply`를 실행한 결과는 `StringBuilder` 객체다. 따라서 그 객체의 `toString`을 호출해서 `String` 객체를 얻을 수 있다.

- 객체의 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화해야 하는 경우 유용하다.
- 자바에서는 `Builder` 객체가 이런 역할을 담당한다.
