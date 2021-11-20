# 연산자 오버로딩과 기타 관례
* 연산자 오버로딩
* 관례: 여러 연산을 지원하기 위해 특별한 이름이 붙은 메서드
* 위임 프로퍼티
* 어떤 언어 기능과 미리 정해진 이름의 함수를 연결해주는 기법을 kotlin에서는 관례라 한다.
* java와 같이 type에 의존하지 않고 kotlin은 함수 이름을 통한 관례에 의존 한다.

## 산술 연산자 오버로딩
### 이항 산술 연산 오버로딩
* 연산자를 오버로딩하는 함수에는 operator를 명시해서 관례를 따르는 함수임를 표기해야 한다.
* opertator를 안하는 경우 exception이 발생한다.
* 확장함수로 연산자를 정의할 수 있다.
* kotlin은 언어에서 미리 정해둔 연산자만 오버로딩할 수 있다.
* 관례에 따르기 위해 클래스에서 정의해야 하는 이름이 연산자 별로 정해졌다.

|연산자|method|
|-----|------|
|*|times|
|/|div|
|%|mod or rem|
|+|plus|
|-|minus|
* 연산자의 우선 순위도 동일하게 적용된다.(*, /, %)는 (+, -)보다 우선 순위가 높다.
* 연산자를 정의할때 꼭 두 피연산와 타입이 같을 필요는 없다.
* 연산자가 자동으로 교환법칙을 지원 안한다.
* 연산자의 반환 타입은 두 피연산자 중 하나가 아니어도 된다.
* operator도 일반 함수와 같이 오버로딩 할 수 있다.
```kotlin
data class Point(val x: Int, val y: Int) {
    
    operator fun plus(other: Point): Point {
        return Point(x+other.x, y+other.y)
    }

    //param은 반드시 1개여야 한다.
/*    operator fun plus(x: Int, y: Int): Point {
        return Point(x+other.x, y+other.y)
    }*/

    operator fun minus(other: Point): Point {
        return Point(x-other.x, y-other.y)
    }

    operator fun times(other: Point): Point {
        return Point(x*other.x, y*other.y)
    }

    operator fun plus(double: Double): Point {
        return Point(x+double.toInt(), y+double.toInt())
    }

}

fun main() {
    val p1 = Point(1, 1)
    val p2 = Point(1, 1)
    println(p1+p2)
    println(p1-p1)
    println(p1*p1)
}
```


### 복합 대입 연산자 오버로딩
* plus와 같은 산술 연산자를 구현하면 자동으로 복합 대입 연산자도 지원한다.(+=, -= 등)
* 복합 대입 연산자를 사용하면 새로운 참조가 된다.
* 복합 대입 연산자를 통해 새로운 참조가 아닌 내부 변경도 가능하다.
* 예시로는 컬렉션이다. 산술연산을 써도 새로운 참조가 아닌 내부 변화가 이루어 진다.
* 반환 type이 unit인 plusAssign를 정의하면 가능하다. 비슷하게 minusAssign과 같이 사용된다.
* kotlin에서 변경가능 컬렉션은 Assign을 정의되어 있다.
* tip으로는 plus, plusAssign은 동시에 class에 정의하면 에러난다. 권장은 변경 불가능은 plus를 변경가능은 plusAssign을 권장 한다.
```kotlin
    operator fun plus(double: Double): Point {
        return Point(x+double.toInt(), y+double.toInt())
    }

    //이미 plus가 정의 되어 있어 에러 난다.
//    operator fun plusAssign(a: Point): Point {
//        return Point(0, 0)
//    }

fun main() {
    val p1 = Point(1, 1)
    var p3 = Point(3, 3)
    //단 참조하는 변수가 변경 된다.
    p3 += p1
    println(p3)//Point(x=4, y=4)

    val list = arrayListOf(1, 2)
    list += 3

    val list2 = listOf(4, 5)
    val newList = list + list2
    println(list)
    println(newList)
//    [1, 2, 3]
//    [1, 2, 3, 4, 5]
}
```

### 단항 연산자 오버로딩
* 이항 연산자와 같이 operator를 사용하고 미리 정해진 함수를 사용한다.
  |연산자|method|
  |-----|------|
  |+a|unaryplus|
  |-a|unaryminus|
  |++a, a++|inc|
  |--a, a--|dec|
  |!a|not|
* param은 취하지 않는다.
* 전위와 후위 증가/감소도 작용한다.
```kotlin
    operator fun unaryPlus(): Point {
        return Point(+x, +y)
    }
    
    operator fun inc(): Point {
        return Point(x+1, y+1)
    }
```

## 비교 연산자 오버로딩
### 동등성 연산자: equals
* '=='는 equals와 같다. 즉 관계를 적용한다.
* '!='도 equals를 호출 한다.
* '==', '!='는 null 인지도 검사한다.
  * a==b -> a?.equals(b)?:(b==null)
* data class는 자동으로 equals를 지원 한다.
* equals는 any에 속해있어 override가 붙는다.
* 상위 class에서 정의된 method를 override하기에 operator가 안붙는다.
* any의 equals를 보면 operator가 붙어 있다.
* equals는 확장함수 보다 우위라 확장함수로도 정의 불가능 하다.
```kotlin
class Point01(val x: Int, val y: Int) {
    override fun equals(other: Any?): Boolean {
        if (this === other) return true //최적화를 위해 식별자 비교 연산자를 사용한다.
        if (javaClass != other?.javaClass) return false

        other as Point01

        if (x != other.x) return false
        if (y != other.y) return false

        return true
    }

    override fun hashCode(): Int {
        var result = x
        result = 31 * result + y
        return result
    }
}
```

###순서 연산자: compareTo
* kotlin도 Comparable 인터페이스를 지원한다.
* Comparable안에 compareTo 메소드를 호출하는 관례를 제공한다.
* <, > <=, >= 는 compareTo 호출로 컴파일 된다.
* compareTo가 반환하는 값은 int다
* p1 < p2는 p1.compareTo(p2)와 같다.
```kotlin
data class Person(val firstName: String, val lastName: String) : Comparable<Person> {

    //Comparable에도 operator가 붙어 있어 여기서는 생략 된다.
    override fun compareTo(other: Person): Int {
        //kotlin 표준 라이브러리 compareValuesBy를 사용해 코드를 간결하게 한다.
        return compareValuesBy(this, other, Person::firstName, Person::lastName)
    }

}
```

## 컬렉션과 범위에 대해 쓸 수 있는 관례
* 컬렉션을 사용하면 인덱스를 통한 get, set, in 이다.
* 이런 연산도 관례를 지원 한다.
### 인덱스로 원소에 접근: get, set
* 우리가 사용하는 컬렉션 접근 방법과 같이 get, set에서 인덱스를 통한 관례도 작성 가능 하다.
* param은 고정되어 있지 않아 여러번 오버로딩을 통해 추가적인 구현이 가능하다.
```kotlin
operator fun Point.get(idx: Int): Int {
  return when(idx) {
    0 -> this.x
    1 -> this.y
    else -> throw IndexOutOfBoundsException("exception")
  }
}

operator fun Point.get(idx: Int, sault: Int): Int {
  return when(idx) {
    0 -> this.x + sault
    1 -> this.y + sault
    else -> throw IndexOutOfBoundsException("exception")
  }
}

operator fun Point01.set(idx: Int, value: Int) {
  when(idx) {
    0 -> x = value
    1 -> y = value
    else -> throw IndexOutOfBoundsException("exception")
  }
}
```

### in 관례
* 컬렉션이 지원하는 다른 연산자로는 in이 있다.
* in은 객체가 컬렉션에 들어있는지 검사한다.
* in 연산자와 대응하는 함수는 contains다.
```kotlin
data class Rectangle(val upperLeft: Point, val lowerRight: Point) {
    operator fun contains(other: Point): Boolean {
      //in의 우항이 수신객체이다.
        return other.x in upperLeft.x .. lowerRight.x && other.y in upperLeft.y .. lowerRight.y
    }
}
val rect = Rectangle(Point(10, 20), Point(50, 50))
println(Point(20, 30) in rect)
println(rect.contains(Point(5, 5)))
```

### rangeTo 관례
* 범위를 만들려면 '..' 구문을 사용한다.
* '..'연산자느 rangeTo 함수를 간략하게 표현하는 방법이다.
* start..end -> start.rangeTo(end)
* rangeTo는 어떤 클래스에서 정의 가능하지만 Comparable을 상속했다면 그냥 사용가능 하다.
```kotlin
val now = LocalDate.now()
val vacation = now..now.plusDays(10)

println(now.plusDays(4) in vacation)

(0..10).forEach{}
//0..10.forEach{} 이와 같이 사용하려면 ()가 있어야 한다. 이유는 우선 순위가 낮다.
```

### for 루프를 위한 iterator 관례
//흠... 이해가 조금 안된다.
* for(x in list) {...}는 사실 list.iterator()를 호출하고 원소를 loop돌리는 방식이다.
* 여기는 보충이 필요하다 나중에 다시 확인한다.

## 구조 분해 선언과 component 함수
* 내부에서 구조 분해 선언은 다시 관례를 사용한다.
* 구조 분해 선언의 각 변수를 초기화하기 위해 componetN이라는 함수를 호출한다.
* N은 구조 분해 선언에 있는 변수 위치에 따라 붙는 번호다.
* data 클래스의 주 생성자에 들어있는 프로퍼티에 대해서는 컴파일러가 자동으로 componentN 함수를 생성한다.
* 일반 class에는 component를 관례를 해서 사용해야 한다.
```kotlin
//일반 클래스에는 이와 같이 선언 해야 한다.
class Point02(val x: Int, val y: Int) {
    operator fun component1() = x
    operator fun component2() = y
}

data class NameComponents(val name: String, val extension: String)

fun splitFilename(fullName: String): NameComponents {
    val result = fullName.split(".", limit = 2)
    return NameComponents(result[0], result[1])
}

fun main() {
    val (name, ext) = splitFilename("abc.kt")
}
```

### 구조 분해 선언과 루프
* 함수 본문 내의 선언문뿐 아니라 변수 선언이 들어갈 수 있는 장소면 어디서든 구조 분해는 가능하다.
* 푸르 안에서도 구조 분해 선ㄱㄷ언을 사용할 수 있다.
```kotlin
fun main() {
    //map의 확장함수로 component1, component2를 제공 한다.
    val map = mapOf("1" to "a", "2" to "b")
    
    for((key, value) in map) {
        println("$key - $value")
    }
}
```