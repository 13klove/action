# 고차 함수: 파라미터와 반환 값으로 람다 사용
* 함수 타입
* 고차 함수와 코드를 구조화할 떄 고차 함수를 사용하는 방법
* 인라인 함수
* 비로컬 return과 레이블
* 무명 함수
----
* 람다는 추상화를 하기 좋은 도구며, 클래스나 컬렉션에만 람다의 능력을 적용할 수 있는 것은 아니다.
* 람다를 인자로 받거나 반환하는 고차함수 다루는 방법을 다룬다.
* 람다를 사용함에 따라 발생할 수 있는 성능상 부가 비용을 없애고 람다 안에서 더 유연하게 흐름을 제어할 수 있는 인라인 함수를 설명한다.


## 고차 함수 정의
* 고차 함수는 다른 함수를 인자로 받거나 함수를 반환하는 함수다.
* 람다나 함수 참조를 사용해 함수를 값으로 표현할 수 있다.
* 고차 함수는 람다나 함수 참조를 인자로 넘길 수 있거나 람다나 함수 참조를 반환하는 함수다.
* 또한 함수를 인자로 받는 동시에 함수를 반환하는 함수도 고차 함수다.
---
### 함수 타입
* 람다 인자의 타입을 선언하기 전에 단순하게 함다를 로컬 변수에 대입하는 경우를 확인 한다.
```kotlin
//람다를 변수에 대입하는 간단한 식이다. 
val sum = { x, y -> x+y }
val action = { println("hi~") }

//람다를 변수에 대입하는 구체적인 식이다.
//함수의 타입을 정의하려면 아래와 같이 괄호로 인자의 타입을 선언하고 '->' 뒤에 반환 값을 정의 한다.
//Unit은 의미 없지만 함수의 타입을 명시할때는 필수로 작성 해야 한다.
val sum: (x: Int, y: Int) -> Int = {x, y -> x+y}
val cation: () -> Unit = {println("hi~")}

//함수의 반환이 null인 경우와 함수의 타입이 null이 될 수 있음을 표기한 식이다.
val canRetrunNull: (x: Int, y: Int) -> Int? = {x, y -> x+y}
val funOrNull: (() -> Unit)? = null

//함수에 적용되면 아래와 같다.
fun performRequest(
    url: String,
    callBack: (code: Int, content: String) -> Unit
) {
    
}

performRequest("http://localhost:8080") {code, content -> /*...*/}
```

### 인자로 받은 함수 호출
* 간단한 고차 함수를 코드로 본다.
```kotlin
//인자로 받은 함수는 기존 사용법과 동일하게 사용한다.
fun twoAndThree(operation: (Int, Int) -> Int ){
    return operation(2, 3)
}

twoAndThree {x, y -> x+y}
```
* filter를 활용한 예시를 본다.
* fun String.filter(predicate: (Char) -> Boolean): String
* String은 수신객체이고, predicate는 파라미터 이름이다. (Char) -> Boolean은 파람으로 받는 함수 타입니다.
```kotlin
fun String.filter(predicate: (Char) -> Boolean ): String {
    val sb = StringBuilder()
    for (idx in 0 until length) {
        if ( predicate(get(idx)) ) sb.append(get(idx))
    }
    return sb.toString()
}

"ab123da".filter { it in 'a'..'z' }
//ab123da 문자열에서 a ~ z가 포함된 문자열만 반환 된다.
```

### 자바에서 코틀린 함수 타입 사용
* 해당 부분은 skip한다.

### 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터
* 코드로 설명한다.
```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)
    for( (idx, element) in this.withIndex() ) {
        if ( idx > 0 ) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}

//위코드에서 result의 문자열을 대문자 혹은 소문자로 변환하는 함수 타입을 추가한다.

//transform을 통함 함수 타입을 추가 했다.
//함수 타입은 다른 파람과 같이 default로 정의 할 수 있다.
//함수 타입에 T는 컬렉션의 확장함수의 영향이다.
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: (T) -> String = {it.toString()},
): String  {
    val result = StringBuilder(prefix)
    for( (idx, element) in this.withIndex() ) {
        if ( idx > 0 ) result.append(separator)
        result.append(transform(element))
    }
    result.append(postfix)
    return result.toString()
}

//함수 타입이 nullable하게 하는 경우 안전하게 호출 하는 방법을 볼 설명 한다.
//함수 타입은 invoke를 호출하는 인터페이스라는 근거를 기반으로 개발한다.
//transform 안전한 호출 연산자와 invoke, 엘비스 연산자를 통해 호출 한다.
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: ((T) -> String)? = null
): String  {
    val result = StringBuilder(prefix)
    for( (idx, element) in this.withIndex() ) {
        if ( idx > 0 ) result.append(separator)
        result.append(
            transform?.invoke(element)?:element.toString()
        )
    }
    result.append(postfix)
    return result.toString()
}
```

### 함수를 함수에서 반환
* 함수가 함수를 반환할 필요는 적지만 알아두는게 좋다.
```kotlin
enum class Delivery { STANDARD, EXPEDITED }
class Order(val itemCount: Int)

fun getShippingCostCalculator(delivery: Delivery): (Order) -> Double {
    if (delivery = Delivery.EXPEDITED) return {order -> order.itemCount*2.3}
    return {order -> order.itemCount*1.0}
}

val costCalc = getShippingCostCalculator(Delivery.EXPEDITED)
costCalc(Order(3))
```

### 람다를 활용한 중복 제거
* 코드를 통해서 내용을 파악 한다.
* 예시는 웹사이트 방문 기록을 분석하는 예로 본다.
```kotlin

enum class OS{WINDOWS, LINUX, MAC, IOS, ANDROID}
data class SiteVisit (
    val path: String,
    val duration: Double,
    val os: OS
        )

val log = listOf(
    SiteVisit("/", 34.0, OS.WINDOWS),
    SiteVisit("/", 22.0, OS.MAC),
    SiteVisit("/login", 12.0, OS.WINDOWS),
    SiteVisit("/signup", 8.0, OS.IOS),
    SiteVisit("/", 16.3, OS.ANDROID)
)

// window 사용자의 평균 사용시간을 구한다.
val windowsAvgDuration = log.filter { it.os == OS.WINDOWS }.map(SiteVisit::duration).average()

// 리팩토링 OS가 WINDOWS 뿐만 아니라 조건에 맞게 duration을 구하고 싶다.
//list에 제네릭은 siteVisit로 확장함수를 만들었다.
fun List<SiteVisit>.osAvgDuration(os: OS) = this.filter { it.os == os }.map(SiteVisit::duration).average()

//만약 안드로드, ios 둘의 평군을 구하고 싶다면 또 다른 코드를 작성해야 한다.
//고차 함수를 통해 이를 해결해 본다.
fun List<SiteVisit>.avgDurationFor(predicate: (SiteVisit) -> Boolean) = this.filter(predicate).map(SiteVisit::duration).average()
log.avgDurationFor { it.os in listOf(OS.ANDROID, OS.IOS) }

```

## 인라인 함수: 람다의 부가 비용 없애기
* 코틀린은 보통 람다를 무명 클래스로 컴파일하지만 그렇다고 람다 식을 사용할 떄마다 새로운 클래스가 만들어지지는 않는다.
* 람다가 변수를 포획하면 람다가 생성되는 시점마다 새로운 무명 클래스 객체가 생성된다.
* 이런 경우 무명 클래스 생성에 따른 부가 비용이 든다. 즉 덜 효율적이다.
* 반복되는 코드를 별도의 라이브러리 함수로 빼내되 컴파일러가 자바의 일반 명령문만큼 효율적인 코드를 생성하게 코틀린에서는 inline 변경자를 통해 제공한다.
* inline 변경자를 어떤 함수에 붙이면 컴파일러는 그 함수를 호출하는 모든 문장을 함수 본문에 해당하는 바이트코드로 변경한다.
* inline은 해당 함수를 호출함이 아닌 함수의 코드가 호출하는 위치에 적용됨을 의미한다.
---
### 인라이닝이 작동하는 방식
* 어떤 함수를 inline으로 성ㄴ언하면 그 함수의 본문이 인라인된다.
```kotlin
inline fun <T> synchronized(lock: Lock, action: () -> T): T {
    lock.lock()
    try {
        return action()
    } finally {
        lock.unlock()
    }
}

val lock = Lock()
synchronized(lock) {/*..*/}

//synchronized는 inline 함수로 호출했기에 에 아래와 같아진다.
fun foo(lock: Lock) {
    println("before sync")
    lock.lock()
    try {
        return action()
    } finally {
        lock.unlock()
    }
    println("after sync")
}
//위 코드는 예시일 분이다. 이런식으로 바이트 코드로 적용된다.
//synchronized 함수는 본문뿐만 아니라 synchronized에 전달된 람다의 본문도 함께 인라이닝된다.

//인라인 함수를 호출하면서 람다를 넘기는 대신에 함수 타입의 변수를 넘길 수도 있다.
class LockOwner(val lock: Lock) {
    fun runUnderLock(body: () -> Unit) {
        synchronized(lock, body)
    }
}
//이런경우는 synchronized 함수의 본문만 인라이닝 된다.
//body를 알수 없기에 인라인 안된다.
```

### 인라인 함수의 한계
* 인라이닝을 하는 방식으로 인해 람다를 사용하는 모든 함수를 인라이닝할 수는 없다.
* 함수 본문에서 파라미터로 받은 람다를 다른 변수에 저장하고 나중에 그 변수를 사용한다면 람다를 표현하는 객체가 어딘가는 존재해야 하기 떄문에 람다를 인라이닝할 수 없다.
* 일반적으로 인라인 함수의 본문에서 람다 식을 바로 호출하거나 람다 식을 인자로 전달받아 바로 호출하는 경우에는 그 람다를 인라이닝할 수 있다.
* 그런 경우가 아니면 Illegal usage of inline-parameter와 함꼐 금지시킨다.
* 둘 이상의 람다를 인자로 받는 함수에서 일부 람다만 인라이닝하고 싶을 때도 있다.
* 이런경우 noinline을 통해 인라이닝을 금지할 수 있다.


### 컬렉션 연산 인라이닝
* 컬렉션에서 filter, map과 같은 함수는 인라인 함수다.
* 즉 인라이닝되며 추가 객체나 클래스 생성은 없다.
* 하지만 중간 연산에 컬렉션 생성으로 성능을 고민할 수 있다.
* asSequence를 통해 사용하면 중간연산으로 컬렉션은 생성 안하지만 filter, map과 같은 함수를 필드에 저장하기에 인라이닝 효과를 못본다.

### 함수를 인라인으로 선언해야 하는 경우
* 일반 함수 호출의 경우 jvm은 이미 강력하게 인라이닝을 지원한다.
* jvm은 코드 실행을 분석해서 가장 이익이 되는 방향으로 호출한다.
* 코틀린 인라인 함수는 바이트코드에서 각 함수 호출 지점을 함수 본문으로 대치하기 때문에 코드 중복이 생긴다.
* 람다를 인자로 받는 함수를 인라이닝하면 생기는 이익
  * 인라이닝으로 부가 비용이 상당히 감소한다.
  * 람다를 표현하는 클래스와 람다 인스턴스에 해당하는 객체를 만들 필요도 없다.
  * 인라이닝으로 람다에서 넌로컬 반환 기능을 제공한다.
  * inline를 사용시에 코드 크기에 주의해야한다.

### 자원 관리를 위해 인라인된 람다 사용
* 람다로 중복을 없앨 수 있는 일반적인 패턴 중 한 가지는 언떤 작업을 하기 전에 자원을 획등하고 작업을 마친 후 자원을 해제하는 자원 관리다.
* 파일, 락, 트랜잭션 등 여러 대상이다.
* 람다 본분ㅇ서 사용된 return은 넌로컬 return이다. readFirstLineFromFile 함수를 끝내면서 값을 반환한다.
```kotlin
val lock: Lock...
lock.withLock {
    //
}

//closeable 자원에 대한 확장 함수
fun readFirstLineFormFile(path: String): String {
    BufferedReader(FileReader(path)).use{ br ->
        return br.readLine()
    }
}
```


## 고차 함수 안에서 흐름 제어
### 람다 안의 return문: 람다를 둘러싼 함수로부터 반환
* 코드로 내용 파악한다.
* 람다식의 return이 아닌 본인을 호출한 상위의 return이 넌로컬 return이다.
* 이런경우는 inline함수만 적용된다.
```kotlin
data class Person(val name: String, val age: Int)

val people = listOf(Person("alice", 29), Person("bob", 31))

//여기서 사용되는 return은 lookForAlice의 반환이다 즉 넌로컬 return이다.
fun lookForAlice(people: List<Person>) {
    for (person in people) {
        if(person.name == "alice") {
            println("found")
            return
        }
    }
    println("where alice")
}

//forEach는 inline 함수기에 넌로컬 return이 발동한다.
fun lookForAlice01(people: List<Person>) {
    people.forEach {
        if(person.name == "alice") {
            println("found")
            return
        }
    }
    println("where alice")
}
```

### 람다로부터 반환: 레이블을 사용한 return
* 람다 식도 로컬 return을 사용할 수 있다.
* 로컬 return은 loop문의 break와 비슷한 역할을 한다.
  * 람다식을 종료하고 람다를 호출한 곳으로 이동해 로직을 계속 진행한다.
* 로컬 return, 넌 로컬 return을 구분하기 위해 레이블을 사용한다.
* 로컬 return을 하기 위해서는 람다에 레이블을 지정하고 return 후에 레이블을 입력한다.
* 람다 식의 레이블을 명시하면 함수 이름을 레이블로 사용할 수 없고 2개 이상 레이블을 붙일 수 없다.
```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach label@ {
        if (it.name == "alice") return@label
    }
  println("alice might be somewhere")
}

//혹은 인라인 함수의 이름을 사용해도 된다.
fun lookForAlice(people: List<Person>) {
    people.forEach { 
        if (it.name == "alice") return@forEach
    }
}
```

### 무명 함수: 기본적으로 로컬 return
* 무명 함수는 코드 블록을 함수에 넘길 때 사용할 수 있는 다른 방법이다.
* 무명 함수는 일반 함수와 비슷해 보이지만 함수 이름이나 파라미터 타입을 생략할 수 있다.
* 무명 함수는 기본적으로 로컬 return을 한다.
```kotlin
people.filter(fun (person) = person.age < 30)
```