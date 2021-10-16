# 3장 함수 정의와 호출

### * 컬렉션, 문자열, 정규식을 다루기 위한 함수
### * 이름을 붙인 인자, default param, 중위 호출
### * 확장 함수와 프로퍼티

## 컬렉션
* 간단하게 컬렉션 사용법을 익힌다.
```kotlin
val set = setOf(1, 2, 3, 4, 4)
val list = arrayListOf(1, 2, 3, 4, 5)
val map = hashMapOf(1 to "a", 2 to "b")
map.javaClass //-> java에서 class를 가리키는 표현이다.
```

* kotlin은 param에 이름을 붙일 수 있다.
```kotlin
fun sum(left: Int, right: Int): Int = left + right

sum(1, 3)
sum(left = 1, right = 3)
```

* kotlin은 default param으로 value를 정의 할 수 있다.
  * kotlin은 java에서 오버로딩을 통한 중복 코드를 default param으로 대신 할 수 있다.
  * 주의 사항은 default param은 method 선언에서 정해진다. default param이 변경되면 전체에 영향을 준다.
  * 자바 호환을 생각한다면 @JvmOverloads를 선언 한다.
```kotlin
class Man(
    val id: String,
    val name: String,
    val gender: String,
    val age: Int,
) {

    //이와 같이 오버로딩이 되지만 kotlin은 default param을 사용하여 중복되는 내용을 제거 할 수 있다.
    fun getName(id: String): String = name
    fun getName(id: String, gender: String): String = name

    fun getAge(id:String, name: String = "bob", gender: String = "man"): Int = age

}
```

## 정적인 유틸리티 class 제거: 최상위 함수와 property
* kotlin은 method를 class 밖에서 선언이 가능하다.
```kotlin
package chat03

fun sum(left: Int, right: Int): Int = left + right
```
* 사용하기 위해서는 package가 같은면 바로 method명으로 사용 가능하고 package가 다르면 import를 통해서 사용가능 하다.
* jvm이 자동으로 .kt 이름으로 class 생성한다.
* @JvmName으로 jvm이 자동으로 생성하는 class 이름 변경 가능하다.
```kotlin
@field: JvmName("Action")
package CollectionUnit
```
* 자바에서 상수선언 kotlin 예시
```kotlin
public static final int SIZE = 15 //java
const val SIZE = 15 //kotlin
```

## 확장 함수, 확장 프로퍼티
* 확장 함수는 class 내부에 선언되지 않고 외부에서 선언된 함수다.
* 확장 함수는 class의 캡슐화를 깨지 않는다.
* 확장 함수는 수신객체의 대부분의 요소가 사용가능 하지만 private, protected 맴버는 접근 불가능 한다.
* 서로 다른 package에서 확장 함수를 사용하려면 import 한다.
* class 내부의 함수와 확장함수가 같은 경우 import를 통해서 사용가능 하고 'as' 로 이름을 변경해 사용 가능 하다.
* class 내부에 선언된 확장함수는 내부에서만 작동한다.
```kotlin
package chat03

import chat03.getExMethodInfo as info

fun main() {
    val exMethod = ExMethod("1", "bob", "man", 15)
  
    val innerMethodInfo = exMethod.getExMethodInfo()
    println(innerMethodInfo)
  
    val exMethodInfo = exMethod.info()
    println(exMethodInfo)
}

// import만 하면 어디서나 사용 가능
// ExMethod -> 수신객체타입 // this는 수신객체(생략 가능)
fun ExMethod.getExMethodInfo(): String = "id: ${this.id} | name: $name" // | point: $point 불가능

class ExMethod(
    val id: String,
    val name: String,
    val gender: String,
    val age: Int,
) {
    
    private val point = "ex"

    fun getExMethodInfo(): String = "id: $id | name: $name | gender: $gender | age: $age"

}

class User(
  val name: String
){

  fun ExMethod.getInfo(): String = "id: $id | name: $name" //여기서만 사용 가능

}
```

## 확장함수로 유틸리티 함수 정의
* 확장 함수는 단지 정적 메소드 호출에 대한 문법적인 편의 일뿐이다. 그래서 class가 아닌 더 구체적인 타입을 수신 객체로 할 수 있다.
```kotlin
//이렇게 정의 되면 타입이 문자열 인경우만 사용 가능하다.
fun Collection<String>.join(left: String, right: String) {}
```
* 확장 함수는 하위 class에 오버라이드 할 수 없다.
* 확장 함수를 호출할때 수신객체로 지정한 변수의 정적 타입에 의해 어떤 확장 함수가 호출될지 결정 된다.
* 변수에 저장되는 동적 타입에 의해 결정 안된다.
```kotlin
fun main() {
  val button: View = Button()
  button.click() // button click이 호출된다. 즉 오버라이드 된다.

  button.show() // 오버라이드가 안되고 view show가 호출 된다.
  val exButton: Button = Button()
  exButton.show() //이와 같이 사용해야 button show가 호출 된다.
}

// 오버라이드
open class View {
  open fun click() = println("view click")
}
open class Button: View() {
  override fun click() = println("button click")
}

//확장함수
fun View.show() = println("view show")
fun Button.show() = println("button show")
```
* class 내부에서 선언시 해당 class에서만 적용 된다.
* 확장함수로 유틸리티 함수 정의 내용
```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ",",
    prefix: String = "",
    postfix: String = "",
): String {
    val result = StringBuilder(prefix)
    for((idx, value) in withIndex()) {
        if (idx > 0) result.append(separator)
        result.append(value)
    }
    result.append(postfix)
    return result.toString()
}
```

## 확장 프로퍼티
* class에 프로퍼티 추가가 가능하다
* 하지만 프로퍼티라는 이름으로 불릴 뿐 상태 저장은 불가능하다.
```kotlin
val String.lastChar: char
    get() = get(length-1)
```

## 컬렉션 처리: 가변길이 인자, 중의 함수
* 가변인자 함수
  * 인자의 개수가 달라질 수 있는 함수 정의
  * 함수 호출시 원하는 만큼 param을 넘길 수 있고 param은 컴파일러가 자동으로 배열에 넣어준다.
  * java는 '...'을 쓰지만 kotlin은 vararg를 사용한다.
  * kotlin은 배열을 가변인자로 넘길때 함수에 '*'를 분여 넘긴다.(스프레드 연산자)
```kotlin
fun <T> listOf(vararg elements: T): List<T> = if (elements.size > 0) elements.asList() else emptyList()

fun varargMethod(vararg name: String): String {
    val result = StringBuilder()
    for(value in name) {
        result.append(value)
    }
    return result.toString()
}

fun main() {
    println(varargMethod("a", "b", "c"))
    val list = arrayOf("d", "e", "f")
    println(varargMethod(*list))
}
```

* 값의 쌍 다루기: 중위 호출과 구조 분해
  * 중위호출의 예시는 'to'이다.
  * 중위호출은 인자가 하나인 메서드 혹은 확장함수에서 가능하다.
  * 중위호출을 허용하려면 method에 infix를 선언한다.
  * infix는 최상위 method에서는 사용 불가능
```kotlin
fun main() {
  val sample = Sample("1", "bob")
  sample checkMethod "alice" //1 - smapleName: bob - param: alice
  sample.viewMethod() //view click // check
}

class Sample(
  val id: String,
  val name: String
) {

  infix fun checkMethod(param: String) {
    println("$id - smapleName: ${this.name} - param: $param")
  }

  fun viewMethod() {
    val view = View()
    view.checkView("check")
  }
  infix fun View.checkView(param: String) {
    click() //여기서 함수는 view의 함수를 호출한다.
    println(param)
  }

}

open class View {
  open fun click() = println("view click")
}
```
* 구조 분해의 예시는 pair를 두 원소로 분리함을 예시로든다.
```kotlin
val (idx, value) = Pair("1", "a")
```
* 가변인자, 중위함수, 구조분해를 예시로든 map
```kotlin
fun <K, V> mapOf(vararg pairs: Pair<K, V>): Map<K, V> {} // map은 param으로 가변인자를 받는다.
val map = mapOf(1 to "a") // 'to'로 중위호출을 한다.
for((idx, value) in map){ // 구조분해로 loop를 돈다.
    println("index: $idx - value: $value")
}
```

## 코드다듬기: 로컬함수
* 함수 내부에 함수를 호출 할 수있다.
* 확장함수 내부에도 로컬함수를 호출 할 수 있다.
* 이는 예제로 보는게 더 좋다.
```kotlin
class UserInfo(val id: String, val name: String, val address: String){}

fun saveUserInfo01(userInfo: UserInfo) {
    if (userInfo.name.isEmpty()) {
        throw IllegalArgumentException("can't save user ${userInfo.id}: empty Name")
    }
    
    if (userInfo.address.isEmpty()) {
        throw IllegalArgumentException("can't save user ${userInfo.id}: empty Address")
    }
}

fun saveUserInfo02(userInfo: UserInfo) {
    //local method는 위부 method의 param을 사용할 수 있다.
    fun valid(value: String, key: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException("can't save user ${userInfo.id}: empty $key")
        }
    }
    
    valid(userInfo.name, "name")
    valid(userInfo.address, "address")
}

//확장함수를 이용한 local method
fun UserInfo.userInfoValid() {
    fun valid(value: String, key: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException("can't save user ${this.id}: empty $key")
        }
    }

    valid(name, "name")
    valid(address, "address")
}

fun saveUserInfo03(userInfo: UserInfo) {
    userInfo.userInfoValid()
}
```