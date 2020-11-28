# 4장 클래스, 객체, 인터페이스

- 클래스와 인터페이스
- 뻔하지 않은 생성자와 프로퍼티
- 데이터 클래스
- 클래스 위임
- object 키워드 사용

## 4.1 클래스 계층 정의

코틀린 가시성/접근 변경자와 sealed 변경자

### 4.1.1 코틀린 인터페이스

> 코틀린 인터페이스 안에는 추상 메소드뿐 아니라 구현이 있는 메소드도 정의할 수 있다. 다만 인터페이스에는 아무런 상태(필드)도 들어갈 수 없다.

```kotlin
// 인터페이스의 선언
interface Clickable {   // 이를 구현하는 모든 비추상 클래스는 click에 대한 구현을 제공해야 한다
    fun click()
}

// 인터페이스 구현하기
class Button: Clickable {   // 콜론(:) 뒤에 인터페이스 또는 클래스 이름을 적는 것으로
                            // extends와 implements의 역할을 할 수 있다
    override fun click() = println("I was clicked") // @Override
}
>>> Button().click()
I was clicked
```

- 자바와 마찬가지로 클래스는 인터페이스를 원하는 만큼 구현할 수 있지만, 클래스는 오직 하나만 확장할 수 있다.
- 상위 클래스나 상위 인터페이스를 확장하거나 구현할 때는 반드시`override` 변경자를 선언해야 한다.
- `override` 변경자는 실수로 상위 클래스의 메소드를 오버라이드하는 경우를 방지해준다.

```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable!")
}

interface Focusable {
    fun setFocus(b: Boolean) = println("I ${if (b) "got" else "lost"} focus.")
    fun showOff() = println("I'm focusable!")
}

class Button: Clickable, Focusable {
    override fun click() = println("I was clicked")
    override fun showOff() {        // 이름과 시그니처가 같은 멤버 메소드에 대해 둘 이상의
                                    // 디폴트 구현이 있는 경우 인터페이스를 구현하는
                                    // 하위 클래스에서 명시적으로 새로운 구현을 제공해야 한다.
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

### 4.1.2 open, final, abstract 변경자: 기본적으로 final

코틀린의 클래스와 메소드는 기본적으로 `final`이다. 어떤 클래스이 상속을 허용하려면 클래스 앞에 `open` 변경자를 붙여야 한다. 그와 더불어 오버라이드를 허용하고 싶은 메소드나 프로퍼티 앞에서 `open` 변경자를 붙여야 한다.

```kotlin
open class RichButton: Clickable {  // 이 열려있기 때문에 다른 클래스가 상속할 수 있다
    fun disable() {}                // 이 함수는 final이기 때문에 하위클래스가 이 메소드를 오버라이드 할 수 없다
    open fun animate() {}           // 이 함수는 열려있기 때문에 하위 클래스에서 이 메소드를 오버라이드 할 수 있다
    override fun click() {}         // 오버라이드한 메소드는 기본적으로 열려있다
    // 오버라이드된 메소드를 하위 클래스에서 구현하지 못하게 하려면 앞에 `final` 변경자를 붙인다
    final override showOff() = super<Clickable>.showOff()
}
```

- 코틀린도 자바처럼 `abstract`로 추상 클래스를 선언할 수 있다.
- 추상 클래스는 인스턴스화 할 수 없다.
- 추상 멤버는 항상 열려있다.

```kotlin
abstract class Animated {
    abstract fun animate()      // 추상 함수는 하위 클래스에서 반드시 구현되어야 한다
    fun stopAnimating() {} // 추상 클래스에 속하는 비추상 함수는 기본적으로 final이다
    open fun animateTwice() {}       // 하지만 open으로 열 수 있다
}
```

### 4.1.3 가시성 변경자: 기본적으로 공개

> **가시성 변경자<sup>visibility modifier</sup>는 코드 기반에 있는 선언에 대한 클래스 외부 접근을 제어한다. 어떤 클래스의 구현에 대한 접근을 제한함으로써 그 클래스에 의존하는 외부 코드를 깨지 않고도 클래스 내부 구현을 변경할 수 있다.

**모듈<sup>module</sup>** 한 번에 한꺼번에 컴파일되는 코틀린 파일들

변경자 | 클래스 멤버 | 최상위 선언
---|---|---
public(default) | 모든 곳에서 볼 수 있다. | 모든 곳에서 볼 수 있다.
internal | 같은 모듈 안에서만 볼 수 있다. | 같은 모듈 안에서만 볼 수 있다.
protected | 하위 클래스 안에서만 볼 수 있다. | (최상위 선언에 적용할 수 없음)
private | 같은 클래스 안에서만 볼 수 있다. | 같은 파일 아니에서만 볼 수 있다.

```kotlin
internal open class TalkativeButton: Focusable {
    private fun yell() = println("Hey")
    protected fun whisper() = println("Let's talk")
}
fun TalkativeButton.giveSpeech() {  // Error: "public" 멤버가 자신의 "internal" 수신 타입인 "TalkativeButton"을 노출함
    yell()      // Error: "yell"에 접근할 수 없음: "yell"은 "TalkativeButton"의 "private" 멤버임
    whisper()   // Error: "whisper"에 접근할 수 없음: "whisper"는 "TalkativeButton"의 "protected" 멤버임
}
```

> 자바에서는 같은 패키지 안에서 `protected` 멤버에 접근할 수 있지만, 코틀린에서는 그렇지 않다는 점에서 자바와 코틀린의 `protected`가 다르다는 사실에 유의하라.

### 4.1.4 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스

> 자바처럼 코틀린에서도 클래스 안에 다른 클래스를 선언할 수 있다. 중첩 클래스는 명시적으로 요청하지 않는 한 바깥쪽 클래스 인스턴스에 대한 접근 권한이 없다.

- 자바에서 다른 클래스 안에 정의한 클래스는 자동으로 내부 클래스가 된다.
- 외부의 인터페이스나 상위 클래스를 내부 클래스로 구현이나 확장하면 내부 클래스를 참조하려 할때 외부의 인터페이스나 상위 클래스를 참조하기에 직렬화를 할 수 없다는 에러가 발생한다.
- 위 문제를 해결하려면 내부 클래스를 `static`으로 선언해야 한다.

```kotlin
class Button: View {
    override fun getCurrentState(): State = ButtonState()
    override fun restoreState(state: State) {...}
    class ButtonState: State {...}      // 자바 static 중첩 클래스와 같다.
                                        // 바깥쪽 클래스에 대한 참조를 포함하게 만들고 싶으면 inner 변경자를 붙여야 한다.
}
```

클래스 B안에 정의된 클래스 A | in Java | in Kotlin
---|---|---
중첩 클래스(바깥쪽 클래스에 대한 참조를 저장하지 않음) | static class A | class A
내부 클래스(바깥쪽 클래스에 대한 참조를 저장함) | class A | inner class A

`inner` 클래스 안에서 바깥쪽 클래스 outer에 접근하려면 `this@Outer`라고 써야한다.

```kotlin
class Outer {
    inner class Inner {
        fun getOuterReference(): Outer = this@Outer
    }
}
```

### 4.1.5 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한

```kotlin
// 인터페이스 구현을 통해 식 표현하기
interface Expr
class Num(val value: Int): Expr
class Sum(val left: Expr, val right: Expr): Expr
fun eval(e: Expr): Int =
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.right) + eval(e.left)
        else -> throw IllegalArgumentException("Unknown expression")
    }
```

`sealed` 변경자를 상위 클래스에 붙이면 그 상위 클래스를 상속한 하위 클래스 제한할 수 있다. `sealed` 클래스의 하위 클래스를 정의할 때는 반드시 상위 클래스 안에 중첩시켜야 한다.

```kotlin
sealed class Expr {     // 기반 클래스를 sealed로 봉인한다.
    class Num(val value: Int): Expr()   // 기반 클래스의 모든 하위 클래스를 중첩 클래스로 나열한다.
    class Sum(val left: Expr, val right: Expr): Expr()
}
fun eval(e: Expr): Int =
    when (e) {      // when 식이 모든 하위 클래스를 검사하므로 별도의 else 분기가 없어도 된다.
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.right) + eval(e.left)
    }
```

- `when` 식에서 `sealed` 클래스의 모든 하위 클래스를 처리한다면 `else`와 같은 디폴트 분기가 필요 없다.
- `sealed`로 표시된 클래스는 자동으로 `open` 처리되어 `open` 변경자를 별도로 선언할 필요 없다.
- `sealed` 클래스에 속한 값에 대해 디폴트 분기를 사용하지 않고 `when` 식을 사용하면 나중에 `sealed` 클래스의 상속 계층에 새로운 하위 클래스를 추가해도 `when` 식이 컴파일되지 않는다.

### 4.2 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

자바와 마찬가지로 코틀린에서도 생성자를 하나 이상 선언할 수 있다.

- 코틀린에서는 **주 생성자**와 **부 생성자**를 구분한다.
- 코틀린에서는 **초기화 블록**을 통해 초기화 로직을 추가할 수 있다.

### 4.2.1 클래스 초기화: 주 생성자와 초기화 블록

#### 주 생성자

- 생성자 파라미터를 지정
- 그 생성자 파라미터에 의해 초기화되는 프로퍼티를 정의

```kotlin
// 간단한 클래스 선언
class User(val nickname: String)    // 주 생성자 '(val nickname: String)'

// 실제로 행해지는 일
class User constructor(_nickname: String) { // constructor 키워드는 주 생성자나 부 생성자 정의를 시작할 때 사용
    val nickname: String
    init {  // python의 __init__, 하지만 여러개 생성 가능
        nickname = _nickname
    }
}
```

> 모든 생성자 파라미터에 디폴트 값을 지정하면 컴파일러가 자동으로 파라미터가 없는 생성자를 만들어준다.

클래스에 기반 클래스가 있다면 주 생성자에서 기반 클래스의 생성자를 호출해야 할 필요가 있다.

```kotlin
open class User(val nickname: String) {...}
class TwitterUser(nickname: String): User(nickname) {...}
```

- 클래스를 상속할 때는 부모 클래스의 생성자를 호출해야 한다.
- 인터페이스를 구현하는 경우에는 인터페이스 자체가 생성자를 가지지 않기 때문에 별도의 괄호가 필요하지 않다

### 4.2.2 부 생성자: 상위 클래스를 다른 방식으로 초기화

```kotlin
open class View {   // 주 생성자가 없는 클래스
    constructor(ctx: Context) {...}     // 부 생성자를 선언할 땐 constructor 키워드로 선언한다.
    constructor(ctx: Context, attr: AttributeSet) {...}
}

// 부 생성자를 상속할 수 있다
class MyButton: View {
    constructor(ctx: Context): super(ctx) {...}
    constructor(ctx: Context, attr: AttributeSet): super(ctx, attr) {...}
}

// 자바와 마찬가지로 생성자에서 this()를 통해 클래스 자신의 다른 생성자를 호출할 수 있다.
class MyButton: View {
    constructor(ctx: Context): this(ctx, MY_STYLE) {...}
    constructor(ctx: Context, attr: AttributeSet): super(ctx, attr) {...}
}
```

- 클래스에 주 생성자가 없다면 모든 부 생성자는 반드시 상위 클래스를 초기화하거나 다른 생성자에게 생성을 위임해야 한다.

### 4.2.3 인터페이스에 선언된 프로퍼티 구현

> 코틀린에서는 인터페이스에 추상 프로퍼티 선언을 넣을 수 있다.

```kotlin
interface User {
    val nickname: String    // 해당 인터페이스를 구현하는 클래스는 nickname의 값을 얻을 수 있는 방법을 제공해야 한다
}
```

```kotlin
class PrivateUser(override val nickname: String): User  // 주 생성자 선언으로 게터 생성
class SubscribingUser(val email: String): User {        // 커스텀 게터 생성
    override val nickname: String
        get() = email.substringBefore('@')
}
class FacebookUser(val accountId: Int): User {
    override val nickname = getFacebookName(accountId)  // 프로퍼티 초기화 식
}
>>> println(PrivateUser("test@kotlinlang.org").nickname)
test@kotlinlang.org
>>> println(SubscribingUser("test@kotlinlang.org").nickname)
test
```

### 4.2.4 게터와 세터에서 뒷받침하는 필드에 접근

어떤 값을 저장하되 그 값을 변경하거나 읽을 때마다 정해진 로직을 실행하는 유형의 프로퍼티를 만드는 방법

```kotlin
// 세터에서 뒷받침하는 필드 접근하기
class User(val name: String) {
    var address: String = "unspecified"
        set(value: String) {
            println("""
                Address was changed for $name:
                "$field" -> "$value".""".trimIndect()
            )
            field = value   // 'field' 키워드로 backing field에 접근
        }
}
>>> val user = User("Alice")
>>> user.address = "Elsenheimerstrasse 47, 80687 Munchen"
Address was changed for Alice:
"unspecified" -> "Elsenheimerstrasse 47, 80687 Munchen".
```

### 4.2.5 접근자의 가시성 변경

> 접근자의 가시성은 기본적으로는 프로퍼티의 가시성과 같다. 하지만 변경할 수는 있다.

```kotlin
class LengthCounter {
    var counter: Int = 0
        private set     // 이 클래스 밖에서 이 프로퍼티의 값을 바꿀 수 없다.
    fun addWord(word: String) {
        counter += word.length
    }
}
```

게터만 public이므로 counter 값을 `addWord` 메소드 이외의 외부의 방법으로는 바꿀 수 없다.

## 4.3 컴파일러가 생성한 메소드: 데이터 클래스와 클래스 위임

> 자바 플랫폼에서는 클래스가 equals, hashCode, toString 등의 메소드를 구현해야 한다. 코틀린에서는 이런 메소드를 기계적으로 생성하는 작업을 보이지 않는 곳에서 해준다.

### 4.3.1 모든 클래스가 정의해야 하는 메소드

> 자바와 마찬가지로 코틀린 클래스도 toString, equals, hashCode 등을 오버라이드 할 수 있다.

고객 이름과 우편번호를 저장하는 간단한 Client 클래스 예제

```kotlin
class Client(val name: String, val postalCode: Int)
```

#### 문자열 표현: toString()

코틀린에서는 기본적으로 만들어주기는 하지만 만족스럽지는 않을 것이다. 따라서 오버라이드를 해주는 것이 좀 더 유용하게 사용할 수 있을 것이다.

```kotlin
class Client(val name: String, val postalCode: Int) {
    override fun toString() = "Client(name=$name, postalCode=$postalCode)"
}
>>> val client1 = Client("오현석", 4122)
>>> println(client1)
Client(name=오현석, postalCode=4122)
```

#### 객체의 동등성: equals()

서로 다른 두 객체가 **내부에 동일한 데이터**를 포함하는 경우 그 둘을 **동등한 객체**로 간주해야 할 수도 있다.

```kotlin
// 하지만, 위와 같은 상태에서는 같지 않다
>>> val client1 = Client("오현석", 4122)
>>> val client2 = Client("오현석", 4122)
>>> println(client1 == client2) // 코틀린에서 '==' 연산자는 참조 동일성을 검사하지 않고
                                // 객체의 동등성을 검사한다. 따라서 '==' 연산은 equals를
                                // 호출하는 식으로 컴파일된다.
false
```

위 == 연산의 결과를 true가 되게 하고 싶다면 `equals`를 오버라이드 해야한다.

##### 동등성 연산에 ==를 사용함

자바에서 참조 타입의 동등성을 비교하려고 == 연산자를 사용하면 각 인스턴스의 주소를 비교하게 된다. 따라서 자바에서는 `equals`를 호출해 비교해야 한다.

하지만 코틀린에서는 == 연산자를 두 객체를 비교하기 위해 사용하면 내부적으로 `equals`를 호출해서 객체를 비교한다. 따라서 클래스가 equals를 오버라이드하면 ==를 통해 안전하게 클래스의 인스턴스를 비교할 수 있다.

그리고 코틀린에서 참조 비교를 할 때는 === 연산자를 사용할 수 있다. 이는 자바에서 객체의 참조를 비교할 때 사용하는 == 연산자와 같다.

``` kotlin
class Client(val anme: String, val postalCode: Int) {
    override fun equals(other: Any?): Boolean { // Any는 자바에서 java.lang.Object에 해당 하는 클래스이다.
                                                // 코틀린의 모든 클래스이 최상위 클래스.
        if (other == null || other !is Client)  // other가 Client인지 검사한다.
            return false
        // 두 객체의 프로퍼티 값이 서로 같은 지를 검사한다.
        return name == other.name && postalCode == other.postalCode
    }
    override fun toString() = "Client(name=$name, postalCode=$postalCode)"
}
```

- 코틀린의 `is` 검사는 자바의 `instanceof`와 같다. 둘다 어떤 타입의 값을 검사한다.

Q. override 변경자가 필수인 것과 override fun equals(other: Client)를 작성할 수 없는 것과는 어떤 관련이 있는가?

#### 해시 컨테이너: hashCode()

자바에서 `equals`를 오버라이드할 때 반드시 `hashCode`도 함께 오버라이드해야 한다.

```kotlin
>>> val processed = hashSetOf(Client("오현석", 4122))
>>> println(processed.contains(Client("오현석", 4122)))
false
```

위 경우는 `Client` 클래스가 `hashCode` 메소드를 정의하지 않았기 때문이다. JVM 언어에서는 `hashCode`가 지켜야 하는 "**equals()가 true를 반환하는 두 객체는 반드시 같은 hashCode()를 반환해야 한다.**"라는 제약이 있는데 Client는 이를 어기고 있다.

`processed` 집합은 `HashSet`이다. `HashSet`은 원소를 비교할 때 비용을 줄이기 위해 **먼저 객체의 해시 코드를 비교**하고 해시 코드가 같은 경우에만 실제 값을 비교한다.

```kotlin
class Client(val name: String, val postalCode: Int) {
    ...
    override fun hashCode(): Int = name.hashCode() * 31 + postalCode
}
```

코틀린에서는 위 과정을 생략할 수 있게 해준다.

### 4.3.2 데이터 클래스: 모든 클래스가 정의해야 하는 메소드 자동 생성

어떤 클래스가 데이터를 저장하는 역할만을 수행한다면 `toString`, `equals`, `hashCode`를 반드시 오버라이드해야 한다. 코틀린에서는 `data`라는 변경자를 클래스 앞에 붙이는 것으로 필요한 메소드를 컴파일러가 자동으로 만들게 할 수 있다. `data` 변경자가 붙은 클래스는 **데이터 클래스**라고 부른다.

```kotlin
data class Client(val name: String, val postalCode: Int)
```

이제 위 Client 클래스는 자바에서 요구하는 모든 메소드를 포함한다.

- 인스턴ㄴ스 간 비교를 위한 `equals`
- `HashMap`과 같은 해시 기반 컨테이너에서 키로 사용할 수 있는 `hashCode`
- 클랫의 각 필드를 선언 순서대로 표시하는 문자열 표현을 만들어주는 `toString`

> `equals`와 `hashCode`는 **주 생성자**에 나열된 모든 프로퍼티를 고려해 만들어진다. 따라서 주 생성자 밖에 정의된 프로퍼티는 `equals`나 `hashCode`를 계산할 때 고려의 대상이 아니라는 사실에 유의해야 한다.

코틀린 컴파일러는 `data` 클래스에게 방금 말한 세 메소드뿐 아니라 몇 가지 유용한 메소드를 더 생성해준다.

#### 데이터 클래스와 불변성: copy() 메소드

> 데이터 클래스의 모든 프로퍼티가 꼭 `val`일 필요는 없으나 데이터 클래스는 모든 프로퍼티를 읽기 전용으로 만드는 것이 권장된다. `HashMap` 등의 컨테이너에 데이터 클래스 객체를 담는 경우엔 불변성이 필수적이다.

`copy` 메소드는 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있게 해준다.

```kotlin
>>> val lee = Client("이계영", 4122)
>>> println(lee.copy(postalCode = 4000))    // 복사된 후 프로퍼티 값이 바뀐 Client 인스턴스를 반환한다.
Client(name=이계영, postalCode=4000)
```

### 4.3.3 클래스 위임: by 키워드 사용

> 하위 클래스가 상위 클래스의 메소드 중 일부를 오버라이드하면 하위 클래스는 상위 클래스의 세부 구현 사항에 의존하게 된다. 시스템이 변함에 따라 하위 클래스가 상위 클래스에 대해 갖고 있던 가정이 깨져서 코드가 정상적으로 작동하지 못하는 경우가 생길 수 있다.

따라서 코틀린에서는 모든 클래스를 기본적으로 `final`로 취급하여 상속을 염두에 두고 `open` 변경자로 열어둔 클래스만 확장할 수 있게 했다.

#### 데코레이터 패턴

상속을 허용하지 않는 클래스(기존 클래스) 대신 사용할 수 있는 새로운 클래스(데코레이터)를 만들되 기존 클래스와 같은 인터페이스를 데코레이터가 제공하게 만들고, **기존클래스를 데코레이터 내부에 필드로** 유지하는 것이다.

- 새로 정의해야 하는 기능은 데코레이터의 메소드에 새로 정의한다.
- 기본 기능이 그대로 필요한 부분은 데코레이터의 메소드가 기존 클래스이 메소드에게 요청을 전달한다.

```kotlin
class DelegatingCollection<T>: Collection<T> {
    private val innerList = arrayListOf<T>()k

    override val size: Int get() = innerList.size
    override fun isEmpty(): Boolean = innerList.isEmpty()
    override fun contains(element: T): Boolean = innerList.contains(element)
    override fun iterator(): Iterator<T> = innerList.iterator()
    override fun containsAll(elements: Collection<T>): Boolean = innerList.containsAll(elements)
}

// 위 코드를 by 키워드를 통해 위임을 사용해 구현
class DelegatingCollection<T> {
    innerList: Collection<T> = ArrayList<T>()
}: Collection<T> by innerList {}   // Collection 인터페이스에 대한 구현을 innerList 객체에 위임 중
```

- 메소드 중 일부의 동작을 변경하고 싶은 경우 메소드를 오버라이드하면 컴파일러가 생성한 메소드 대신 오버라이드한 메소드가 쓰인다.

```kotlin
class CountingSet<T>(
    val innerSet: MutableColletion<T> = HashSet<T>()
): MutableCollection<T> by innerSet {   // MutableCollection의 구현을 innerSet에 위임
    var objectsAdded = 0
    override fun add(element: T): Boolean { // 해당 메소드는 위임하지 않고 새롭게 구현
        objectsAdded++
        return innerSet.add(element)
    }
    override fun addAll(c: Collection<T>): Boolean { // 해당 메소드는 위임하지 않고 새롭게 구현
        objectsAdded += c.size
        return innerset.addAll(c)
    }
}
>>> val cset = CountingSet<Int>()
>>> cset.addAll(listOf(1, 1, 2))
>>> println("${cset.objectsAdded} objects were added, ${cset.size} remain")
3 objects were added, 2 remain
```

## 4.4 object 키워드: 클래스 선언과 인스턴스 생성

> 코틀린에서는 object 키워드를 다양한 상황에서 사용하지만 모든 경우 클래스를 정의하면서 동시에 인스턴스(객체)를 생성한다는 공통점이 있다.

- **객체 선언<sup>object declaration</sup>** 싱글턴을 정의하는 방법 중 하나이다.
- **동반 객체<sup>companion object</sup>** 인스턴스 메소드는 아니지만 어떤 클래스와 관련있는 메소드와 팩토리 메소드를 담을 때 쓰인다. 동반 객체 메소드에 접근할 때는 동반 객체가 포함된 클래스의 이름을 사용할 수 있다.
- 객체 식은 자바의 **무명 내부 클래스<sup>anonymous inner class</sup>** 대신 쓰인다.

### 4.4.1 객체 선언: 싱글턴을 쉽게 만들기

코틀린은 **객체 선언** 기능을 통해 싱글컨을 언어에서 기본 지원한다.

```kotlin
object Payroll {
    val allEmployees = arrayListOf<Person>()
    fun calculateSalary() {
        for (person in allEmployees) {...}
    }
}
```

- 객체 선언은 object 키워드로 시작한다.
- 객체 선언은 **클래스를 정의**하고 그 클래스의 **인스턴스를 만들어서 변수에 저장**하는 모든 작업을 **단 한 문장**으로 처리한다.
- 클래스와 마찬가지로 객체 선언 안에도 프로퍼티, 메소드, 초기화 블록 등이 들어갈 수 있다.
- 생성자는 객체 선언에 쓸 수 없다.
- 변수와 마찬가지로 객체 선언에 사용한 이름 뒤에 마침표(.)를 붙이면 객체에 속한 메소드나 프로퍼티에 접근할 수 있다.

```kotlin
Payroll.allEmployees.add(Person(...))
payroll.calculateSalary()
```

- 객체 선언도 클래스나 인터페이스를 상속할 수 있다. 구현 내부에 다른 상태가 필요하지 않은 경우에 사용하면 좋다.

```kotlin
object CaseInsensitiveFileComparator: Comparator<File> {
    override fun compare(file1: File, file2: File): Int {
        return file1.path.compareTo(file2.path, ignoreCase=true)
    }
}
>>> println(CaseInsenseitiveFileComparator.compare(File("/User"), File("/user")))
0
```

- 일반 객체(클래스 인스턴스)를 사용할 수 있는 곳에서는 항상 싱글턴 객체를 사용할 수 있다.

```kotlin
>>> val files = listOf(File("/Z"), File("/a"))
// Comparator를 인자로 받는 함수인 List.sortedWith에 CaseInsensitiveFileComparator를 인자로 넘긴다.
>>> println(files.sortedWith(CaseInsensitiveFileComparator))
[/a, /Z]
```

- 클래스의 안에 객체를 선언해 클래스 인스턴스마다 새로운 중첩 객체가 생성되는 것을 방지할 수 있다.

```kotlin
data class Person(val name: String) {
    objects NameComparator: comparator<Person> {
        override fun compare(p1: Person, p2: Person): Int =
            p1.name.compareTo(p2.name)
    }
}
>>> val persons = listOf(Person("Bob"), Person("Alice"))
>>> println(persons.sortedWith(Person.NameComparator))
[Person(name=Alice), Person(name=Bob)]
```

#### 싱글턴과 의존 관계 주입

대규모 소프트웨어에서는 싱글턴 패턴이나 객체 선언이 적합하지 않을 수 있다.

### 4.4.2 동반 객체: 팩토리 메소드와 정적 멤버가 들어갈 장소

> 코틀린 클래스 안에는 정적인 멤버가 없다. 코틀린 언어는 자바의 `static` 키워드를 지원하지 않기 때문이다. 그 대신 코틀린에서는 패키지 수준의 **최상위 함수**와 **객체 선언**을 활용한다.

- **최상위 함수**는 자바의 정적 메소드 역할을 거의 대신할 수 있다.
- **객체 선언**은 자바의 정적 메소드 역할 중 코틀린 최상위 함수가 대신할 수 없는 역할이나 정적 필드를 대신할 수 있다.

클래스 안에 정의된 객체 중 하나에 `companion`을 붙이면 그 클래스의 **동반 객체**로 만들 수 있다. 동반 객체의 프로퍼티나 메소드에 접근하려면 그 동반 객체가 정의된 클래스의 이름을 사용한다.

```kotlin
class A {
    companion object {
        fun bar() {
            println("Companion object called")
        }
    }
}
>>> A.bar()
Companion object called
```

- 동반 객체는 자신을 둘러싼 클래스의 모든 `private` 멤버에 접근할 수 있다.

```kotlin
// 부 생성자를 팩토리 메소드로 대신하기
class User private constructor(val nickname: String) {  // 주 생성자를 비공개로 만든다
    companion object {  // 동반 객체를 선언한다.
        fun newSubscribingUser(email: String) =
            User(email.substringBefore('@'))
        fun newFacebookUser(AccountId: Int) =   //  페이스북 사용자 ID로 사용자를 만드는 팩토리 메소드
            User(getFacebookName(accountId))
    }
}
>>> val subscribingUser = User.newSubscribingUser("bob@gmail.com")
>>> val facebookUser = User.newFacebookUser(4)
>>> println(subscribingUser.nickname)
bob
```

### 4.4.3 동반 객체를 일반 객체처럼 사용

> 동반 객체는 클래스 안에 정의된 일반 객체다.

- 동반 객체에 이름을 붙일 수 있다.
- 동반 객체가 인터페이스를 상속할 수 있다.
- 동반 객체 안에 확장 함수와 프로퍼티를 정의할 수 있다.

```kotlin
class Person(val name: String) {
    companion object Loader {   // 동반 객체에 이름을 붙인다.
        fun fromJSON(jsonText: String): Person = ...
    }
}
>>> person = Person.Loader.fromJSON("{name: 'Dmitry'}")
>>> person.name
Dmitry
>>> person2 = Person.fromJSON("{name: 'Brent'}")
>>> person2.name
Brent
```

특별히 지정하지 않으면 동반 객체의 이름은 `Companion`이 된다.

#### 동반 객체에서 인터페이스 구현

```kotlin
// 동반 객체에서 인터페이스 구현하기
interface JSONFactory<T> {
    fun fromJSON(jsonText: String): T
}

class Person(val name: String) {
    companion object: JSONFactory<Person> {
        override fun fromJSON(jsonText: String): Person = ...   // 동반 객체가 인터페이스를 구현한다.
    }
}

fun loadFromJSON<T> (factory: JSONFactory<T>): T {...}
loadFromJSON(Person)    // JSONFactory의 인스턴스를 넘길 때 Person 클래스의 이름을 사용했다.
```

##### 코틀린 동반 객체와 정적 멤버

> 클래스의 동반 객체는 클래스에 정의된 인스턴스를 가리키는 정적 필드로 컴파일된다. 동반 객체에 이름을 붙이지 않았다면 자바 쪽에서 `Companion`이라는 이름으로 그 참조에 접근할 수 있다.

```java
// in java
Person.Companion.fromJSON("...");
```

#### 동반 객체 확장

기존 클래스에 대해 호출할 수 있는 새로운 함수를 정의하고 싶을 때 클래스에 동반 객체가 있으면 그 객체 안에 함수를 정의하여 클래스에 대해 호출할 수 있는 확장 함수를 만들 수 있다.

```kotlin
// 비즈니스 로직 모듈
class Person(val firstName: String, val lastName: String) {
    companion object {} // 비어있는 동반 객체를 선언한다.
}

// 클라이언트/서버 통신 모듈
fun Person.Companion.fromJSON(json: String): Person {...}   // 확장 함수를 선언한다.
val p = Person.fromJSON(json)
```

- 동반 객체에 대한 확장 함수를 정의하려면 원래 클래스에 동반 객체를 꼭 선언해 두어야 한다.

### 4.4.4 객체 식: 무명 내부 클래스를 다른 방식으로 작성

**무명 객체<sup>anonymous object</sup>를 정의할 때도 `object` 키워드를 쓴다.

```kotlin
window.addMouseListener(
    object: MouseAdapter() {    // MouseAdapter를 확장하는 무명 객체를 선언한다.
        override fun mouseClicked(e: MouseEvent) {...}  // MouseAdapter의 메소드를 오버라이드한다.
        override fun mouseEntered(e: MouseEvent) {...}  // MouseAdapter의 메소드를 오버라이드한다.
    }
)
```

위 예제에서 알 수 있는 것은, 무명 객체 선언 또한 인반 객체 선언과 크게 다르지 않다는 것이다.

- 유일한 차이는 객체 이름이 없다는 것이다.

> 객체 식은 클래스를 정의하고 그 클래스에 속한 인스턴스를 생성하지만, 그 클래스나 인스턴스에 이름을 붙이지는 않는다. 하지만 객체에 이름을 붙여야 한다면 변수에 무명 객체를 대입하면 된다.

```kotlin
val listener = object: MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {...}
    override fun mouseEntered(e: MouseEvent) {...}
}
```

- 객체 선언과 달리 무명 객체는 싱글턴이 아니다. 객체 식이 쓰일 때마다 새로운 인스턴스가 생성된다.
- Q. 변수에 대입하고 변수를 호출하면 같은 인스턴스가 호출되는 것인가?
- 자바의 무명 클래스와 같이 객체 식 안의 코드는 그 식이 포함된 함수의 변수에 접근할 수 있다.
- 하지만 자바와 달리 `final`이 아닌 변수도 객체 식 안에서 사용할 수 있다.
- 따라서 객체식 안에서 그 변수의 값을 변경할 수 있다.

```kotlin
fun countClicks(window: Window) {
    var clickCount = 0
    window.addMouseListener(object: MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            clickCount++
        }
    })
}
```
