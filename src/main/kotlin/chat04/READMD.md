# 클래스, 객체, 인터페이스

## 클래스 계층 정의
* kotlin interface 추상 method, 구현 method 둘다 가능하고 field는 불가능하고 추상 property는 가능하다.
* interface를 상속 받은 비추상 class는 반드시 추상 method를 구현 해야 한다.
* kotlin은 interface, class 상속시 ":"를 사용한다.
* interface는 원하는 만큼 상속이 가능하다.
* interface의 맴버는 final이 불가능하다.
* class가 동일한 추상 method가 있는 interface를 다수 상속받은 경우 반드시 사용할 method를 정의 해야 한다.
```kotlin
package chat04

interface Clickable {
    fun click()
    fun showOff() {
        println("clickable showOff")
    }
}

interface Focusable {
    fun setFocust(b: Boolean) = println("i ${if (b) "got" else "lost"} focus")
    fun showOff() {
        println("focusable showOff")
    }
}

class Button: Clickable, Focusable {
    override fun click() {
        println("i was clicked")
    }

    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
        println("total")
    }
}

fun main() {
    val button = Button()
    button.showOff()
}
```

## open, final, abstract 변경자 default final
* 쉽게 상속이 가능하다면 우리는 취약한 기반 클래스라는 문제를 가진다.
  * 처음에 상속을 통해 구현 했지만 상위 클래스의 변화로 하위 클래스에도 영향을 주게 된다.
  * 이를 해결하기 위해 kotlin은 클래스, method가 기본으로 final로 정의 되어 있다.
* 상속을 허용하려면 open을 명시하면 된다.
  * 즉 상속을 하고 싶은 class, method, property에 open을 명시한다.
* 하위 클래스에서 오버라이드된 method는 자동으로 open으로 되어 있다. 하위 class에서 오버라이드된 method의 상속을 방지하려면 final을 붙여야 한다.
* abstract
  * java와 같이 추상 클래스 선언이 가능하다 abstract class
  * 단독 인스턴스는 불가능하다.
  * 추상 맴버는 하위 클래스에서 구현해야 한다.
  * 추상 맴버는 항상 open이라 명시할 필가 없다.
```kotlin
abstract class Animated {

    abstract fun animate()
    
    //비추상 method라 상속을 하려면 open을 명시해야 한다.
    //open fun stopAnimate()
    fun stopAnimate() {
        
    }

}
```
* 상속 제어 변경자 
* |변경자|오버라이드|설명|
  |------|---|---|
  |final|오버라이드 불가능|class 맴버의 default|
  |open|오버라이드 가능|오버라이드 하기 위해서는 필요|
  |abstract|반드시 오버라이드|추상 class 맴버만 사용가능|
* |override|오버라이드 표기|오버라이드한 맴버는 표기 되고 기본이 open이다 오버라이드를 방지하려면 final을 선언해야 한다.|

## 가시성 변경자 기본이 public
* 가시성 변경자는 class 외부 접근을 제어한다.
* 가시성 변경자로 class 의존 규칙을 줄 수 있다.
* kotlin은 java와 달리 package는 namespace를 관리만하고 대신 internal이 도입 되었다.
* internal은 모듈 내부에서만 호출이 가능하다.
  * 모듈은 다양한 범위다 internal과 같이 빌드되는 환경에서만 사용이 가능하다.
* |변경자|접근 범위|
    |------|---|
  |public|모든 곳에서 접근|
  |internal|같은 모듈에서 접근|
  |protected|하위 class만 접근|
* |private|같은 class만 접근|


## 내부 class와 중첩된 class - 기본이 중첩된 class
* kotlin의 중첩 class는 명시적으로 요청하지 않으면 바깥쪽 class 인스턴스에 대한 접근 권한이 없다.
* 코드로 설명한다.
```kotlin
interface State: Serializable

interface View {
    fun getCurrentState(): State
    fun restoreState(state: State)
}

class Button02: View {
    
    val num = 5
    
    override fun getCurrentState(): State = ButtonState()

    override fun restoreState(state: State) {
        TODO("Not yet implemented")
    }

    // ButtonState는 inner class가 아닌 중첩 class다 즉 Btton02와 접근 권한이 없다.
    // 즉 ButtonState에서는 num, getCurrentState, restoreState에는 접근 못한다.
    class ButtonState: State {
        
    }

}
```
* 내부 class를 정의하고 사용하려면 아래와 같이 사용 해야 한다.
* outer class를 반환하려면 this@(OuterClass)를 명시한다.
```kotlin
class Button03: View {

    val num = 5

    override fun getCurrentState(): State = ButtonState()

    override fun restoreState(state: State) {

    }

    // inner class로 선언 되어 outer class의 맴버를 사용할 수 있다.
    inner class ButtonState: State {
       fun innerMethod() {
           println("Button03 property num: $num")
           getCurrentState()
       } 
        
       fun returnOuter(): Button03 = this@Button03 
    }

}
```
* 이처럼 예시를 Serializable로 한 이유는 Button02의 ButtonState는 Button02와 참조 관계가 없어 serializabe오류가 안난다. 
* 하지만 Button03의 ButtonState는 Button03을 참조 관계가 있어 에러가 난다.
* java와 반대다
  * java에서 중첩 class를 사용하면 자동으로 inner class // kotlin에서는 inner를 명시해야 한다.
  * java에서 중첩 class를 사용하면 static class // kotlin에서는 자동으로 중첩 class


## 봉인된 class - class 계층 정의시 계층 확장 제한
* seeled 키워드로 하위 class에 제한을 둔다. interface는 불가능
* sealed가 선언된 class안에 class를 정의 해야 한다.
* sealed는 기본이 open이다.
```kotlin
sealed class Expr {

    class Num(val value: Int): Expr()
    class Sum(val left: Expr, val right: Expr): Expr()

}

fun check(e: Expr): Int = when(e) {
    is Expr.Num -> 2
    is Expr.Sum -> 3
    // else가 있는 이유는 Expr을 상속받은 다른 class가 있을 수 있어 정의 하지만
    // sealed로 class 제한을 두게 되면 else가 없어도 된다.
    // 단 Expr에 다른 class가 추가 되면 when에도 추가를 해야 에러가 안난다.
    //else -> throw IllegalArgumentException("exception")
}
```

## 뻔하지 않은 생성자와 propery와 class
* kotlin은 주생성자, 보조생성자, 초기화 블록으로 초기화 할 수 있다.
* 주생성자는 class 본문 밖에서 선언한다.
* 보조생성자는 class 본문에서 선언한다.
* 초기화 블록으로 초기화 로직을 추가할 수 있다.
* new keyword 없이 인스턴스 가능하다.
* 주생성자는 제한적이어서 별도의 코드를 포함 할 수 없어 초기화 블록이 필요하다.
* 초기화 블록은 여러개 생성이 가능하다.
* 주생성자의 파람은 초기화 식과 초기화 블록에서만 사용가능하다.
```kotlin
class User04(val name: String) 

// 여기서의 name은 val, var로 선언되지 않았다.
// 즉 property가 아니고 param이다.
// 해당 값은 init block에서만 사용 가능하다.
class User05(name: String) {
    init {
        println(name)
    }
}
```
* 아래 코드는 모두 동일한 동작을 한다.
```kotlin
//주생성자
class User01(val name: String)

//주생성자와 초기화 블록
class User02(_name: String) {
    
    val name: String
    
    init {
        name = _name
    }
    
}

//주생성자와 초기화
class User03(_name: String) {
    
    val name = _name
    
}
```
* 생성자로 default value를 지정할 수 있다.
```kotlin
class User06(val name: String, val info: String = "no")
```
* class에 기반 class가 있다면 반드시 기반 class의 생성자를 호출 해야 한다.
* 부생성자는 class를 주생성자가 아닌 다른 방식으로 초기화 한다.
* 부생성자는 constructor를 사용해 구성 한다.
* this를 사용해 다른 생성자에 위임이 가능하다.
```kotlin
open class User07(val name: String, val media: String = "none") {
    
    var age = 0
    
    constructor(name: String, age: Int, media: String) : this(name, media) {
        this.age = age
    }
}

class TwitterUser(name: String) : User07(name, "twitter") { //User07(name,15,  "twitter") { 
    
}
```
* class를 정의할떄 생성자를 정의 하지 않으면 defualt로 정의된다.
* default로 정의 되면 "()" 사용해야 한다.
```kotlin
class User08

fun main() {
    User08()
}
```
* class의 외부에서 인스턴스를 방지하려면 private를 사용한다.
  * private를 사용하는 경우는 싱글턴과 동반객체를 통해서 팩터리 패턴에 유용하다.
* 인터페이스에 선언된 property 구현
  * 추상 property만 가능 하다.
  * 추상 property는 뒷받침하는 field나 getter를 제공하지 않는다.
```kotlin
interface Action {
  val name: String
  var age: Int

  //getter를 제공하고 있어 구현하지 않아도 된다.
  val role: String
    get() = "npc"

  fun userAction() {
    println(name + age)
  }

}

class UserAction(override val name: String, override var age: Int) : Action {

}

class NpcAction(name: String, age: Int): Action {

    override val name = name
    override var age = age
        set(value) {
            field = value*0
        }
        get() {
            return 0
        }

}

fun main() {
    UserAction("a", 12).userAction()
    NpcAction("b", 100).userAction()
}
```
* getter, setter에서 뒷받침하는 field에 접근
  * 접근자의 가시성은 기본적으로 property와 같고 변경도 가능하다.
  * field를 통해서 뒷받침하는 field에 접근 가능하다.
```kotlin
class UserProperty(val name: String) {
    var address: String = "undefined"
        private set(value) {
            field = address
        }
        get() = name+address
}
```

## 컴파일러가 생성한 method - data class, class 위임
* 데이터를 담고있는 class에 우리가 기본으로 구현해야하는 method가 있다.
  * equals, toString, hashCode
  * kotlin도 동일하게 구현해야 한다.
  * 하지만 data class를 사용하면 equals, toString, hashCode등 3가지 method는 자동으로 생성된다.
  * 주의 사항으로 주생성자에 있는 property만 적용 된다.
  * data class는 copy method도 제공한다. 
    * 데이터의 변경보다는 copy를 통한 불변을 권장한다.
```kotlin
// 기본 class
class Client(val name: String, val age:Int, val gender: String) {
    override fun equals(other: Any?): Boolean {
        if (other !is Client) return false
        return other.name == name && other.age == this.age && other.gender == gender
    }

    override fun hashCode(): Int {
        return name.hashCode()*31+age+gender.hashCode()+31
    }

    override fun toString(): String {
        return "Client(name='$name', age=$age, gender='$gender')"
    }
}

// data class //// equal, hashCode, toString 자동구현
data class Client(val name: String, val age:Int, val gender: String) {

}
```

* class 위임 - by keyword
* project가 커지면서 쉽게 발생하는 이슈는 상속을 통한 이슈다. 상위 클래스가 변경되면 하위 클래스도 변경되어야 한다.
* kotlin은 이런 일을 방지하고자 기본이 final이고 open을 해야 상속이 가능하다.
* 상속을 하지 않고 class에 새로운 동작을 추가할 수 있다.(확장 함수도 그러지 않나요?!)
  * 데코레이터 패턴
    * 상속을 허용하지 않은 class 대신 사용할 수 있는 class를 정의하고 인터페이스를 상속 받는다.
    * class에 상속이 불가능한 class를 field로 넣고 method를 다시 정의한다.
  * 복잡한 내용이고 필요없는 로직이 많이 들어간다. 하지만 kotlin by를 통해 간단하게 구현 가능하다.
```kotlin
class CountingSet<T>(
  //위임 받을 객체를 선언한다.
    val set: MutableCollection<T> = hashSetOf()
): MutableCollection<T> by set { // by 키워드로 set에 위임함을 명시한다.
    
    var count = 0

  // override를 통해 method를 재정의 한다.
    override fun add(element: T): Boolean {
        count++
        return set.add(element)
    }
}
```

## Object keyword - class 선언과 instant 생성
* class를 정의 하면서 동시에 인스턴스를 생성한다.
  * 객체 선언은 싱글턴을 정의하는 방법 중 하나다.
  * 동반 객체는 인스턴스 메소드는 아니지만 어떤 class와 관련 있는 메소드와 팩터리 메소드를 담을 떄 쓰인다. 접근은 동반 객체를 담은 class를 통한다.
  * 객체 식은 자바의 무면 내부 class 대신 쓰인다.
* 객체 선언 - 싱글턴 만들기
  * kotlin은 싱글턴을 언어에서 지원한다.
  * 객체 선언은 class선언과 그 class에 속한 단일 인스턴스의 선언을 합친 선언이다.
  * property, method, init를 사용할 수 있고 생성자는 불가능하다.
    * 싱글턴 객체는 객체 선언문이 있는 위치에서 생성자 호출 없이 즉시 만들어지기에 생성자가 필요 없다.
  * 객체 선언도 class, interface를 상속할 수 있다.
  * class 안에서 객체를 선언할 수도 있다. class가 여러번 생성되도 객체 선언은 1개만 존재 한다.
```kotlin
object Payroll {

    val allEmployees: ArrayList<Person>

    init {
        allEmployees = arrayListOf<Person>()
    }

    fun calculateSalary() {
        for(person in allEmployees){
            println(person.name)
        }
    }
}

//주로 상태가 저장되지 않는 경우 사용 된다.
object CaseInsensitiveFileComparator: Comparator<File> {
    override fun compare(o1: File, o2: File): Int {
        return o1.path.compareTo(o2.path, ignoreCase = true)
    }

}

//class 내부에서도 선언 가능하다.
data class Person00(val name: String) {
    object NameComparator: Comparator<Person00> {
        override fun compare(o1: Person00, o2: Person00): Int = o1.name.compareTo(o2.name)
    }
}
```

* 동반객체 - 팩토리 메소드와 정적 맴버가 들어가는 장소
  * kotlin은 static을 지원 안한다. 대신 최상위 함수, 객체 선언으로 대신한다.
  * class안에 companion을 사용하면 동반객체로 만들 수 있다.
  * class안의 모든 property, method에 접근 가능 즉 private도 접근 가능하다.
  * 팩터리 패턴 구현이 쉽다.
  * 동반객체는 오버라이드 할 수 없다.
```kotlin
class UserObject {

    val nickname: String

    constructor(email: String) {
        nickname = email.substringBefore("@")
    }

    constructor(id: Int) {
        nickname = getNicknameById(id)
    }

    fun getNicknameById(id: Int): String = "hello"

}

class UserCompanion private constructor(val nickname: String){
    
    companion object {
        fun newEmailUser(email: String): UserCompanion {
            return UserCompanion(email.substringBefore("@"))
        }
        
      //여기서 getNicknameById는 UserCompanion에 있는 method는 사용 불가능하다.
      //동반 객체 안에있는 함수 이거나 object와 같이 선언과 동시에 인스턴스화되는 method만 가능
      //spring이면 주입받은 객체도 사용가능해 보인다.
        fun newIdUser(id: Int): UserCompanion {
            return UserCompanion(getNicknameById(id))
        }

        private fun getNicknameById(id: Int): String = "hello"
    }
  
  //fun getNicknameById(id: Int): String = "hello"


}
```
* 동반 객체를 일반 객체처럼 사용
  * 동반 객체는 class안에 정의된 일반 객체이다. 동반 객체에 이름을 부이거나 interface 상속, 확장함수, property를 정의할 수 있다.
```kotlin
class UserCompanion private constructor(val nickname: String){

  //이름 정의
    companion object Loader {
        fun newEmailUser(email: String): UserCompanion {
            return UserCompanion(email.substringBefore("@"))
        }

        fun newIdUser(id: Int): UserCompanion {
            return UserCompanion(getNicknameById(id))
        }

        private fun getNicknameById(id: Int): String = "hello"
    }


}
```
* 동반 객체 interface 구현
```kotlin
interface JSONFactory<T> {
    fun fromJSON(text: String): T
}

class PersonJson(val name: String) {
    companion object: JSONFactory<PersonJson> {
        override fun fromJSON(text: String): PersonJson {
            return PersonJson("aa")   
        }
    }
}
```

* 동반 객체 확장
  * 동반 객체도 확장이 가능하다. 단 반드시 동반객체가 선언되어 있어야 한다.
  * 동반 객체는 name을 지정하지 않으면 동반객체 선언한 class명 companion으로 사용할 수 있다.
```kotlin
class PersonObj(
    val firstNamd: String,
    val lastNamd: String
) {
    companion object {
        
    }
}

fun PersonObj.Companion.fromJSON(json: String) {
    println("companion obj")
}
```

* 객체 식 - 무명 내부 클래스를 다른 방식으로 작성
  * 무명 객체를 정의할 떄도 object를 사용한다.
  * 객체 식은 클래스를 정의하고 그 클래스에 속한 인스턴스를 생성하지만, 그 클래스나 인스턴스에 이름을 붙이지 않는다.
  * 보통 함수를 호출하면서 인자로 무명 객체를 넘기기 때문에 클래스와 인스턴스 모두 이름이 필요하지 않다.
  * 객체 선언과 달리 싱글턴이 아니다.
  * 흠.. 객체 식은 아직 잘 이해가 가지 않는다. 다만 java에서 함수형 interface와 비슷한 느낌으로 지금은 이해 한다.