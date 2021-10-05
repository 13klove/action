# 2장 변수, 함수, 클래스, 프로퍼티, 스마트 캐스트, 예외처리

## 함수
* 함수에 fun을 붙여 함수임을 나타낸다.
```kotlin
fun getId() {

}
```

* param에는 변수명 뒤에 ':' 와 type을 작성한다.
```kotlin
fun getId(userId: String) {

}
```

* class보다 상위에 함수 선언 가능
```kotlin
package chat02

fun getId() {

}

class MethodSample {

}    
```

* return 의 타입은 param 다음에 명시해야 한다.
```kotlin
fun getId(): String {
    return "in action"
}
```

* 함수의 본문은 2가지로 구분된다.
* 블록이 본문인 함수, 식이 본문인 함수
* 식이 본문인 경우 return type을 명시 하지 않아도 되지만 가독성을 위해 표기하자
* kotlin은 삼항연산자가 없지만 if가 식으로 값을 리턴하기에 if를 잘 사용하면 된다.
```kotlin
fun getNum(a: Int, b: Int): Int {
    return if(a>b) a else b
}

fun getNum(a: Int, b: Int): Int = if(a>b) a else b
```

## 변수
* 변수 선언시에 초기화가 된다면 type 선언 안해도 된다.
* 초기화가 안된 경우 type선언 해야 한다.
```kotlin
val str = "kotlin"
val str: String
str = "kotlin"

var str = "kotlin"
var str: String
str = "kotlin"
```

* kotlin은 변수 선언시에 변경가능과 불가능으로 구분된다.
* 되도록 변경불가능으로 개발하고 필요시에 변경가능을 사용해라
* 변경불가능으로 개발할 경우 함수형 코드에 더 가깝다.
```kotlin
val str = "kotlin" // 변경 불가능
str = "abc" //불가능
var muStr = "kotlin" //변경 가능
muStr = "action" //가능

val a = 2
val b = 3
val num = if (a > b) a else b 
//이런 경우 컴파일러가 판단하여 num은 1번의 초기화만 된다고 판단하여 에러가 안난다.
```

* val는 참조는 불변이고 참조가 가리키는 object의 값은 변경 간능하다.
```kotlin
val list = arrayListOf(1, 2, 3, 4)
list.add(7)
list.add(8)
```

## 문자열 템플릿
* 변수를 문자열 안에서 사용 가능하다.
* '{}'를 사용하면 식도 표현할 수 있다.
```kotlin
val name = "kotlin"
val pr = "my name is $name"
val list = arrayListOf(1, 2, 3, 4)
val age = "age: ${list.first()}"
```

## class와 property
* class 기본 개념
  * class는 data 캡슐화
  * 캡슐화된 data를 하나의 주체에 가두기 위함
  * java는 field에 데이터를 저장, 접근은 private로 선언해 외부에서 접근 불가능하게 한다.
  * 접근하려면 접근자 메서드(getter, setter)를 통해 한다.
  * kotlin에서는 field와 접근자 메서드를 묶어 property라고 한다.
  
* property 특이사항으로 변수의 이름으로 접근해도 자동으로 getter, setter가된다.  
```kotlin
fun getPerson() {
    val person = Person("kotlin", 25)
    println("${person.name} | ${person.age}")
    person.age = 30
    println("${person.name} | ${person.age}")
}

class Person(
  val name: String, // 읽기전용 field, getter 지원 
  var age: Int // 쓰기가능 field, getter, setter 지원
  ) {}
```
## 커스텀 접근자
* property의 접근자(getter)를 정의할 수 있다.
  * 이런 경우 field 접근시에 매번 개산을 하게 된다.
  * 커스텀 접근자와 param이 없는 메서드는 성능차이는 없으나 가독성이 더 좋다.
  
```kotlin

```





