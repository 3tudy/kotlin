# 부록E 코루틴과 Async/Await

## E.1 코루틴이란?

> 코루틴은 컴퓨터 프로그램 구성 요소 중 하나로 **비선점형 멀티태스킹**<sub>non-preemptive multitasking</sub>을 수행하는 일반화한 **서브루틴**<sub>subroutine</sub>이다. 코루틴은 실행을 **일시 중단**<sub>suspend</sub>하고 **재개**<sub>resume</sub>할 수 있는 여러 **진입 지점**<sub>entry point</sub>를 허용한다.

### 서브 루틴

> 여러 명령어를 모아 이름을 부여해서 반복 호출할 수 있게 정의한 프로그램 구성 요소. *쉽게 말해서, 함수*.

#### 서브루틴에 진입하기

함수(또는 메소드)를 호출하면 그때마다 **활성 레코드**<sub>activation recode</sub>라는 것이 **스택**에 할당되면서 서브루틴 내부의 로컬 변수 등이 **초기화된다.**

`return`을 통해 서브루틴이 실행을 중단하고 **제어**를 호출한 쪽<sub>caller</sub>에게 돌려주는 지점을 여러개로 설정할 수도 있다.

서브루틴에서 반환되고 나면 활성 레코드가 스택에서 사려져 **실행 중이던 모든 상태를 잃어버린다**.

### 비선점형 멀티태스킹

> 멀티태스킹의 각 작업을 수행하는 참여자들의 실행을 **운영체제가** 강제로 일시 중단시키고 **다른 참여자를 실행하게 만들 수 없는** 멀티태스킹을 말한다.

### 그래서 코루틴이란

> **서로 협력**해서 **실행**을 주고받으면서 **작동**하는 **여러 서브루틴**을 말한다.

대표적인 코루틴으로는 제네레이터<sub>generator</sub>를 예로 들 수 있다. 일반적으로 `yield` 키워드를 사용해 caller 쪽으로 제어를 넘겼다가 caller가 코루틴을 호출하면 다시 `yield` 전까지 실행 후 제어를 caller 쪽으로 돌려준다.

## E.2 코틀린의 코루틴 지원: 일반적인 코루틴

> 코틀린은 특정 코루틴을 언어가 지원하는 형태가 아니라, **코루틴을 구현할 수 있는 기본 도구를 언어가 제공하는 형태**다.

### E.2.1 여러 가지 코루틴

#### `kotlinx.coroutines.CoroutineScope.launch`

>`launch`는 코루틴을 잡<sup>Job</sup>으로 반환하며, 만들어진 코루틴은 기본적으로 **즉시 실행된다.** 원하면 `launch`가 반환한 `Job`의 `cancel()`을 호출해 **코루틴 실행을 중단**시킬 수 있다.

```kotlin
package com.enshahar.kotlinStudy

import kotlinx.coroutines.*
import java.time.ZonedDateTime
import java.time.temporal.ChronoUnit

fun now() = ZonedDateTime.now().toLocalTime().truncatedTo(ChronoUnit.MILLIS)

fun log(msg:String) = println("${now()}:${Thread.currentThread()}:${msg}")

fun launchInGlobalScope() {
    GlobalScope.launch {    // GlobalScope에서는 메인 스레드가 실행 중인 동안만 
                            // 코루틴 동작을 보장해준다.
        log("coroutine started.")
    }
}

fun main() {
    log("main() started.")
    launchInGlobalScope()   // main 함수와는 다른 thread에서 실행된다.
    log("launchInGlobalScope() executed")
    Thread.sleep(5000L)     // 이 딜레이(sleep)이 없으면 코루틴은 실행되지 않는다.
    log("main() terminated")
}
```

- GlobalScope에서는 메인 스레드가 실행 중일 때만 코루틴의 동작을 보장한다.
- 하지만 메인 스레드가 멈추지 않고 그대로 프로그램 종료까지 이어지면 코루틴은 실행되지 않는다.
- 이를 방지하기 위해선 비동기적으로 `launch`를 실행하거나, `launch`가 모두 실행될 때까지 기다려야 한다.

##### 코루틴의 실행이 끝날 때까지 현재 스레드를 블록시키는 함수 `runBlocking()`

> `runBlocking()`은 `CoroutineScope`의 확장 함수가 아닌 일반 함수이기 때문에 별도의 코루틴 스코프 객체 없이 사용 가능하다.

```kotlin
package com.enshahar.kotlinStudy

import kotlinx.coroutines.*
import java.time.ZonedDateTime
import java.time.temporal.ChronoUnit

fun now() = ZonedDateTime.now().toLocalTime().truncatedTo(ChronoUnit.MILLIS)

fun log(msg:String) = println("${now()}:${Thread.currentThread()}:${msg}")

fun runBlockingExample() {
    runBlocking {
        launch {
            log("GlobalScope.launch started.")
        }
    }
}

fun main() {
    log("main() started.")
    runBlockingExample()
    log("runBlockingExample() executed")
    Thread.sleep(5000L)
    log("main() terminated")
}
```

- 모든 실행이 메인 스레드에서 이루어진다.

#### `kotlinx.coroutines.CoroutineScope.async`

> `async`는 사실상 `launch`와 같은 일을 한다. 유일한 차이는 `launch`가 `Job`을 반환하는 반면  `async`는 `Deffered`를 반환한다는 점뿐이다. `Deffered`는 `Job`을 상속한 클래스이기 때문에 `launch` 대신 `async`를 사용할 수 있다.

`Deffered`의 타입 파라미터는 `Deffered` 코루틴이 계산을 하고 돌려주는 값의 타입이다.

##### `async`

1. 코드 블록을 비동기로 실행할 수 있다.
2. `async`가 반환하는 `Deffered`의 `await`을 사용해서 코루틴이 결과 값을 내놓을 때까지 기다렸다가 결과값을 얻어낼 수 있다.

```kotlin
fun sumAll() {
    runBlocking {
        val d1 = async { delay(1000L); 1 }
        log("after async(d1)")
        val d2 = async { delay(2000L); 2 }
        log("after async(d2)")
        val d3 = async { delay(3000L); 3 }
        log("after async(d3)")

        log("1+2+3 = ${d1.await() + d2.await() + d3.await()}")
        log("after await all & add")
    }
}
```

- d1, d2, d3가 모두 계산되고 결과 6을 얻을 때까지 3초가 걸린다.(6초가 아니다)
- 모든 함수들이 메인 스레드 안에서 실행되었다.

### E.2.2 코루틴 컨텍스트와 디스패처

> `CoroutineScope`는 `CoroutineContext` 필드를 `launch` 등의 확장 함수 내부에서 사용하기 위한 매개체 역할만을 담당한다.

`CoroutineContext`는 실제로 코루틴이 실행 중인 여러 작업(Job 타입)과 디스패처를 저장하는 일종의 맵이라 할 수 있다. **코틀린 런타임**은 이 `CoroutineContext`를 사용해서 **다음에 실행할 작업을 선정**하고, 어떻게 **스레드에 배정**할지에 대한 방법을 결정한다.

### E.2.3 코루틴 빌더와 일시 중단 함수

> 지금까지 살펴본 `launch`나 `async`, `runBlocking`은 모두 **코루틴 빌더**라고 불린다. 이들은 **코루틴을 만들어주는 함수**다.

#### 코루틴 빌더들

##### `produce`

> **정해진 채널**로 데이터를 **스트림**으로 보내는 코루틴을 만든다. 이 함수는 `ReceiveChannel<>`을 반환한다. 그 채널로부터 메시지를 전달받아 사용할 수 있다.

##### `actor`

> **정해진 채널**로 **메시지**를 받아 처리하는 **액터**를 코루틴으로 만든다. 이 함수가 반환하는 `Sendchannel<>` 채널의 `send()` 메소드를 통해 액터에게 메시지를 보낼 수 있다.

> `delay()`와 `yield()`는 **일시 중단**<sup>suspending</sup> 함수라고 불린다.

#### 일시 중단 함수들

##### `withContext`

> 다른 컨텍스트로 코루틴을 전환한다.

##### `withTimeout`

> 코루틴이 정해진 시간 안에 실행되지 않으면 예외를 발생시키게 한다.

##### `withTimeoutOrNull`

> 코루틴이 정해진 시간 안에 실행되지 않으면 `null`을 결과로 돌려준다.

##### `awaitAll`

> 모든 작업의 성공을 기다린다. 작업 중 어느 하나가 예외로 실패하면 `awaitAll`도 그 예외로 실패한다.

##### `joinAll`

> 모든 작업이 끝날 때까지 현재 작업을 일시 중단시킨다.

## E.3 `suspend` 키워드와 코틀린의 일시 중단 함수 컴파일 방법

> 일시 중단 함수를 코루틴이나 일시 중단 함수가 아닌 함수에서 호출하는 것은 컴파일러 수준에서 금지된다. 코틀린은 일시 중단 함수를 만들 수 있도록 `suspend`라는 키워드를 제공한다.

함수 정의의 `fun` 앞에 `suspend`를 넣으면 일시 중단 함수를 만들 수 있다.

```kotlin
suspend fun yieldThreeTimes() {
    ...
}
```

### `suspend` 함수는 어떻게 작동하는 것인가?

일시 중단 함수안에서 `yield()`를 해야 하는 경우 필요한 동작들:

- 코루틴에 진입할 때와 코루틴에서 나갈 때 코루틴이 **실행 중이던 상태를 저장하고 복구하는 등의 작업**을 할 수 있어야 한다. - CPS 변환과 상태 기계를 활용
- **현재 실행 중이던 위치를 저장**하고 다시 코루틴이 **재개될 때 해당 위치부터 실행을 재개**할 수 있어야 한다. - CPS 변환과 상태 기계를 활용
- 다음에 **어떤 코루틴을 실행**할지 결정한다. - 코루틴 컨텍스트의 디스패처가 수행한다.

#### CPS<sub>continuation passing style</sub> 변환

> 프로그램의 실행 중 **특정 시점 이후에 진행해야 하는 내용을 별도의 함수**<sub>Continuation</sub>로 뽑고, 그 함수에게 **현재 시점까지 실행한 결과**를 넘겨서 처리하게 만드는 **소스코드 변환기술**이다.

## E.4 코루틴 빌더 만들기
