# 코틀린 타입 시스템
* null이 될 수 있는 타입과 null을 처리하는 방법
* 코틀린 원시 타입과 자바 타입의 관계
* 코틀린 컬렉션과 자바 컬렉션과의 관계

## 널 가능성
* kotlin은 null에 대한 접근 방법은 가능한 실행 시점에서 컴파일 시점으로 옮긴다.

### null이 될 수 있는 타입
* 코틀린은 기본으로 null을 허용 안한다.
* kotlin은 nullable한 변수는 변수.메서드()가 불가능하다.
* nullable한 변수를 nullable하지 않은 변수에 대입이 불가능 하다.
* nullable한 변수를 nullable하지 않은 param으로 넘길 수 없다.
* nullable한 변수를 null과 비교 후 사용 가능하다.
```kotlin
 //null 허용 안한다.
 fun strLen(s: String) = s.length

 //null 허용 '?'
 fun strLen(s: String?) = s?.length

 //nullable한 변수는 null과 비교 후 사용
 fun strLen(s: String?): Int {
     if (s != null) s.length
     else 0
 }

```

### 타입의 의미
* 타입은 분류로 어떤 값이 가능한지와 수행 할 수 있는 연산을 결정 한다.

### 안전한 호출 연산자 '?.'
* 안전한 호출 연산자 ?는 null 검사와 method 호출을 한 번에 한다.
* 작성하기에 따라 안전한 호출로 null을 반환 한다.
* 안전한 호출은 메서드, 프로퍼티에도 사용 가능 하다.
* 안전한 호출은 연쇄도 가능하다.
```kotlin
val s: String? = null
//아래 두 로직은 같은 역활을 한다.
s?.length
if (s != null) s.length
else 0

// s가 null인 경우 null을 반환 한다.
fun printCaps(s: String?) {
    val temp: String? = s?.toUpperCase()
}

//메서드, 프로퍼티에도 사용 가능하다.
class Employee(val name: String, val mng: Employee?) {}
fun mngName(mng: Employee?): String? = mng.mng?.name

//안전한 호출 연쇄
class Address(val sc: String, val code: Int, val city: String, val country: String) {}
class Company(val name: String, val address: Address?){}
class Person(val name: String, val company: Company?){}

val person = Person("a", null)
person?.company?.address.country
```

### 엘비스 연산자 '?:'
* 코틀린은 null 대신 사용할 값을 정할 수 있다.
* 코틀린은 return, throw도 식이다. 엘비스 다음으로 올 수 있다.
```kotlin
//null이면 ""를 반환한다.
fun foo(s: String?) = s?:""

//안전한 호출과 사용
fun foo(s: String?) = s?.length?:0

fun printPerson(p: Person) {
    val address = p?.company?.address?.country?: throw IllegalArgumentException("error")
}
```

### 안전한 캐스트 as?
* as ? 연산자는 어떤 값을 지정한 타입으로 캐스트한다. 불가능한 경우는 null을 반환한다.
* 주된 패턴은 as?를 사용하고 앨비스를 붙여 null인 경우 다른 작업을 수행 한다.
```kotlin
override fun equals(o: Any?): Boolean {
    val person = o as? person?:return false
}
```

### 널 아님 단언 !!
* nullable한 타입에 null이 아님을 나타 낼때 '!!'를 사용한다.
* 널 아님 단언을 선언했으나 값이 null인 경우 NPE를 낸다.
* 널 아님 단언은 예외처리시에 stack trace를 남기지만 어디에서 발생한 지는 알 수 없다.
```kotlin
fun test(s: String?) {
    val temp: String = s!!
}
```

### let 함수
* 안전한 호출과 사용하면 원하는 식을 평가해서 결과가 null인지 검사하고 다음에 그 결과를 변수에 넣는 작업을 간단하게 한다.
* nullable한 값을 null이 아닌 값만 인자로 넘길 수 있다.
* let은 자신의 수신 객체를 인자로 받아 람다에 넘긴다.
```kotlin
fun sendEmailTo(email: String) {
    print("$email")
}

var email: String? = "xxxx"
email?.let { sendEmailTo(it) }
```

### 나중에 초기화할 프로퍼티
* 코틀린은 null이 될 수 없는 프로퍼티를 생성자 안에서 초기화 하는 방법만 있다.
* null이 가능하게 프로퍼티를 선언할 수 있지만 널 아님 단언을 선언을 항시 해야 한다.
* lateinit를 사용해 나중에 초기화도 가능하다.
  * 나중에 초기화하는 프로퍼티는 항상 var 여야 한다.
  * lateinit를 사용하면 nullable 하지 않아도 생성자가 아닌 다른 곳에서 초기화 가능하다.
  * lateinit가 선언되고 초기화가 안된 프로퍼티에 접근하면 NPE대신 initialized가 발생한다.
```kotlin
class MyService {
    fun action(): String = "foo"
}

class MyTest {
    
    private lateinit var my: MyService
    
    @Before
    fun setup() {
        my = Myservice()
    }
}
```


### 널이 될 수 있는 타입 확장
* 메소드를 호출하기 전에 수신 객체 역할을 하는 변수가 null이 될 수 없다고 보장하는 대신 직접 변수에 대해 메소드를 호출해도 확장 함수인 메소드가 알아서 널처리를 한다.
* 예시 함수는 isNullOrEmpty, isNullOrBlank
* null이 될 수 있는 타입에 대한 확장을 정의하면 null이 될 수 있는 값에 확장함수 호출 가능 단 수신객체의 this는 null이 될 수 있다.
```kotlin
var str: String? = null
str.isNullOrBlank()
```


### 타입 파라미터의 널 가능성
* 코틀린에서는 메서드, 클래스에 타입파라미터는 null이 될 수 있다.
* 타입파라미터는 Any?와 같다. null을 방지하려면 상한을 지정해야 한다.
* 코틀린에서 "?"를 사용하지 않아도 null이 될 수 있는 유일한 경우다.
```kotlin
fun <T> printHashCode(t: T) {
    print(t?.hashCode())
}

fun <T: Any> printHashCode(t: T) {
    
}
```

### 널 가능성과 자바
* 자바는 널 가능성을 지원하지 않는다.
* 다만 어노테이션으로 표기 가능
* @nullable = String?
* @notnull = String

### 플랫폼 타입
* 코틀린에서 널 관련 정보를 알 수 없는 타입을 말한다. 자바와 호환하는 경우 발생

### 상속
* 자바 interface는 코틀린에서 상속시 널 가능, 널 불가능 둘다 구현가능 하다.

## 코틀린의 원시 타입

### 원시 타입: Int, Boolean
* 코틀린은 원시타입, 래퍼타입 구분이 없다.
* 코틀린은 숫자 타입 등 원시 타입의 값에 대해 method 호출 가능하다.
* 항상 객체가아닌 실행 시점에 숫자 타입은 가장 효율적인 방식으로 표현한다.
```kotlin
val percent = num.coerceIn(0, 100)
```

### 숫자 변환
* 코틀린과 자바의 큰 차이는 숫자 변환이다.
* 코틀린은 모든 워니타입에 변환 함수를 제공한다. 단 boolean 제외
* 표현 범위가 큰수에서 작은 수로 가는 경우 값이 잘리기도 한다.
* 리터럴을 사용하면 함수 호출을 사용할 필요가 없다.
```kotlin
val i = 1
val l: Long = i //불가능
val l: Long = i.toLong()
```

### Any, Any? 최상위 타입
* 자바에서 object가 코틀린에서는 any다
* any는 null이 될 수 없다.
* null허용하려면 ?를 사용해야 한다.
* any에는 toString, equals, hashcode가 있지만 waith, notify등은 object로 캐스팅 해야 한다.

### Unit 타입 - 코틀린에서 void
* 자바에서 void와 같다.
* Unit는 void와 달리 인자로 사용된다.
```kotlin
fun f(): Unit{}
fun f() {}
//두 메서드는 동일 하다.
```

### Nothing 타입 - 이 함수는 결코 정상적으로 끝나지 않느다.
* 코틀린에는 결코 성공적으로 값을 돌려주는 일이 없으므로 변환 값이라는 개념 자체가 의미 없는 함수가 일부 존재한다.
* 예시로 테스트 라이브러리들은 fail이라는 함수, 무한 루프를 도는 함수도 결코 값을 반환 못한다.
* 이런 사실을 알면 유용하다.
* Nothing 타입은 아무 값도 포함하지 않는다. 따라서 Nothing은 함수의 반환 타입이나 반환 타입으로 쓰일 타입 파라미터로만 쓸 수 있다.
* 그 외는 의미 없다.
```kotlin
fun fail(msg: String): Nothing {
    throw IllegalArgumentException(msg)
}
```

## 컬렉션과 배열
### 널 가능성과 컬렉션
* 컬렉션에 null 가능성은 2가지로 구분한다.
* 코틀린은 filterNotNull을 통해 List<Int?>를 List<Int>로 변환 가능하다.
```kotlin
//컬렉션 내부 요소가 null 일 수 있다.
val list = ArrayList<Int?>()
//컬렉션이 null일 수 있다.
val collection = ArrayList<Int>()
```

### 읽기 전용과 변경 가능한 컬렉션
* 코틀린은 데이터에 접근하는 인터페이스와 안의 데이터를 변경하는 인터페이스를 분리 했다.
* Collection, MutableCollection
* Collection은 읽기 전용, MutableCollection은 변경가능
* MutableCollection은 Collection을 확장한다.
* 코드에서는 가능한 읽기전용 사용을 권장한다.
* 읽기전용과 변경가능이 동일한 객체를 참조하는 경우는 알수 없는 예외가 발생하니 주의 해야 한다.

### 코틀린 컬렉션과 자바
|컬렉션 타입|읽기 전용 타입|변경 가능 타입|
|------|---|---|
|list|listOf|mutableListOf, arrayListOf|
|set|setOf|mutableSetOf, hashSetOf, linkedSetOf, sortedSetOf|
|map|mapOf|mutableMapOf, hashMapOf, linkedMapOf, soredMapOf|

### 객체의 배열과 원시 타입의 배열
* arrayOf 함수
* arrayOfNulls 함수: 모든 요소가 널로 채워짐
* Array 생성자는 배열 크기와 람다를 인자로 받아서 생성
* toTypedArray 메서드를 사용하여 컬렉션을 배열로 쉽게 변환 가능하다.
* 래퍼타입 즉 박싱된 값이 들어있는 컬렉션이나 배열의 경우 toInArray 등의 변환 함수를 사용해 박싱하지 않은 값이 들어 있는 배열로 변환 할 수 있다.
* vararg로 넘기기 위해서는 배열로 만들고 스프레드 연산자를 사용한다.
```kotlin
val letters = Array<string>(26) {i -> ('a'+i).toString() }
val list: List<String> = listOf("h","e","l","l","o")
val array: Array<String> = strings.toTypeArray()

fun temp(vararg values: String) {
    
}
temp(* list.toTypedArray())
```