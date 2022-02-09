## 3.1 코틀린에서 컬렉션 만들기

코틀린만의 컬렉션은 없고 자바의 컬렉션을 사용하며 더 많은 기능을 추가했다.

```kotlin
val set = hashSetOf(1,7,53)
val list = arrayListOf(1,7,53)
val map = hashMapOf(1 to "One", 7 to "seven") //to는 키워드가 아니라 일반 함수이다.

val strings = listOf("first", "second")
println(strings.last()) //second
    
val numbers = setOf(1,14,2)
println(numbers.maxOrNull()) //14
```

## 3.2 함수를 호출하기 쉽게 만들기

```kotlin
println(strings) //[first, second]
println(joinToString(strings, "; ", "(", ")")) //(first; second)

fun <T> joinToString(collections: Collection<T>, seperator: String, prefix: String, postfix: String): String {
    val result = StringBuilder(prefix)
    for((index, element) in collections.withIndex()) {
        if(index > 0) result.append(seperator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```

### 3.2.1 이름 붙인 인자

```kotlin
joinToString(collection, seperator = " ", prefix = " ", prefix = ".")
```

인자 중 어느 하나라도 이름을 명시하고 나면 뒤에 오는 모든 인자는 이름을 꼭 명시해야 한다.

### 3.2.2 디폴트 파라미터 값

자바에서는 오버로딩한 메소드가 많아진다는 문제가 있지만 코틀린은 파라미터의 디폴트 값을 지정해서 오버로드를 피할 수 있다.

지정하고 싶은 인자를 이름붙여서 순서와 관계없이 지정할 수 있다.

자바에서 코틀린을 호출하는 경우에는 모든 인자를 명시하거나 @JvmOverloads 어노테이션을 추가하면 자바 컴파일러가 파라미터들을 다 오버로딩해서 생성해준다.

```kotlin
println(joinToString(strings))
println(joinToString(strings, "; "))

fun <T> joinToString(collections: Collection<T>, 
                     seperator: String = ", ", 
                     prefix: String = "", 
                     postfix: String = ""): String {
    val result = StringBuilder(prefix)
    for((index, element) in collections.withIndex()) {
        if(index > 0) result.append(seperator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```

### 3.2.3 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티

자바는 메소드를 클래스 안에 작성해야하기 때문에 JDK 의 Collections Util 클래스처럼 정적 메소드를 모아놓고 특별한 상태를 갖지 않는 경우가 생기는데 코틀린은 그럴 경우 소스 파일의 최 상위 수준에 위치시킬 수 있다.

정적 함수이기 때문에 함수가 정의된 패키지를 임포트해야 한다.

```kotlin
package strings
fun joinToString(~): {~}
```

JVM은 클래스 안의 코드만을 실행할 수 있기 때문에 컴파일러는 새로운 클래스를 정의해준다. 

```java
package string;
public class JoinKt {  //코틀린 소스 파일의 이름과 대응한다. 
	public static String joinToString(~) {~}
}

//클래스 이름을 바꾸고 싶은 경우
@file:JvmName("StringFunctions") //클래스 이름 지정
package string //어노테이션 뒤에 패키지 문이 와야한다.
fun joinToString(~) : String{~}
```

자바에서 joinToString을 아래와 같이 호출할 수 있다.

```java
import strings.JoinKt;
JoinKt.joinToString(~);
```

#### 최상위 프로퍼티 

프로퍼티도 파일의 최상위 수준에 놓을 수 있다. 최상위도 접근자 메소드가 생기는데 상수를 접근자 메소드로 조회하는 것은 자연스럽지 못하다. const val 로 선언하면 public static final 로 컴파일된다.

```kotlin
var opCount = 0//최상위에 프로퍼티 선언
val UNIX_LINE_SEPERATOR = "\n" //최상위에 상수 추가
//-> 변경
const val UNIX_LINE_SEPERATOR = "\n"
fun performOperation() {
	opCount++ //최상위 프로퍼티 변경 
	Println("Operation performed $opCount times") //최상위 프로퍼티 조회
}
```

## 3.3 메소드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티

확장 함수는 어떤 클래스의 멤버 메소드인 것처럼 호출할 수 있지만 그 클래스 밖에 선언된 함수다.

확장 클래스 이름 뒤에 함수를 작성한다.

```kotlin
package strings
fun String.lastChar(): Char = this.get(this.length - 1)
//String: 수신 객체 타입(receiver type)
//this : 수신 객체 (receiver object). 생략 가능 
println("Kotlin".lastChar()) //n
//String : 수신 객체 타입
//"Kotlin" : 수신 객체 
fun String.lastChar(): Char = get(length - 1)
```

단, 클래스 내부에서만 사용할 수 있는 private member나 protected member를 사용할 수 없다. 

### 3.3.1 임포트와 확장 함수

확장 함수는 임포트해야 사용할 수 있다. as 키워드로 임포트한 대상을 다른 이름으로 부를 수 있다. 이름이 동일한 경우 혼란을 방지할 수 있다. 하지만 코틀린 문법 상 반드시 짧은 이름을 사용해야 한다.

### 3.3.2 자바에서 확장 함수 호출

수신 객체를 첫 번째 인자로 받는 정적 메소드로, 호출 시 부가 비용이 발생하지 않는다. 

### 3.3.3 확장 함수로 유틸리티 함수 정의

클래스의 멤버인 것 처럼 joinToString을 호출한다. 정적 메소드 호출에 대한 **syntatic sugar** 

```kotlin
println(listOf(1,2,3).joinToString(separator= "; ", prefix = "(", postfix=")"))

fun <T> Collection<T>.joinToString(
separator: String = ", ",
prefix: String = "",
postfix: String=""): String {
    val result = StringBuilder(prefix)
    for((index, element) in this.withIndex()) {
    	if(index > 0) result.append(separator)
    result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
//(1; 2; 3)
```

```kotlin
fun Collection<String>.join(~) = joinToString(~)
//문자열 컬렉션에서만 호출할 수 있다.
```

확장 함수는 정적 메소드와 같은 특징을 갖기 때문에 하위 클래스에서 오버라이드할 수 없다.

### 3.3.3 확장 함수는 오버라이드할 수 없다.

일반적으로는 함수를 오버라이드할 수 있지만 확장 함수는 클래스 밖에 선언되며 수신 객체로 지정한 변수의 정적 타입에 의해 어떤 확장 함수가 호출될 지 결정되기 때문에 오버라이드할 수 없다.

```kotlin
open class View {
    
}
class Button: View() {
    
}
fun View.showOff() = println("view")
fun Button.showOff() = println("button")

val view : View = Button()
view.showOff() //view
```

어떤 클래스를 확장한 함수와, 그 클래스의 멤버 함수의 이름과 시그니처가 같다면 멤버 함수가 호출된다.

멤버 함수의 우선 순위가 확장 함수보다 높다.

### 3.3.5 확장 프로퍼티 (?)

확장 프로퍼티는 아무 상태도 가질 수 없지만 프로퍼티 문법으로 코드를 짧게 작성할 수 있다.

일반 프로퍼티에 수신 객체 클래스가 추가된 형태이다. 뒷받침하는 필드가 없어서 기본 게터가 없어서 최소한의 게터는 정의해야한다. 상태를 가질 수 없기 때문에 초기화 코드도 사용할 수 없다.

```kotlin
println("kotlin".lastChar2) //n

val String.lastChar2: Char	
	get() = get(length - 1)
```

```kotlin
var sb = StringBuilder("kotlin?")
    sb.lastChar = '!'
    println(sb)
}
var StringBuilder.lastChar: Char 
	get() = get(length - 1)
    set(value: Char) {
        this.setCharAt(length - 1, value)
}
```

```java
StringUtilKt.getLastCHar("java")
//자바에서 확장 프로퍼티를 사용하고 싶다면 게터나 세터를 명시적으로 호출해야한다.
```

## 3.4 컬렉션 처리 : 가변 길이 인자, 중위 함수 호출, 라이브러리 지원

- vararg 로 인자 개수가 달라질 수 있는 함수를 정의
- 중위 함수 호출을 사용 ([https://kotlinlang.org/docs/functions.html#infix-notation](https://kotlinlang.org/docs/functions.html#infix-notation))

```kotlin
infix fun Int.shl(x: Int): Int { ... }

// calling the function using the infix notation
1 shl 2

// is the same as
1.shl(2)
```

- 구조 분해 선언을 사용해서 여러 변수에 나눠 담을 수 있다. ([https://kotlinlang.org/docs/destructuring-declarations.html](https://kotlinlang.org/docs/destructuring-declarations.html))

```kotlin
val (name, age) = person
println(name)
println(age)
```

### 3.4.2 가변 인자 함수 : 인자의 개수가 달라질 수 있는 함수 정의

```kotlin
val list = listOf(2,3,5,7,11)
fun listOf<T>(vararg values: T): List<T> { ~ }
```

스프레드 연산자(spread) 를 사용해서 배열을 명시적으로 풀어 인자로 전달할 수 있다.

```kotlin
val list = listOf(*args)
```

### 3.4.3 값의 쌍 다루기: 중위 호출과 구조 분해 선언

```kotlin
val map = mapOf(1 to "one", 7 to "seven")
```

수신 객체와 유일한 메소드 인자 사이에 메소드 이름을 넣는다.(단 공백이 들어가야한다)

```kotlin
1.to("one)
1 to "one"
```

함수를 중위호출에 사용하고 싶으면 infix 변경자를 함수 선언 앞에 추가해주면 된다.

```kotlin
infix fun Any.to(other: Any) = Pair(this, other)
```

```
! **제네릭 함수** : 제네릭을 인자로 받는 함수 
```

to의 Pair는 두 변수를 **구조 분해 선언**으로 아래와 같이 표현할 수 있다. to 를 이용해 순서쌍을 만든 다음 구조 분해를 통해 순서쌍을 푼다.

```kotlin
val (number, name) = 1 to "one"
```

## 3.5 문자열과 정규식 다루기

### 3.5.1 문자열 나누기

자바의 split은 정규표현식을 담는데, 코틀린의 split 은 구분문자, 정규표현식을 받는 함수를 제공해준다.

```kotlin
"12.345-6.A".split("\\.|-".toRegex())
"12.345-6.A".split(".", "-")
```

### 3.5.2 정규식과 3중 따옴표로 묶은 문자열

코틀린은 정규식을 사용하지 않고도 문자열을 쉽게 파싱할 수 있다.

```kotlin
fun parsePath(path: String) {
    val directory = path.substringBeforeLast("/")
    val fullName = path.substringAfterLast("/")
    val fileName = fullName.substringBeforeLast(".")
    val extension = fullName.substringAfterLast(".")
    println("$directory $fullName $fileName $extension")
}
```

3중 따옴표로 정규식일 쓰면 역슬래시를 포함한 어떤 문자도 이스케이프할 필요가 없다.

```kotlin
fun parsePathForRegex(path: String) {
    val regex = """(.+)/(.+)\.(.+)""".toRegex()
    val mathResult = regex.matchEntire(path)
    val (directory, filename, extension) = mathResult!!.destructured
    println("$directory $filename $extension")
}
```

### 3.5.3 여러 줄 3중 따옴표 문자열

3중 따옴표는 이스케이프 피하는 용도와, 줄 바꿈을 표현하는 아무 문자열이나 그대로 들어간다. 

단, 이스케이프가 안되기 때문에 $를 넣어야 한다면 '$' 문자를 넣어야한다. 줄바꿈 \n 을 넣을 수 없다.

```kotlin
println("""enter
    enter""")
println("""enter\n enter""")
//enter
//    enter
//enter\n enter

println("""$path""")
println("""'$'path""")
// /a/b/c/v.doc
// '$'path
```

## 3.6 코드 다듬기: 로컬 함수와 확장

### 로컬 함수로 코드 중복 제거하기

```kotlin
class User(val id: Int, val name: String, val address: String) {
	fun saveUser(user: User) {
		fun validate(user: User, value: String, fieldName: String) {
			if(value.isEmpty()) throw Excepion
		}

		validate(user, user.name, "Name")
		validate(user, user.address, "Address")
	}

}
```

로컬 함수는 자신이 속한 바깐 함수의 모든 파라미터와 변수를 사용할 수 있다. 

검증 로직은 User를 사용하는 다른 곳에서는 안쓰이기 때문에 User에 포함시키고 싶지 않을 때, 한 객체만을 다루면서 객체의 비공개 데이터를 다룰 필요 없는 경우에 확장함수를 써서 클래스를 간결하게 유지할 수 있다.

일반적으로 한 단계만 함수를 중첩시키는 것을 권장한다.

```kotlin
class User(val id: Int, val name: String, val address: String)
fun User.validateBeforeSave() {
	fun validate~
}
validate(name, "Name")

fun saveUser(user: User) {
	user.validateBeforeSave()
}
```

## 3.7 요약
1. 코틀린은 자체 컬렉션 클래스를 정의하지 않지만 자바 클래스를 확장해서 더 풍부한 API를 제공한다.
2. 함수 파라미터 디폴트 값을 정의하면 오버로딩한 함수를 정의할 필요성이 줄어든다. 이름 붙인 인자를 사용하면 함수의 인자가 많을 떄 함수 호출의 가독성을 더 향상시킬 수 있다.
3. 코틀린 파일에서 클래스 멤버가 아닌 최상위 함수와 프로퍼티를 직접 선언할 수 있다. 이를 활용하면 코드 구조를 더 유연하게 만들 수 있다.
4. 확장 함수와 프로퍼티를 사용하면 외부 라이브러리에 정의된 클래스를 포함해 모든 클래스의 API를 그 클래스의 소스코드를 바꿀 필요 없이 확장할 수 있다. 확장 함수를 사용해도 실행 시점에 부가 비용이 들지 않는다.
5. 중위 호출을 통해 인자가 하나 밖에 없는 메소드나 확장 함수를 더 깔끔한 구문으로 호출할 수 있다.
6. 코틀린은 정규식과 일반 문자열을 처리할 때 유용한 다양한 문자열 처리 함수를 제공한다.
7. 자바 문자열로 표현하려면 수많은 이스케이프가 필요한 문자열의 경우 3중 따옴표 문자열을 사용하면 더 깔끔하게 표현할 수 있다.
