# 3장 함수 정의와 호출

- 컬렉션, 문자열, 정규식을 다루기 위한 함수
- 이름 붙인 인자, 디폴트 파라미터 값, 중위 호출 문법 사용
- 확장 함수와 확장 프로퍼티를 사용해 자바 라이브러리 적용
- 최상위 및 로컬 함수와 프로퍼티를 사용해 코드 구조화

## 3.1 코틀린에서 컬렉션 만들기

컬렉션? \
https://shinjekim.github.io/kotlin/2019/09/10/Kotlin-%EA%B8%B0%EB%B3%B8-%ED%83%80%EC%9E%85(Basic-Types)/

https://thinkground.studio/%EC%BD%94%ED%8B%80%EB%A6%B0-kotlin-%EC%BB%AC%EB%A0%89%EC%85%98-collection-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC/

```kotlin
val set = hashSetOf(1, 7, 53)
val list = arrayListOf(1, 7, 53)
val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three")  // to는 키워드가 아닌 함수
```

```kotlin
// 각 객체의 클래스 알아보기
>>> println(set.javaClass)
class java.util.HashSet
>>> println(list.javaClass)
class java.util.ArrayList
>>> println(map.javaClass)
class java.util.HashMap

// 모두 자바의 컬렉션이다
// 코틀린은 자신만의 컬렉션 기능을 제공하지 않는다
```

## 3.2 함수를 호출하기 쉽게 만들기

```kotlin
>>> val list = listOf(1, 2, 3)
>>> println(list)   // toString() 호출
[1, 2, 3]   // collection: list, prefix: '[', postfix: ']', seperator: ','
```

### 3.2.1 이름 붙인 인자

```kotlin
// 코틀린으로 작성한 함수를 호출할 때는 함수에 전달하는 인자 중 일부(또는 전부)의 이름을 명시할 수 있다.
joinToString(collection, seperator=" ", prefix=" ", postfix=".")
// 단, 자바로 작성한 코드를 호출할 때는 이름 붙인 인자를 사용할 수 없다.
// JDK 6화 호환되는 코틀린 컴파일러는 함수 시그니처의 파라미터 이름을 인식할 수 없다.
```

자바로 작성한 코드를 호출할 때는 코틀린으로 래핑해서 이름 붙인 인자를 사용할 수 있을거 같다.

### 3.2.2 디폴트 파라미터 값

오버로딩: 같은 이름의 메소드를 여러 개 가지면서 매개변수의 타입과 개수가 다르게 만드는 것

오버로딩을 하는 이유

1. 하위 호환성을 유지
2. API 사용자에게 편의 제공

오버로딩으로 인한 비용

1. 코드의 중복
2. 설명의 중복

```kotlin
// 디폴트 파라미터 값을 통해 위의 비용을 해소할 수 있다
// 디폴트 파라미터 값을 설정한 인자는 생략이 가능하다
fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ",",
    prefix: String = "",
    postfix: String = ""
): String

>>> joinToString(list, ", ", "", "")
1, 2, 3
>>> joinToString(list)
1, 2, 3
>>> joinToString(list, "; ")
1; 2; 3
// 디폴트 파라미터 값이 설정되어 있다면 인자의 이름을 붙여 인자의 순서에 상관없이 파라미터를 기술할 수 있다
>>> joinToString(list, postfix=";", prefix="# ")
# 1, 2, 3
```

#### @JvmOverloads annotation

디폴트 파리미터 값이라는 개념이 없는 자바에서는 코틀린 함수를 자바에서 호출해도 디폴트 파라미터 값이 주어진 인자라도 모두 명시해야 한다. 이때, 코틀린 함수에 `@JvmOverload` 어노테이션을 추가하면 코틀린 컴파일러가 자동으로 맨 마지막 파라미터로부터 파라미터를 하나씩 생략한 오버로딩한 자바 메소드를 추가해준다.

---
[개발이 하고 싶어요](https://hyeonstorage.tistory.com/185)

### 3.2.3 정적인 유틸리티 클랙스 없애기: 최상위 함수와 프로퍼티

1. 어느 한 클래스에 포함시키기 애매한 코드
2. 일부 연산에서 비슷하게 중요한 역할을 하는 클래스가 둘 이상 있을 때(?)
3. 중요한 객체는 하나뿐이지만 그 연산을 객체의 인스턴스 API에 추가해서 API를 너무 크게 만들고 싶지 않을 때

위와 같은 상황들에서 JDK의 collections와 같은 무의미한(또는 무상태인) 클래스를 생성하는 것을 피하기 위한 방법

```kotlin
// join.kt
package strings
fun joinToString(...): String {...}
```

JVM이 클래스 안에 들어있는 코드만을 실행할 수 있기 때문에 컴파일러는 이 파일을 컴파일할 때 새로운 클래스를 아래와 같이 정의해준다.

```java
// 자바에서 컴파일된 join.kt의 joinToString
package strings;
public class JoinKt {
    public static String JoinToString(...) {...}
}

// 위 클래스를 자바 코드에서 실제로 사용할 때
import strings.JoinKt;
...
JoinKt.joinToString(list, ", ", "", "");
```

#### 파일에 대응하는 클래스의 이름 변경하기

코틀린 최상위 함수가 포함되는 클래스의 이름을 바꾸고 싶을 때 `@JvmName` 어노테이션을 파일의 맨 앞, 패키지 이름 선언 이전에 위치시킨다.

```kotlin
@file:JvmName("StringFunctions")
package strings
fun joinToString(...): String {...}

/* java */
import strings.StringFunctions;
StringFunctions.joinToString(list, ", ", "", "");
```

#### 최상위 프로퍼티

1. 파일의 최상위 수준에 두는 프로퍼티
2. 정적 필드에 저장된다
3. 이를 활용해 코드에 상수를 추가할 수 있다
4. primitive 타입과 String 타입은 `const` 변경자를 이용해 `public static final` 필드로 컴파일 되도록 만들 수 있다

## 3.3 메소드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티

기존 Java API를 재작성하지 않고도 코틀린이 제공하는 기능을 사용할 수 있도록 해주는 **확장 함수<sup>extension function</sup>**.

- 어떤 클래스의 멤버 메소드인 거처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수
- **수신 객체 타입<sup>receiver type</sup>**의 클래스에 새로운 함수를 추가로 정의하는 거라 생각하면 될 거 같다

```kotlin
package strings
fun String.lastChar(): Char = this.get(this.length-1)
// fun <수신객체타입>.lastChar(): Char = <수신 객체>.get(<수신 객체>.length-1)
// Q. 수신 객체는 항상 'this'인가?

// 일반 메소드의 본문에서 this를 사용할 때와 마찬가지로 확장 함수 본문에도 this를 쓸 수 있다.
// 일반 메소드와 마찬가지로 확장 함수 본문에서도 this를 생략할 수 있다.
package strings
// 하지만 클래스 내부에서만 사용가능한 `private`나 `protected` 멤버를 사용할 수는 없다
fun String.lastChar(): Char = get(length-1)
```

- **수신 객체 타입<sup>receiver type</sup>**: 확장이 정의될 클래스의 타입
- **수신 객체<sup>receiver object</sup>**: 확장이 정의될 클래스에 속한 인스턴스 객체

```kotlin
>>> println("Kotlin".lastChar())
n
```

### 3.3.1 임포트와 확장 함수

확장 함수를 사용하기 위해서는 그 함수를 다른 클래스나 함수와 마찬가지로 임포트해야만 한다.

```kotlin
import strings.lastChar
val c = "Kotlin".lastChar()

// as 키워드를 통해 다른 이름으로 래핑도 가능하다
import strings.lastChar as last
val c = "Kotlin".last()
```

### 3.3.2 자바에서 확장 함수 호출

내부적으로 확장 함수는 수신 객체를 첫 번째 인자로 받는 정적 메소드다. 그렇기에 자바에서 확장 함수를 사용하기 위해서는 단지 해당 정적 메소드를 호출하면서 첫 번째 인자로 수신 객체를 넘기기만 하면 된다.

```java
char c = StringUtilKt.lastChar("Java");
```

### 3.3.3 확장 함수로 유틸리티 함수 정의

```kotlin
// joinToString()을 확장으로 정의하기
fun <T> Collections<T>.joinToString(
    separater: String=", ",
    prefix: String="",
    postfix: String=""
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
>>> val list = listOf(1, 2, 3)
>>> println(list.joinToString(separator="; ", prefix="(", postfix=")"))
(1; 2; 3)

>>> val list = arrayListOf(1, 2, 3)
>>> println(list.joinToString(" "))
1 2 3
```

### 3.3.4 확장 함수는 오버라이드할 수 없다

오버라이딩: 상위 클래스가 가지고 있는 메소드를 하위 클래스가 재정의해서 사용하는 것

1. 확장 함수는 클래스의 일부가 아니다.
2. 확장 함수는 클래스 밖에 선언된다.
3. 이름과 파라미터가 완전히 같은 확장 함수를 기반 클래스와 하위 클래스에 대해 정의해도 실제로는 확장 함수를 호출할 때 수신 객체로 지전한 변수의 **정적 타입에 의해** 어떤 확장 함수가 호출될지 결정된다.

```kotlin
fun View.showOff() = println("I'am a view!")
fun Button.showOff() = println("I'm a button!")

>>> val view: View = Button()
>>> view.showOff()  // 정적으로 결정되는 확장함수
I'm a view!
```

`view`가 가리키는 객체의 실제 타입이 `Button`이지만, 이 경우 `view`의 타입이 `View`이기 때문에 무조건 `View`의 확장 함수가 호출된다.

Q. 컴파일 당시에는 `view` 타입은 `View`이지만 runtime에서는 `Button`으로 바뀌는가?

클래스의 멤버 함수와 확장 함수의 이름이 같다면 우선순위가 더 높은 멤버 함수가 항상 호출되게 된다

---
[개발이 하고 싶어요](https://hyeonstorage.tistory.com/185)

### 3.3.5 확장 프로퍼티

확장 프로퍼티를 사용하면 기존 클래스 객체에 대한 프로퍼티 형식의 구문으로 사용할 수 있는 API를 추가할 수 있다. 하지만 실제로 확장 프로퍼티는 아무 상태도 가질 수 없다.

```kotlin
val String.lastChar: Char
    get() = get(length-1)
```

뒷바침하는 필드가 없기 때문에 기본 게터를 제공하지 않아 최소한 게터는 정의를 해줘야한다. 마찬가지로 계산한 값을 담을 장소가 없기에 초기화 코드도 쓸 수 없다.

```kotlin
var StringBuilder.lastChar: Char
    get() = get(length-1)               // getter
    set(value: Char) {
        this.setCharAt(length-1, value) // setter
    }

>>> println("Kotlin".lastChar)
n
>>> val sb = StringBuilder("Kotiln?")
>>> sb.lastChar = '!'
>>> println(sb)
Kotlin!
```

## 3.4 컬렉션 처리: 가변 길이 인자, 중위 함수 호출, 라이브러리 지원

- `varang` 키워드를 사용하면 호출 시 인자 개수가 달라질 수 있는 함수를 정의할 수 있다
- 중위<sup>infix</sup> 함수 호출 구문을 사용하면 인자가 하나뿐인 메소드를 간편하게 호출할 수 있다
- 구조 분해 선언<sup>destructuring declaration</sup>을 사용하면 복합적인 값을 분해해서 여러 변수에 나눠 담을 수 있다

### 3.4.1 자바 컬렉션 API 확장

last()와 max()는 확장 함수였습니다(짜잔)

### 3.4.2 가변 인자 함수: 인자의 개수가 달라질 수 있는 함수 정의

```kotlin
// 길이가 동적인 컬렉션을 인자로 넘길때
fun listOf<T>(vararg values: T): List<T> {...}
```

**스프레드 연산자(\*)**: 이미 배열에 들어있는 원소를 가변 길이 인자로 넘길 땐 배열을 명시적으로 풀어서 배열의 각 원소가 인자로 전달되게 하는 연산자

```python
# in Python
# Q. **kwargs에 해당하는 건 없나?
def get(self, *args, **kwargs):
    ...
```

### 3.4.3 값의 쌍 다루기: 중위 호출과 구조 분해 선언

```kotlin
val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
```

`to`: 중위 호출<sup>infix call</sup>을 이용해 호출된 일반 메소드

- 중위 호출 시에는 **수신 객체**와 **유일한 메소드 인자** 사이에 메소드 이름을 넣는다

```kotlin
to("one")   // "to" 메소드를 일반적인 방식으로 호출함
to "one"    // "to" 메소드를 중위 호출 방식으로 호출함
```

함수를 중위 호출로 사용하도록 허용하려면 `infix` 변경자를 함수 선언 앞에 추가해야 한다

```kotlin
infix fun Any.to(other: Any) = Pair(this, other)

val (number, name) = 1 to "one" // 구조 분해 선언
```

## 3.5 문자열과 정규식 다루기

### 3.5.1 문자열 나누기

코틀린에서는 자바의 `spilt` 대신에 여러 가지 다른 조합의 파라미터를 받는 `split` 확장 함수를 제공해 혼동을 야기하는 메소드를 감춘다.

- 정규식을 파라미터로 받는 함수는 `String`이 아닌 `Regex` 타입의 값을 받는다.

```kotlin
>>> println("12.345-6.A".split("\\/|-"/toRegex()))  // 정규식을 명시적으로 만든다
[12, 345, 6, A]
```

### 3.5.2 정규식과 3중 따옴표로 묶은 문자열

```kotlin
fun parsePath(path: String) {
    val regex = """(.+)/(.+)\.(.+)""".toRegex()     // 3중 따옴표 문자열에서는 역슬래시(\)를 포함한 어떤 문자도 이스케이프할 필요가 없다.
    val matchResult = regex.matchEntire(path)
    if (matchResult != null) {
        val (directory, filename, extension) = matchResult.destructed
        println("Dir: $directory, name: $filename, ext: $extension")
    }
}
```

### 3.5.3 여러 줄 3중 따옴표 문자열

줄바꿈이 있는 문자열이 3중 따옴표에는 그대로 들어갈 수 있다,

```kotlin
val kotlinLogo = """ |  //
                    .| //
                    .|/ \"""
>>> println(kotlinLogo.trimMargin("."))
|  //
| //
|/ \
```

## 3.6 코드 다듬기: 로컬 함수와 확장

코틀린에서는 함수에서 추출한 함수를 원 함수 내부에 중첩시킬 수 있다.

```kotlin
class User(val id: Int, val name: String, val address: String)
fun saveUser(user: User) {
    fun validate(user: User,
                 value: String,
                 fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
                "Can't save user ${user.id}: empty $fieldName"
            )
        }
    }
    validate(user, user.name, "Name")
    validate(userm user.address, "Address")
}
```

로컬 함수는 자신이 속한 바깥 함수의 모든 파라미터와 변수를 사용할 수 있다.

```kotlin
class User(val id: Int, val name: String, val address: String)
fun saveUser(user: User) {
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
                "Can't save user ${user.id}: " +
                    "empth $fieldName"
            )
        }
    }
    validate(user.name, "Name")
    validate(user.address, "Address")
}
```

확장 함수로 클래스에 종속 시킬 수 있다.

```kotlin
class User(val id: Int, val name: String, val address: String)
fun User.validateBeforeSave() {
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
                "Can't save user $id: empty $fieldName"
            )
        }
    }
    validate(name, "Name")
    validate(address, "Address")
}
fun saveUser(user: User) {
    user.validateBeforeSave()
}
```

파이썬의 디스크립터처럼 사용할 수 도 있을 거 같다.
