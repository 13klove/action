# 제네릭스
* 제네릭 함수와 클래스를 정의하는 방법
* 타입 소거와 실체화한 타입 파라미터
* 선언 지점과 사용 지점 변성
---
* 실체화한 타입 파라미터를 사용하면 인라인 함수 호출에서 타입 인자로 쓰인 구체적인 타입을 실행 시점에 알 수 있다.
* 선언 지점 변성을 사용하면 기저 타입은 같지만 타입 인자가 다른 두 제네릭 타입 Type\<A>, Type\<B>가 있을 때 타입 인자 A와 B의 상위/하위 타입 관계에 따라 두 제네릭 타입의 상위/하위 타입 관계가 어떻게 되는지 지정할 수 있다.
* 사용 지점 변성은 같은 목표를 제네릭 타입 값을 사용하는 위치에서 파라미터 타입에 대한 제약을 표시하는 방식으로 달성한다.
* 자바 와일드카드는 사용 지점 변성에 속하며, 코틀린 선언 지점 변성과 같은 역할을 한다.
---
## 제네릭 타입 파라미터
* 제네릭스를 사용하면 타입 파라미터를 받는 타입을 정의할 수 있다.
* 제네릭 타입의 인스턴스를 만들려면 구체적인 타입 인자로 치환해야 한다.
  * List -> List\<String> 과 같이 구체적인 타입 인자로 치환해야 한다.
* 클래스에 타입 파라미터가 여럿 있을 수도 있다.
  * Map<k, v> -> Map<String, String>과 같이 여럿 있을 수 있다.
* 코틀린은 컴파일러가 타입 추론이 가능하다./반면 빈 리스트를 만들때는 타입 추론이 불가능해 구체적인 타입 인자를 넣어야 한다.
  * val list = listOf("a", "b") 와 같이 구체적인 타입 인자를 선언 안해도 추론한다.
  * val list = listOf\<String>()

### 제네릭 함수와 프로퍼티
* 컬렉션을 다루는 라이브러리 함수는 대부분 제네릭 함수다.
  * 특정 타입만 컬렉션으로 다루지 않고 구체화된 타입을 모두 컬렉션으로 사용할 수 있게 하기 위함이다.
  * fun \<t>(타입 파라미터) List\<t>.slice(indices: IntRange): List\<t>
  * 타입 파라미터가 수신객체와 반환 타입에 쓰인다.
```kotlin
val letters = ('a'..'z').toList()
letters.slice<Char>(0..2)
letters.slice(0..2)
//컴파일러가 타입 추론하기에 위와 같은 경우는 구체적인 타입을 명시하지 않아도 된다.
```
* 클래스나 인터페이스 안에 정의된 메소드, 확장 함수, 최상위 함수에서 타입 파라미터를 선언할 수 있다.
* 확장 프로퍼티에도 제네릭을 선언 할 수 있다.(프로퍼티는 불가능)
```kotlin
val <T> List<T>.penultimate: T
    get() = this[size-2]

listOf(1, 2, 3, 4).penultimate
//3
```

### 제네릭 클래스 선언
* 자바와 마찬가지로 코틀린도 타입 파라미터를 넣은 꺾쇠 기호를 클래스 or 인터페이스 이름 뒤에 붙이면  제네릭하게 만들 수 있다.
* 타입 파라미터를 이름 뒤에 붙이고 나면 클래스 본문 안에서 타입 파라미터를 다른 일반 타입처럼 사용할 수 있다.
```kotlin
interface List<T> { //T라는 타입 파라미터를 정의한다.
    operator fun get(index: Int): T //인터페이스 안에서 T를 일반 타입처럼 사용할 수 있다.
}
```
* 제네릭 클래스를 확장하는 클래스 혹은 인터페이스를 정의하려면 기반 타입의 제네릭 파라미터에 대해 타입 인자를 지정해야 한다.
  * 구체적인 타입을 넘길 수도 있고 타입 파라미터를 그대로 넘길 수 있다.
```kotlin
class StringList: List<String> { //구체적인 타입을 넘긴다.
    
}
class ArrayList<T>: List<T> { //타입 파라미터를 넘긴다.
    
}
```
* 클래스가 자기 자신을 타입 인자로 참조할 수도 있다.
```kotlin
interface Comparable<T> {
    fun compareTo (other: T): Int
}

class String: Comparable<String> {
    
}
```

### 타입 파리미터 제약
* 타입 파라미터 제약은 클래스나 함수에 사용할 수 있는 타입 인자를 제한하는 기능이다.
* 타입 파라미터에 대한 상한으로 지정하면 그 제네릭 타입을 인스턴스화할 떄 사용하는 타입 인자는 반드시 그상한 타입이거나 상한 타입의 하위 타입이어야 한다.
```kotlin
//: Number로 상한을 걸었다.
fun <T: Number> List<T>.sum(): T
```
* 상한이 걸리면 T 타입의 값을 그 상한 타입의 값으로 취급할 수 있다.
```kotlin
fun <T: Number> oneHalf(value: T): Double {
    return value.toDouble()
}
```
* 타입 파라미터를 제약하는 함수 선언하기
```kotlin
//상한을 Comparable을 주었기에 Comparable을 구현한 타입만 사용이 가능하다.
fun <T: Comparable<T>> max(first:T, second: T): T {
    return if (first > second) first else second
}
```
* 타입 파라미터에 여러 제약 가하기
```kotlin
fun <T> ensureTrailingPeriod(seq: T)
    where T: CharSequence, T: Appendable {
        if(!seq.endsWith('.')) {
            seq.append('.')
        }
    }
```

### 타입 파라미터를 널이 될 수 없는 타입으로 한정
* 제네릭 클래스나 함수를 정의하고 그 타입을 인스턴스화할 때는 널이 될 수 있는 타입을 포함하는 어떤 타입으로 다입 인자를 지정해도 타입 파라미터를 치환할 수 있다.
```kotlin
// 제네릭에는 널 허용이 표기 되어 있지 않지만 널이 될 수 있는 타입을 넘길 수도 있다.
class Processor<T> {
    fun process(value: T) {
        value?.hashCode()
    }
}

val nullableClass = Processor<String?>
nullableClass.process(null)
```
* 항상 널 불가능으로 하려면 상한을 Any 혹은 다른 타입으로 할 수 있다.
```kotlin
class Processor2<T: Any> {
    fun process(value: T) {
        value.hashCode()
    }
}
//아래는 에러가 나는 코드다.
val nullableClass2 = Processor2<String?>
```

## 실행 시 제네릭스의 동작: 소거된 타입 파라미터와 실체화된 타입 파라미터
* jvm의 제네릭스는 보통 타입 소를 사용해 구현된다.
* 인스턴스화가 되면 타입 정보는 들어가지 않는다.
* 함수를 inline으로 선언함으로써 제약을 어떻게 우회할 수 있는지 살펴본다.
* 함수를 inline으로 만들면 타입 인자가 지워지지 않게 할 수 있다.(실체화라고 kotlin에서는 한다.)
---
### 실행 시점의 제네릭: 타입 검사와 캐스트
* 코틀린도 제네릭은 인스턴스화가 되면서 타입 정보는 소각 된다.
```kotlin
//아래 2개 list는 실행 시점에는 서로 동일한 타입니다.(list로)
val list1: List<String> = listOf("a")
val list2: List<Int> = listOf(1)
```
* 하지만 우리가 컴파일러가 인지할 수 있게 구체적인 타입을 정했기에 컬렉션에서는 구체적인 타입만 원소로 받을 수 있다.
* 타입 소거로 인해 생기는 한계도 있다.
  * 타입 인자가 따로 저장 안되어 실행 시점에 타입 인자를 검사할 수 없다.
```kotlin
// 실행시점에 해당 list인지는 알 수 있으나 string 혹은 어떤 객체를 가지고 있는지 알 수 없다.
fun <T>check(value: List<T>) {
    val a = value as List<Int>
    if(value is List<String>) { // 컴파일러는 여기서 cannot check for instance of erased type 에러를 낸다.
        println(value)
    }
}
```
  * 어떤 값이 집합이나 맵이 아니라 리스트라는 사실을 알게 하려면 스타 프로젝션을 사용한다.
```kotlin
if (value is List<*>) {}
```
  * as나 as? 캐스팅에도 여전히 제네릭 타입을 사용할 수 있다. 하지만 기저 클래스는 같지만 타입 인자가 다른 타입으로 캐스팅해도 성공한다.
    * 위의 내용은 중요하다.
```kotlin
fun printSum(c: Collection<*>) {
  val intList = c as? List<Int> ?: throw IllegalArgumentException("error")
  println(intList.sum())
}

printSum(listOf(1, 2, 3))
printSum(listOf("a", "b"))
//위의 경우 컴파일러는 허용하지만 실행하면 에러난다.
```
* 알려진 타입 인자를 사용한다면 타입 검사가 가능하다.
```kotlin
fun printSum(c: Collection<Int>) {
    if (c is List<Int>) {
      println(c.sum())
    }
}
```

### 실체화한 타입 파라미터를 사용한 함수 선언
* 인라인 함수의 타입 파라미터는 실체화되므로 실행 시점에 인라인 함수의 타입 인자를 알 수 있다.
* 인라인 함수 복습
  * inline 키워드를 붙이면 컴파일러는 그 함수를 호출한 식을 모두 함수 본문으로 바꾼다.
  * 함수가 람다를 인자로 사용하는 경우 그 함수를 인라인 함수로 만들면 람다 코드도 함꼐 인라이닝되어 무명 클래스, 객체 생성 안한다.
```kotlin
inline fun <reified T> isA(value: Any) = value is T //이 코드는 컴파일 된다.
isA<String>("abc")
```
  * inline함수를 사용할때는 항상 함수의 크기를 생각 해야 한다.

### 실체화한 타입 파라미터로 클래스 참조 대신
### 실체화한 타입 파리머터의 제약
* 지금 학습하기 어려움이 있어 조금 미룹니다.
---
## 변성: 제네릭과 하위 타입
* 변성은 List\<String>, List\<Any> 와 같이 기저 타입은 같지만 타입 인자가 다른 여러 타입이 서로 어떤 관계가 있는지 설명 한다.
---
### 변성이 있는 이유: 인자를 함수에 넘기기
* List\<Any> 타입의 파라미터를 받는 함수에 List\<String>을 넘기면 안전하다. 단 변경 불가능인 List만 가능하고 mutable은 안전하지 않다.

### 클래스, 타입, 하위 타입
* 제네릭 클래스는 올바른 타입을 얻으려면 구체적인 타입을 명시 해야 한다.
* 단순 List는 타입이 아니고 List\<String>, List\<Int> 이 타입이고 제네릭 클래스는 무수히 많은 타입을 만들 수 있따.
* 하위 타입은 어떤 A의 값이 필요한 모든 장소에 어떤 타입 B의 값을 넣어도 아무 문제가 없다면 타입 B는 A의 하위 타입이다.
* 상위 타입은 하위 타입과 반대로 B는 A의 상위 타입니다.
* 컴파일러는 변수 대입이나 함수 인자 전달시 매번 하위 타입 검사를 한다.
* 널 허용으로도 하위 타입은 구분된다.
  * Int는 Int?의 하위 타입니다.
  * Int?는 Int의 하위 타입이 될 수 없다.
* 제네릭에서 하위 클래스와 하위 타입의 차이는 중요하다.
* 제네릭 타입을 인스터스화할 때 타입 인자로 서로 다른 타입이 들어가면 인스턴스 타입 사이의 하위 관계가 성립 하지 않는다. 이를 무공변이라 한다.
* 코틀린의 List interface는 읽기 전용이다. a가 b의 하위 타입이면 List\<a>는 List\<b>의 하위 타입이다. 이를 공변적이라 한다.

### 공변성: 하위 타입 관계를 유지
* a가 b의 하위 타입일때 producer\<a>가 producer\<b>의 하위 타입이면 producer는 공변적이다.
* 제네릭 클래스가 타입 파라미터에 대해 공변적임을 명시하려면 타입파라미터 앞에 out을 넣어야 한다.
```kotlin
interface Producer<out T> {
    
}
```
* 모든 클래스를 공변적으로 할 수 없다.
  * 공변적으로 만드는 경우 안전하지 않을 때도 있다.
  * 공변적으로 하면 파라미터 사용을 제안 한다.
```kotlin
open class Animal {
    fun feed() {}
}

class Herd<out T: Animal> { //out 공변적임을 명시하지 않으면 feedAll에서 error 발생한다.
    val size: Int get() = 5
    operator fun get(i: Int): T {return Animal() as T}
}

fun feedAll(animals: Herd<Animal>) {
    for( i in 0 until animals.size) {
        animals.get(i).feed()
    }
}

class Cat: Animal() {
    fun cleanLitter() {}
}

fun takeCareOfCats(cats: Herd<Cat>) {
    for( i in 0 until cats.size) {
        cats[i].cleanLitter()
        feedAll(cats)//오류발생
    }
}
```
* 타입 안전성을 보장하기 위해 공변적 파라미터는 항상 out에 위치 해야 한다.
  * 생산만 가능하고 소비는 불가능 하다.
* 타입 파라미터 T가 함수의 반환 타입인 경우 out 은 producer 함수의 파라미터인 경우 in consumer이다.
```kotlin
interface Transformer<T> {
    //fun transform(t(in): T): T(out)
}
```
* out은 공변성, 사용 제한을 의미한다.
* 생성자 파람은 in, out 어디에도 포함안되어 사용가능 하다.
  * 변성은 코드에서 위험할 여지가 있는 메소드를 호출할 수 없게 만듦으로써 제네릭 타입의 인스턴스 역할을 하는 클래스 인스턴스를 잘못 사용하는 일이 없게 방지한다.
  * 생성자는 나중에 호출할 수 잆는 메소드가 아니라 위험이 없다.
  * 하지만 val or var인경우 생성자 파라미터에 적는다면 게터나 세터를 정의하는 것과 같다.
  * 읽기 전용 프로퍼티는 아웃 위치, 변경 가능 프로퍼티는 아웃과 인 위치 모두에 해당한다.
```kotlin
class Herd<T: Animal> (var leadAnimal: T, vararg animals: T){}
```
* leadAnimal 프로퍼티가 인 위치에 있기 떄문에 T를 out으로 표시할 수 없다.
* 위치 규칙은 외부에서 볼 수 있는 클래스 api에만 적용할 수 있다.

### 반공변성: 뒤집힌 하위 타입 관리
* 반공변 클래스의 하위 타입 관계는 공변 클래스와는 반대다.
* comparator는 주어진 두 객체를 비교한다.
```kotlin
interface Comparator<in T> {
    fun compare(e1: T, e2: T): Int
}

val anyComparator = Comparator<Any> {
    e1, e2 -> e1.hashCode() - e2.hashCode()
}

val strings: List<String>
strings.sortedWith(anyComparator)
//any를 통해 문자열을 비교할 수 있다.
```
* 어떤 타입을 비교한다면 조상 객체 타입을 사용할 수 있다.
* 즉 Any는 String의 상위 타입이지만 Comparator\<Any>, Comparator\<String>은 정반대 이다.
* 반공병성은 Consumer\<T>를 예로 든다.
  * 타입 B가 타입 A의 하위 타입인 경우
  * Consumer\<A>가 Consumer\<B>의 하위 타입인 관계가 성립하면 제네릭 클래스 Consumer\<T>는 타입 인자 T에 대해 반공변이다.
  * animal <- cat (상위 타입) // producer\<animal> <- producer\<cat> (공변성) // consumer<animal> -> consumer<cat> (반공변성)
* in은 공변성과 마찬가지로 타입 파라미터의 사용을 제한하고 특정 하위 타입 관계에 도달할 수 있다.

|공변성|반공변성|무공변성|
|-----|-------|---------|
|Producer\<out T>|Consumer\<in T>|MutableList<T>|
|타입 인자의 하위 타입 관계가 제네릭 타입에서도 유지된다.|타입 인자의 하위 타입 관계가 제네릭 타입에서 뒤집힌다.|하위 타입 관계가 성립하지 않는다.|
|T는 아웃 위치에서만 사용할 수 있다.|T를 인 위치에서만 사용할 수 있다.|T를 아무 위치에서나 사용할 수 있다.|
* Function 인터페이스는 공변, 반공변 둘다 가지고 있는 예시다.
```kotlin
interface Function<in P, out R> {
}
```

### 사용 지점 변셩: 타입이 언급되는 지점에서 변성 지정
* 클래스를 선언하면서 변성을 지정하면 그 클래스를 사용하는 모든 장소에 변성 지정자가 영향을 끼치므로 편리하다.
  * 이런 방식을 선언 지점 변성이라 부른다.
* 자바에서는 타입 파라미터가 있는 타입을 사용할 때마다 해당 타입 파라미터를 하위 타입이나 상위 타입 중 어떤 타입으로 대치할 수 있는지 명시해야한다.
  * 이런 방식을 사용 지점 변셩이라 부른다.
* 코틀린도 사용 지점 변성을 지원한다.
```kotlin
//아래 코드는 컬렉션의 원소를 다른 컬렉션으로 복사한다. 두 컬렉션 모두 무공변이고 컬렉션에서는 읽기만 하고 대상 컬렉션은 쓰기만 한다.
//이 경우 두 컬렉션의 원소 타입이 정확하게 일치할 필요가 없다. 즉 문자열이 원소인 컬렉션에서 객체의 컬렉션으로 원소를 복사해도 아무 문제 없다.
fun <T> copyData(source: MutableList<T>, destination: MutableList<T>) {
    for (item in source) {
        destination.add(item)
    }
}

//이 함수가 여러 다른 리스트 타입에 대해 작동하게 만들려면 두 번 째 제네릭 타입 파라미터를 도입할 수 있다.
//아래 코드는 source에서 destination으로 이동 시키려면 T는 R의 하위 타입만 가능하다.
fun <T: R, R> copyData(source: MutableList<T>, destination: MutableList<R>) {
    for (item in source) {
      destination.add(item)
    }
}

val ints = mutableListOf(1, 2, 3)
val anyItems = mutableListOf<Any>()
copyData(ints, anyItems)

//out 키워드를 타입을 사용하는 위치 앞에 붙이면 T는 in 위치에서 사용하는 메소드를 호출하지 못한다.
fun <T> copyData(source: MutableList<out T>, destination: MutableList<T>) {
    for (item in source) {
      destination.add(item)
    }
}
//반대로 in도 가능하다.
fun <T> copyData(source: MutableList<T>, destination: MutableList<in T>) {
  for (item in source) {
    destination.add(item)
  }
}
```
* 타입 선언에서 타입 파라미터를 사용하는 위치라면 어디에나 변성 변강자를 붙일 수 있다.
* 이런 경우 copyData 함수는 MutableList의 메소드 중에서 반환 타입으로 타입 파라미터 T를 사용하는 메소드만 호출할 수 있다.


### 스타 프로젝션: 타입 인자 대신 * 사용
* 타입 검사와 태스트에 대해 설명할 때 제네릭 타입 인자 정보가 없음을 표현하기 위해 스타 프로젝션을 사용한다.
  * 원소 타입이 알려지지 않은 리스트는 List\<*>라는 구문으로 표현할 수 있다.
* MutableList\<*>는 MutableList\<Any?>와 같지 않다.
  * MutableList\<Any?> 는 모든 타입의 원소를 담을 수 있다.
  * MutableList\<*> 는 정해진 구체적인 타입의 원소만을 담는 리스트지만 그 원소의 타입을 정활히 모른다는 사실을 표현한다.
    * 리스트의 원소 타입이 어떤 타입인지 모른다고 아무 원소나 담기는 어렵다. 
    * 리스트에서 원소는 얻을 수 있다. 결국 Any? 라는 최상위 타입이 있기 때문이다.
```kotlin
val list: MutableList<Any?> = mutableListOf('a', 1, "qwe")
val chars = mutableListOf('a', 'b', 'c')

val unknownElements: MutableList<*> = if(random().nextBoolean()) list else chars
//unknownElements.add(43) -> 컴파일러는 호출을 금지한다.
//unknownElements.first() -> Any?타입의 원소를 반환한다.
```
* 위 내용을 보면 스타 프로젝션은 아웃 프로젝션 타입으로 인식하는걸 볼 수 있다.
  * MutableList\<*>는 MutableList\<out: Any?>처럼 동작한다.
  * 타입을 모르면 input은 어렵지만 output은 가능하다.
  * 타입 파라미터를 시그니처에서 전혀 언급하지 않거나 데이터를 읽기는 하지만 그 타입에는 관심이 없는 경우와 같이 타입 인자 정보가 중요하지 않을 때도 스타 프로젝션 구문을 사용한다.
```kotlin
fun printFirst(list: List<*>) {
    if(list.isNotEmpty()) { //제네릭 파람을 사용하지 않기에 사용 가능하다.
      println(list.first())//output은 Any? 타입으로 가능하다.
    }
}

//스타 프로젝션 우회방법은 제네릭을 사용한다.
//스타 프로젝션을 사용할 때는 값을 만들어내는 메소드만 호출할 수 있고 그 값의 타입에는 신경을 쓰지 말아야 한다.
fun <T> printFirst(list: List<T>) {
    if(list.isNotEmpty()) {
      println(list.first())
    }
}
```
* 스타 프로젝션 예제
```kotlin
interface FieldValidator<in T> {
    fun validate(input: T): Boolean
}

object DefaultStringValidator: FieldValidator<String> {
    override fun validate(input: String) = input.isNotEmpty()
}

object DefaultIntValidator: FieldValidator<Int> {
  override fun validate(input: Int) = input>0
}


//이와 같이 사용하면 검증기를 쓸 때 문제가 생긴다. map에는 결국 FieldValidator<*>가 어떤 타입인지를 모르기 떄문 이다.
val validator = mutableMapOf<KClass<*>, FieldValidator<*>>()
validators[String::class] = DefaultStringValidator
validator[Int::class] = DefaultIntValidator

//검증기를 원하는 타입으로 캐스팅하면 이런 문제는 고칠 수 있다.
//하지만 그런 타입 캐스팅은 안전하지 못하고 권장할 수 없다.
val stringValidator = validators[String::class] as FieldValidator<String>
stringValidator.validate("")

//타입 캐스팅은 통과하지만 결국 validate에서 문제가 생긴다.
//검증기 캐스팅이 잘못되면 이 또한 에러가 난다.
```
* 이와 같은 방법은 타입 안전성을 보장할 수도 없고 실수하기 쉽다. 개발자의 책임이 된다.
* 개선된 코드
* 검증기를 등록하거나 가져오는 작업을 수행할 때 타입을 제대로 검사하게 캡슐화한다.
* 캐스팅 오류를 컴파일 시 발생시키지만 Validators 객체가 맵에 대한 접근을 통제하기 떄문에 맵에 ㄷ잘못된 값이 들어가지 못하게 막는다.
```kotlin
object Validators {
    private val validators = mutableMapOf<KClass<*>, FieldValidator<*>>
  
    fun <T: Any> registerValidator (kClass: KClass<T>, fieldValidator: FieldValidator<T>) {
        validators[kClass] = fieldValidator
    }
  
  @Suppress("UNCHECKED_CAST")
  operator fun <T: Any> get(kClass: KClass<T>): FieldValidator<T> {
      validators[kClass] as? FieldValidator<T>?: throw IllegalArgumentException("No validator for")
  }
}

Validators.registerValidator(String::class, DefaultStringValidator)
Validators.registerValidator(Int::class, DefaultIntValidator)
```
* 안전하지 않은 로직은 내부에 감춰져있다.
* 