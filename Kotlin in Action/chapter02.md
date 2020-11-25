# 2장 코틀린 기초

- 함수, 변수, 클래스, enum, 프로퍼티를 선언하는 방법
- 제어 구조
- 스마트 캐스트
- 예외 던지기와 예외 잡기

## 2.1 기본 요소: 함수와 변수

### 2.1.1 Hello, World

```kotlin
fun main(args: Array<String>) {     // line 1
    println("Hello, world!")        // line 2
}
```

1. line 1, 함수를 선언할 때 `fun` 키워드를 사용
2. line 1, 파라미터 이름 **뒤에** 그 파라미터의 **타입**을 쓴다(변수도 마찬가지)
3. 함수를 최상위 수준에 정의할 수 있다(클래스 안에 함수를 넣을 필요가 없다)
4. 배열도 일반적인 클래스와 똑같은 방법으로 선언할 수 있다
5. line 2, `System.out.println` 대신 `println`으로 wrapping되어 있다
6. 세미콜론 `;`을 사용하지 않는다

### 2.1.2 함수

```kotlin
fun max(a: Int, b: Int): Int {      // fun 함수이름(파라미터): 반환 타입
    return if (a > b) a else b
}

fun max(a: Int, b: Int): Int = if (a > b) a else b  // 식이 본문인 함수
fun max(a: Int, b: Int) = if (a > b) a else b       // Kotlin은 반환 타입을 적지 않아도 컴파일러가 타입 추론을 통해 반환타입을 정해준다
                                                    // (식이 본문인 함수의 반환 타입만 생략 가능하다)
```

### 2.1.3 변수

```kotlin
// 변수 선언 시 타입 표기를 생략하는 것이 가능하다
// 단, initializer가 필요하다
val question = "삶, 우주, 그리고 모든 것에 대한 궁극적인 질문"
val answer = 42

// 타입을 표기하는 것 또한 가능하다
val answer: Int = 42

// 부동소수점(floating point) 상수를 사용한다면 변수 타입은 Double
val yearsToCompute = 7.5e6

// 초기화 식을 사용하지 않고 변수 선언 시
// 변수 타입을 반드시 명시해야 한다
val answer: Int
answer = 42
```

#### 변경 가능한 변수와 변경 불가능한 변수

1. `val`(from value): 변경 불가능한<sub>*immutable*</sub> 참조를 저장하는 변수(`final` in Java)

    ```kotlin
    val languages = arrayListOf("Java")     // 불변 참조를 선언
    languages.add("Kotlin")                 // 참조가 가리키는 객체 내부는 변경 가능
    ```

2. `var`(from variable): 변경 가능한<sub>*mutable*</sub> 참조를 저장하는 변수
    - `var` 키워드를 사용해 변수를 선언하면 **변수의 값**은 바꿀 수 있으나 **타입**은 변경 불가능하다

        ```kotlin
        var answer = 43
        answer = "no answer"    // Error: type mismatch, 이미 추론된 타입은 변경할 수 없다
        ```

### 2.1.4 더 쉽게 문자열 형식 지정: 문자열 템플릿

```kotlin
fun main(args: Array<String>) {
    val name = if (args.size > 0) args[0] else "Kotlin"
    println("Hello, $name!")        // $name을 이용해 문자열안에 변수를 사용할 수 있다
    println("\$ 1,000")             // '$'를 문자열 안에 넣고 싶을 땐 escape 사용
    println("Hello, ${args[0]}!")   // 넘기고 싶은 식은 중괄호({}) 안에 넣어서
    println("Hello, ${if (args.size > 0) args[0] else "Kotlin"}!")
}
```

## 2.2 클래스와 프로퍼티

가시성 변경자<sup>*visibility modifier*</sup>

- `public` (기본적인 가시성 변경자이기에 생략 가능)
- `private`

### 2.2.1 프로퍼티

```kotlin
class Person(
    val name: String,       // 읽기 전용 프로퍼티, private field와 public getter 생성
    var isMarried: Boolean  // 쓸 수 있는 프로퍼티, priavte field와 public getter & setter 생성
)
```

```java
Person person = new Person("Bob", true);
System.out.println(person.getName());
//>> Bob
System.out.println(person.isMarried()); // Java에서 is가 붙은 프로퍼티는 getter이름이 프로퍼티 이름과 동일하다, setter는 is 대신 set이 들어간다
//>> true

person.setMarried(false);   // setter 메소드 호출
```

```kotlin
val person = Person("Bob", true)    // new 키워드를 사용하지 않고 생성자를 호출
println(person.name)                // 프로퍼티 이름을 직접 사용해도 getter를 자동으로 호출
//>> Bob
println(person.isMarried)
//>> true

person.isMarried = false    // 프로퍼티를 직접 바꾸듯이, 하지만 setter를 호출한것
```

### 2.2.2 커스텀 접근자

```kotlin
class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() {
            return height == width
        }
}

class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() = height == width
}
```

### 2.2.3 코틀린 소스코드 구조: 디렉터리와 패키지

1. 같은 패키지 내에 있다면 다른 파일에서 정의한 선언도 직접 사용할 수 있다
2. 다른 패키지에서 정의한 선언을 사용하려면 import를 통해 선언을 불러와야 한다

```kotlin
// 클래스와 함수 선언을 패키지에 넣기
// 1. 클래스 import와 함수 import에 차이가 없음
// 2. 모든 선언을 import 키워드로 가져올 수 있다
package geometry.shapes     // 패키지 선언
import java.util.Random     // 표준 자바 라이브러리 클래스를 import

class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() = height == width
}

fun createRandomRectangle(): Rectangle {
    val random = Random()
    return Rectangle(random.nextInt(), random.nextInt())
}
```

```kotlin
// 최상위 함수는 그 이름을 써서 import할 수 있다
package geometry.example
import geometry.shapes.createRandomRectangle    // 이름으로 함수 import
fun main(args: Array<String>) {
    println(createRandomRectangle().isSquare)
}

// .*과 같은 star import는 패키지 안에 있는 모든 클래스뿐 아니라 최상위에 정의된 함수나 프로퍼티까지 모드 불러온다
```

패키지와 디렉터리 구조를 자바와 다르게 할 순 있지만 대체로 그대로 따르는게 좋다

## 2.3 선택 표현과 처리: enum과 when

### 2.3.1 enum 클래스 정의

**soft keyword**: 특정 위치에서 특별한 의미를 지니지만, 다른 위치에서는 일반 이름으로 사용가능한 키워드 (e.g. enum)

```kotlin
// 간단한 enum 클래스 정의하기
enum class Color {
    RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET
}

// 프로퍼티와 메소드가 있는 enum 클래스 선언하기
enum class Color(
    val r: Int, val g: Int, val b: Int  // 상수의 프로퍼티를 정의
) {
    RED(255, 0, 0), ORANGE(255, 165, 0),    // 각 상수를 생성할 때 그에 대한 프로퍼티 값을 정의
    YELLOW(255, 255, 0), GREEN(0, 255, 0), BLUE(0, 0, 255),
    INDIGO(75, 0, 130), VIOLET(238, 130, 238);  // 세미콜론은 반드시 필요

    fun rgb() = (r * 256 + g) * 256 + b     // enum class 안에서 메소드를 정의
}
>>> println(Color.BLUE.rgb())
255
```

### 2.3.2 when으로 enum 클래스 다루기

Java의 `switch`를 대체한다

```kotlin
fun getMnemonic(color: Color) = // 함수의 반환 값을 when식을 직접 사용
    when (color) {              // 색이 특정 enum 상수와 같은 때 그 상수에 대응하는 문자열을 돌려준다
        Color.RED -> "Richard"
        Color.ORANGE -> "Of"
        Color.YELLOW -> "York"
        Color.GREEN -> "Gave"
        Color.BLUE -> "Battle"
        Color.INDIGO -> "In"
        Color.VIOLET -> "Vain"
    }
>>> println(getMnemonic(Color.Blue))
Battle
```

### 2.3.3 when과 임의의 객체를 함께 사용

when의 분기 조건에 *임의의 객체*를 허용한다

```kotlin
import ch02.colors.Color
import ch02.colors.Color.*

fun mix(cl: Color, c2: Color) =
    when (setOf(c1, c2)) {               // when 식의 인자로 아무 객체나 사용할 수 있다
                                        // when은 이렇게 인자로 받은 객체가 각 분기 조건에 있는 객체와 같은지 테스트한다
        setOf(RED, YELLOW) -> ORANGE    // 두 색을 혼합해서 다른 색을 만들 수 있는 경우를 열거
        setOf(YELLOW, BLUE) -> GREEN
        setOf(BLUE, VIOLET) -> INDIGO
        else -> throw Exception("Dirty color")  // 매치되는 분기 조건이 없으면 이 문장을 실행
    }
```

### 2.3.4 인자 없는 when 사용

2.3.3에서 만들었던 함수는 호출될 때마다 함수 인자로 주어진 두 색이 when의 분기 조건에 있는 다른 두 색과 같은지 비교하기 위해 여러 Set 인스턴스를 생성한다. 이 함수가 아주 자주 호출된다면 불필요한 Garbage 객체가 들어나기에 함수를 고쳐야한다.

'인자 없는 when' 구문을 사용해 위와 같은 불필요한 객체 생성을 막을 수 있다. 다만, 코드의 가독성이 떨어질 수 있다.

```kotlin
fun mixOptimized(c1: Color, c2: Color) {
    when {
        (c1 == RED && c2 == YELLOW) ||  // when에 아무 인자도 없으려면
        (c1 == YELLOW && c2 == RED) ->  // 각 분기의 조건이 Boolean 결과를 계산하는 식이어야 한다
            ORANGE

        (c1 == YELLOW && c2 == BLUE) ||
        (c1 == BLUE && c2 == YELLOW) ->
            GREEN

        (c1 == BLUE && c2 == VIOLET) ||
        (c1 == VIOLET && c2 == BLUE) ->
            INDIGO
    }
}
>>> println(mixOptimization(BLUE, YELLOW))
GREEN
```

### 2.3.5 스마트 캐스트: 타입 검사와 타입 캐스트 조합

간단한 산술식을 계산하는 함수 만들기

```kotlin
// 식을 표현하는 클래스 계층
interface Expr  // 여러 타입의 식 객체를 아우르는 공통 타입 역할만을 수행
class Num(val value: Int) : Expr    // value라는 프로퍼티만 존재하는 단순한 클래스로 Expr 인터페이스를 구현
class Sum(val left: Expr, val right: Expr) : Expr   // Expr 타입의 객체라면 어떤 것이나 Sum 연산의 인자가 될 수 있다.
                                                    // 따라서 Num이나 다른 Sum이 인자로 올 수 있다
```

위 클래스 계층을 통해 `(1 + 2) + 4`라는 식을 저장하면 `Sum(Sum(Num(1), Num(2)), Num(4))`라는 구조의 객체가 생긴다

```kotlin
// Java 식으로 구현한 eval 함수
fun eval (e: Expr) : Int {
    if (e is Num) {
        val n = e as Num    // Num으로의 타입 변환이 불필요함에도 중복으로 해준다
        return n.value
    }
    if (e is Sum) {
        return eval(e.right) + eval(e.left) // 변수 e에 대해 스마트 캐스트를 사용한다
    }
    throw IllegalArgumentException("Unknown expression")
}
>>> println(eval(Sum(Sum(Num(1), Num(2)), Num(4))))
7
```

kotlin의 컴파일러는 어떤 변수가 원하는 타입인지 일단 `is`로 검사하고 나면 굳이 변수를 원하는 타입으로 캐스팅하지 않아도 알아서 원하는 타입으로 캐스팅을 수행해준다. 이를 **스마트 캐스트**<sup>*smart cast*</sup>라고 부른다.

### 2.3.6 리팩토링: if를 when으로 변경

```kotlin
// 값을 만들어내는 if 식
fun eval (e: Expr): Int =
    if (e is Num) {
        e.value
    } else if (e is Sum) {
        eval(e.right) + eval(e.left)
    } else {
        throw IllegalArgumentException("Unknown expression")
    }

// if 중첩 대신 when 사용하기
fun eval(e: Expr) : Int =
    when (e) {
        is Num -> e.value   // e가 Num으로 스마트 캐스트 됨
        is Sum -> eval(e.right) + eval(e.left)  // e가 Sum으로 스마트 캐스트 됨
        else -> throw IllegalArgumentException("Unknown expression")
    }
```

### 2.3.7 if와 when의 분기에서 블록 사용

if나 when에 사용된 블록의 마지막 문장은 그 블록의 전체 결과가 되어 반환된다

**Q. return을 하지 않아도 반환되는 것인가?**

- '블록의 마지막 식이 블록의 결과'라는 규칙은 블록이 값을 만들어내야 하는 경우 항상 성립
- try, catch문에서도 마찬가지
- 함수에 대해서는 성립하지 않는다(void 함수는 존재한다)
- if나 when은 값을 반환한다

```kotlin
fun evalWithLogging(e: Expr): Int = // when의 결과가 여기로 반환된다
    when (e) {
        is Num -> {
            println("num: ${e.value}")
            e.value
        }
        is Sum -> {
            val left = evalWithLogging(e.left)
            val right = evalWithLogging(e.right)
            println("sum: $left + $right")
            left + right
        }
        else -> throw IllegalArgumentException("Unknown expression")
    }
>>> println(evalWithLogging(Sum(Sum(Num(1), Num(2)), Num(4))))
num: 1
num: 2
num: 1 + 2
num: 4
num: 3 + 4
```

## 2.4 대상을 이터레이션: while과 for 루프

### 2.4.1 while 루프

while과 do while loop

### 2.4.2 수에 대한 이터레이션: 범위와 수열

kotlin에서 범위<sup>range</sup> 선언

```kotlin
val oneToTen = 1..10    // 1부터 10까지 모두 포함하는 폐구간
100 downTo 1 step 2     // 100부터 1까지 2씩 감소하는 범위
                        // downTo: 역방향 수열, step: 증감 간격
0 until size            // 0부터 size-1까지
0 .. size-1             // 0부터 size-1까지
```

### 2.4.3 맵에 대한 이터레이션

```kotlin
val binaryReps = TreeMap<char, String>()    // Q. TreeMap이 뭐야..

for (c in 'A'..'F') {
    val binary = Integer.toBinaryString(c.toInt())
    binaryReps[c] = binary
}

for ((letter, binary) in binaryReps) {
    println("$letter = $binary")
}
```

### 2.4.4 in으로 컬렉션이나 범위의 원소 검사

`in` 연산자를 사용해 어떤 값이 범위에 속하는지 검사할 수 있다.(`!in`은 그 반대)

```kotlin
// in을 사용해 값이 범위에 속하는지 검사하기
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'

>>> println(isLetter('q'))
true
>>> println(isNotDigit('x'))
true

// when에서 in 사용하기
fun recognize(c: Char) = when (c) {
    in '0'..'9' -> "It's a digit"
    in 'a'..'z', in 'A'..'Z' -> "It's a letter" // ','(쉼표)는 or
    else -> "I don't know"
}
>>> println(recognize('8'))
It's a digit
```

## 2.5 코틀린의 예외 처리

> 자바와 달리 코틀린의 throw는 하나의 식<sub>expression</sub>이므로 다른 식에 포함될 수 있다

### 2.5.1 try, catch, finally

```kotlin
fun readNumber(reader: BufferedReader): Int? {  // 함수가 던질 수 있는 예외를 명시할 필요가 없다
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    }
    catch (e: NumberFormatException) {          // 예외 타입을 `:`의 오른쪽에 쓴다
        return null
    }
    finally {                                   // Java와 같다
        reader.close()
    }
}
>>> val reader = BufferedReader(StringReader("239"))
>>> println(readNumber(reader))
239
```

Java와 달리 `throws`절이 코드에 없다

### 2.5.2 try를 식으로 사용

```kotlin
// try를 식으로 사용하기
fun readNumber(reader: BufferedReader) {
    val number = try {  // 변수에 대입되는 try식의 값
        Integer.parseInt(reader.readLine()) // 이 식의 값이 try식의 값이 된다
    } catch (e: NumberFormatException) {
        return                              // 예외가 발생했을 때 함수를 끝낸다
    }
    println(number)
}
>>> val reader = BufferedReader(StringReader("not a number"))
>>> readNumber(reader)      // 아무것도 출력되지 않는다

// catch에서 값 반환하기
fun readerNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine()) // 예외가 발생하지 않으면 이 값을 사용
    } catch (e: NumberFormatException) {
        null                                // 예외가 발생하면 null 값을 사용
    }
    println(number)
}
>>> val reader = BufferedReader(StringReader("not a number))
>>> readerNumber(reader)
null        // 예외가 발생했으므로 함수가 null을 출력
```
