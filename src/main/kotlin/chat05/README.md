# 람다로 프로그래밍

* 람다 식과 멤버 참조
* 함수형 스타일로 컬렉션 다루기
* 시퀀스: 지연 컬렉션 연산
* 자바 함수형 인터페이스를 코틀린에서 사용
* 수신 객체 지정 람다 사용

* 람다는 기본적으로 다른 함수에 넘길 수 있는 코드다.
* 람다를 통해 공통 코드구조를 뽑아낼 수 있다.

## 람다 식과 맴버 참조

### 람다 소개: 코드 블록을 함수 인자로 넘기기
* 과거 자바에서는 무명 내부 클래스를 통해 코드를 함수에 넘기거나 변수에 저장할 수 있었다.
* 함수형 프로그래밍에서는 함수를 값처럼 다루는 접근 방식으로 람다를 통해 넘길 수 있다.
```java
button.setOnclickListener(new OnClickListener() {
    @Override
    public void onClick(View view) {
        
    }
})
```
* 자바는 위의 코드와 같이 무명 내부 클래스를 통해 정의 할 수 있지만 보기 어렵고 번잡하다.
```kotlin
button.setOnClickListener {
//구현
}
```
* kotlin은 람다를 통해 코드의 간결함을 줄 수 있다.

### 람다와 컬렉션
* 코드 기반으로 예시를 본다.
* findTheOldest보다 maxOf에 람다 혹은 맴버 참조를 넘긴 코드가 더 간결함으 느낄 수 있다.
```kotlin
data class Person(val name: String, val age: Int)

fun findTheOldest(persons: List<Person>) {
    var maxAge = 0
    var oldestPerson: Person? = null
    for(person in persons) {
        if (person.age > maxAge) {
            maxAge = person.age
            oldestPerson = person
        }
    }

    println(oldestPerson)
}

fun findTheOldstLamda(persons: List<Person>) {
    val person = persons.maxOf { it.age }
    val person02 = persons.maxOf(Person::age)

    println(person)
    println(person02)
}
```

### 람다 식의 문법
* 람다를 바로 넘길 수도 있고, 변수에 저장할 수도 있다.
* 람다는 중괄호로 둘러싸여 있다.
* 파라미터와 본문은 '->'로 구분 된다.
* 람다를 바로 실행하려면 run 사용을 권장한다.
* 바로 사용되면 파라미터를 받을 수 없다.
* 함수의 파라미터에서 람다가 마지막 파라미터인 경우 괄호 밖에서 사용한다.
* 람다의 파라미터가 1개이고 타입 유추가 된다면 it으로 코드를 간결하게 만들 수 있다.
* 람다를 변수에 저장하는 경우에는 반드시 파라미터 타입을 명시 해야 한다.
* 람다에서 가장 마지막에 있는 식이 결과 값이 된다.
```kotlin
{x: Int, y: Int -> x+y}

val lamda = {x: Int, y: Int -> x+y}
lamda(1, 2)

run { println("3") }

val people = listOf(Person("a", 29), Person("b", 30))
people.joinToString(separator = "") { it.name }
```

### 현재 영역에 있는 변수에 접근
* 람다는 함수 안에 정의하면 함수의 파라미터, 로컬 변수까지 접근 가능하다.
* 접근한 변수들을 '람다가 포획한 변수'라고 한다.
* 함수 내부에서 사용된 람다는 로컬 변수의 생명주기는 함수가 반환되면 끝난다.
* 어떤 함수가 자신의 로컬 변수를 포획한 람다를 반환하거나 다른 변수에 저장한다면 로컬 변수의 생명주기가 달라진다.
* 파이널 변수를 포획한 경우에는 람다 코드를 변수 값과 함께 저장한다. 파이널이 아닌 변수를 포획한 경우에는 변수를 특별한 래퍼로 감싸서 나중에 변경하거나 읽을 수 있게 한 다음 래퍼에 대한 참조를 람다 코드와 함꼐 저장한다.
  * 이를 클로저하고 부른다.
* 람다를 이벤트 핸들러나 다른 비동기적으로 실행되는 코드로 활용하면 우리가 의도하지 않은 경우가 발생할 수 있다.
```kotlin
//prefix는 람다가 포획한 변수가 된다.
fun printMessagesWithPrefix(msgs: List<String>, prefix: String) {
    msgs.forEach {
        println("$prefix - $it")
    }
}

//clientErrors, serverErrors가 팜다가 포획한 변수가 된다.
fun printProblemCounts(response: List<String>) {
    var clientErrors = 0
    var serverErrors = 0
    
    response.forEach { 
        if (it.startsWith("4")) clientErrors ++
        else if(it.startsWith("5")) serverErrors ++
    }

    println("client: $clientErrors server: $serverErrors")
}
```

### 맴버 참조
* '::'를 사용하는 식을 멤버 참조라고 한다.
* Person(클래스)::(참조 구분)age(멤버)
* 참조 대상이 함수, 프로퍼티인지와는 관계없이 멤버 참조 뒤에는 괄호를 안넣는다.
* 람다와 같은 타입니다.
* 생성자 참조도 만들 수 있다.
* 확장 함수도 멤버 함수와 똑같은 방식으로 참조할 수 있다.
* 바운드 멤버 참조로 변수명으로 멤버 참조가 가능하다.
```kotlin
person.maxBy(Person::age)
data class Person(val name: String, val age: Int = 0) {
  fun toMessage(): PersonMessage = PersonMessage(name, age)
}

data class PersonMessage(val name: String, val age: Int){
  fun toMessage(person: Person): PersonMessage = PersonMessage(person.name, person.age)
}

fun sendEmail(person: Person, msg: String) {
  println("send msg")
}

fun main() {
  val action = {person: Person, msg: String -> sendEmail(person, msg)}
  val newAction = ::sendEmail

  val list = listOf(Person("a", 29), Person("b", 30))
  list.map(Person::toMessage)

  //생성자 멤버 참조
  val createPerson = ::Person
  createPerson("alice", 29)
  val list2 = listOf("a", "b")
  list2.map(::Person)

  //바운드 멤버 참조
  val p = Person("b")
  p::toMessage
}
```

## 컬렉션 함수형 api
### 필수적인 함수: filter와 map
### all, any, count, find
### groupBy 
### flatMap, flatten
```kotlin
    val collections = listOf(Person("a", 29), Person("b", 30), Person("aa", 13), Person("bb", 25), Person("c", 10))
    //filter
    collections.filter { it.name == "a" }
    //map
    collections.map { it.name+"map" }

    val map = mapOf("a" to 29, "b" to 30)
    map.filterKeys { it == "b" } // mapKeys
    map.filterValues { it == 29 } //mapValues

    //all
    collections.all { it.age > 20 }
    //any
    collections.any { it.age > 30 }
    //count
    collections.count { it.name == "a" }
    //find
    collections.find { it.age == 30 }
    //find도 데이터가 없는경우 null을 반화한다 명시적으로 아래 메서드를 쓰는 것도 좋다.
    collections.firstOrNull { it.age == 30}
    collections.groupBy {
      it.name.toList()[0]
    }
    
    val a = collections.groupBy (Person::age) {
      it.name + "abc"
    }
    val numberList = listOf(listOf(1, 2, 3), listOf(4, 5, 6, 7, 8))
    val flatmap = numberList.flatMap { it }
    val flatten = numberList.flatten()
```

## 지연 계산 컬렉션 연산
* 컬렉션 함수는 즉시 연산한다. 이는 컬렉션 함수를 연쇄하면 매 단계마다 중간 결과를 새로운 컬렉션에 임시로 담는다.
* 시퀀스를 사용하면 중간 임시 컬렉션을 사용하지 않고도 컬레션 연산을 연쇄 한다.
* 원소가 적은경우는 컬렉션 함수를 사용해도 된다. 하자미나 컬렉션이 클수록 시퀀스를 권장한다.
* 최종 연산자가 있어야 시쿼스는 연산을 시행한다.
```kotlin
val collections = listOf(Person("a", 29), Person("b", 30), Person("aa", 13), Person("bb", 25), Person("c", 10))
collections.asSequence() //시퀀스로 변환
  .map(Person::name) //연산
  .filter{ it.startsWith("a")} //연쇄 연산
  .toList() //결과 반환.
```

### 시쿼스 연산 실행: 중간 연산과 최종 연산
* 중간 연산과 최종 연산으로 나뉜다.
* 중간 연산은 다른 시퀀스를 반환하고 최종 연산은 결과를 반환한다.
* 중간 연산은 항상 지연 계산된다.
* 연산 수행 순서!
  * 직접 연산을 구현한다면 map 함수를 각 원소에 대해 먼저 수행해서 새 시퀀스를 얻고, 그 시퀀스에 다시 filter를 수행 한다.
  * 시퀀스는 모든 연산은 각 원소에 대해 순차적으로 적용된다. 즉 첫 번째 원소가 처리되고, 두 번째원소가 처리
  * 이 차이는 조건이 모두 만족되면 시쿼스는 모든 원소를 연산할 필요가 없다.
* 중간 연산자의 순서도 고려해야한다. 코드로 본다.
```kotlin
val collections = listOf(Person("a", 29), Person("b", 30), Person("aa", 13), Person("bb", 25), Person("c", 10))

collections.asSequence().map {
  it.name+"abc"
  it
}.filter { it.age > 10 }

//map이 filter 뒤에 있는 경우 filter된 원소만 map을 거치기에 속도가 더 빠르다.
collections.asSequence().filter { it.age > 10 }
  .map {
    it.name + "abc"
    it
  }
```

### 시퀀스 만들기
* asSequence를 호출해 시퀀스를 만들 수 있다.
* generateSequence 함수를 통해 시퀀스로 만들 수 있다.
```kotlin
    val naturalNumbers = generateSequence(0) { it +1}
    val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
    println(numbersTo100.sum())
```

## 자바 함수형 인터페이스 활용
* 코틀린에서는 무명 클래스 인스턴스 대신 람다를 넘길 수 있다.
* 함수형 인터페이스 또는 SAM 인터페이스라고 한다.
* 자바에서 Runnable, Callable과 같은 함수형 인터페이스다.

### 자바 메소드에 람다를 인자로 전달
* 함수형 인터페이스를 인자로 원하는 자바 메소드에 코틀린 람다를 전달할 수 있다.
* 코틀린은 람다를 함수형 인터페이스로 컴파일러가 변환해 넘겨 준다.
* 람다와 무명 객체는 차이가 있다.
  * 객체를 명시적으로 선언하는 경우 메소드를 호출할 때마다 새로운 객체가 생성된다.
  * 람다는 반복사용 된다.
  * 람다가 변수를 포획한다면 새로운 인스턴스를 생성하게 된다.

### SAM 생성자: 람다를 함수형 인터페이스로 명시적으로 변경
* SAM 생성자는 람다를 함수형 인터페이스의 인스턴스로 변환할 수 있게 컴파일러가 자동으로 생성한 함수다.
* 컴파일러가 자동으로 람다를 변환 못하는 경우 SAM 생성자를 사용한다.
* 람다 앞에 사용될 수신객체의 이름을 명시한다.
  * Runnable {println("all done!"}

## 수신 객체 지정 람다: with와 apply
* 수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메소드를 호출할 수 있다.
* '수신 객체 지정 람다'라고 부른다.

### with 함수
* 파라미터가 2개있는 함수다. 
* 인자로 받은 람다 본문에서는 this를 사용해 그 수신 객체에 접근할 수 있다.(this 생략가능)
```kotlin
fun alphaber() = with(StringBuilder()) {
    for (letter in 'a'..'z') {
        append(letter)
    }
    toString()
}
```

### apply 함수
* with와 비슷하지만 차이점은 수신객체를 반환한다.
* apply는 확장 함수로 정의 되어 있다.
* apply 함수는 객체의 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화에 유용하다.
```kotlin
fun alhpabet() = StringBuilder().apply { 
    for (letter in 'a'..'z') {
        append(letter)
    }
}.toString()
```