# scala, spark로 시작하기
- Spark는 In-memory 기반의 처리로 hadoop의 mapreduce보다 100배 빠른 처리를 제공하고,  
  머신러닝, 그래프, 라이브러리등 빅데이터 분석을 위한 통합 컴포넌트 제공
- Spark는 스칼라로 되어 있어 보다 간결한 코드로 작성이 가능하고, JVM 기반으로 작동해 기존에 사용하던 자바 라이브러리를 사용할 수 있음

## 스칼라? 
- 객체지향언어와 함수형언어를 동시에 지원하는 언어
- JVML 언어
- 스칼라는 스칼라 컴파일러를 통해 스칼라 코드를 바이트 코드로 변환하고 이 바이트 코드는 JVM 상에서 JAVA와 동일하게 돌아감

### 스칼라의 장점
- Java에 비해 코드가 간결함  
  getter, setter, 생성자를 제거하고, 표현식을 간소화함
- 바이트 코드를 최적화하여 Java 언어 대비 20%정도 빠름
- 스칼라는 변경 불가능한 Immutable 변수를 많이 가지고 있음  
  따라서 이를 통해 속성이 변경 불가능하게 하고, 순수 함수를 사용하여 병렬 처리를 가능하게 함

## 함수형 프로그래밍
- 함수형 프로그래밍은 프로그래밍 페러다임 중 하나로, 자료 처리를 수학적 함수의 계산으로 취급하고 상태 변화와 가변 데이터를 피하는 것
- 순수 함수와 보조 함수의 조합을 통해서 조건문과 반복문의 사용을 피하고 변수의 사용을 억제하여 상태 변경을 피하고자 하는 것
- 함수형 프로그래밍은 다음과 같은 특징을 이용함
  - 순수 함수(pure function)  
    함수의 실행이 외부의 영향을 끼치지 않는 함수  
    병렬 계산이 가능
  - 익명 함수(anonymous function)  
    익명함수를 사용하여 코드 길이를 줄이고, 프로그래밍 로직에 집중할 수 있음
  - 고차 함수(high-order function)  
    함수를 인수로 취하는 함수  
    함수를 입력 파라미터나 출력 값으로 처리할 수 있음
 
 ## 0. Hello world
 - 스칼라는 Java와 동일하게 진입점으로 `main()`함수를 구현해야 함
 - 이를 위해 `main()`를 구현하는 방법은 두가지가 있음
   - 첫 번째는 싱글톤 객체(object)가 main 함수를 구현하는 방법
   - 두 번째는 싱글톤 객체가 App 트레잇을 상속받는 방법

###  싱글톤 객체(object)가 main 함수를 구현하는 방법
~~~scala
// 싱글톤 오브젝트의 main 함수 구현
object S01_HelloWorldObject {
  def main(args: Array[String]): Unit = {
    println("Hello World main")
  }
}
~~~

### App 트레잇 상속
- App 트레잇을 상속하는 방법은 extends로 App 트레잇을 상속하고, 실행하고자 하는 코드를 작성하면 순차적으로 실행
~~~scala
objects S01_HelloWorldObject extends App {
     println("Hello World")
}
~~~

## 1. 객체
- 스칼라는 기본자료형, 함수, 클래스등 모든 것을 객체로 취급
- 객체는 Any를 최상위 클래스로 값(AnyVal) 타입과 참조(AnyRef) 타입을 상속하여 구현
- Int, Byte, String 같은 기본 자료형은 AnyVal을 상속하고, 클래스는 AnyRef를 상속

### 객체의 비교
- 객체의 비교는 `==`, `!=` 연산자 사용
- java 에서는 문자열의 비교는 `equals()`를 이용했지만 scala에서는 전부 ==와 !=로 통일
~~~scala
scala> 1 == 1
res15: Boolean = true

scala> 'a' == 'a'
res16: Boolean = true

scala> 1 == 'a'
res17: Boolean = false

// 문자형의 비교
val s1 = "Hello"
val s2 = "Hello"
val s3 = "HELLO"

scala> s1 == s2
res13: Boolean = true

scala> s2 == s3
res14: Boolean = false

// 클래스의 비교
case class Person( p1:String, p2:String )
val p1 = Person("a", "b")
val p2 = Person("a", "b")
val p3 = Person("b", "c")

scala> p1 == p2
res18: Boolean = true

scala> p2 == p3
res19: Boolean = false
~~~

## 2. 자료형
- Scala는 Java의 자료형과 동일함
- Scala는 원시 자료형이 존재하지 않고, 모든 것이 클래스 
- Scala의 자료형은 컴파일 시점에 자동으로 변환됨
- Scala의 자료형은 숫자형, 논리형, 문자형이 존재하고 명시적으로 변수의 타입을 선언할수도 있고, 그렇지 않을수도 있음
- 선언하지 않으면 자동으로 컴파일러가 정수형은 Int, 실수형은 Double로 변환

### 숫자형
- 숫자를 표현하는 자료형은 정수형, 실수형이 존재
  - 정수형  
   Byte, Short, Int, Long
  - 실수형  
    Float, Double
- 다운케스팅(Double -> Int)는 명시적으로 선언 필요
~~~scala
// 더 큰 크기의 값으로 변환하는 업캐스팅은 자동으로 지원 
scala> s = b
// 더 작은 크기의 값으로 변환하는 다운 캐스팅은 오류 발생 
scala> i = l
<console>:13: error: type mismatch;
 found   : Long
 required: Int
       i = l
           ^
// 메소드를 이용하여 명시적으로 변환 
scala> i = l.toInt
i: Int = 10
~~~

### 데이터 형 선언
- 암시적인 선언은 컴파일러가 자동으로 타입을 선택합니다. 데이터 타입에 맞춰 선언하시면 됨
~~~scala
// 암시적인 선언 
var x = 10
var y = "abc"

// 명시적인 선언 
var b: Byte = 10
var s: Short = 10
var i: Int = 10
var l: Long = 10

// 값에 약어를 추가하여 명시적 선언 
var f = 10.0f
var d = 20.0d

// 암시적인 선언, 컴파일러가 자동으로 타입을 선택 
scala> var ii = 10
ii: Int = 10

scala> var ff = 1.0
ff: Double = 1.0
~~~

### 논리형
- 논리형 `Boolean`은 조건문에서 분기를 처리할 때 주로 사용합니다. `true`, `false` 값이 존재합니다.
~~~scala
  var t = true

  if(t)
    println("참")
  else
    println("거짓")
~~~

### 문자형
- 문자형 `Char`은 하나의 문자를 표현 할 수 있음
~~~scala
scala> var c1:Char = 'a'
c1: Char = a

scala> var c2 = 'b'
c2: Char = b
~~~

## 3. 문자열
- 스칼라에서 문자열의 표현은 쌍따옴표(")를 이용하여 처리
~~~scala
scala> val str1 = "aaa"
str1: String = aaa
~~~
- 멀티라인 문자열은 세개의 쌍따옴표(""")를 이용하여 생성
~~~scala
scala> val str2 = """a
     | b
     | c"""
str2: String =
a
b
c
~~~

### 접두어를 이용한 문자열 처리
- 스칼라 2.10.0 부터 문자열에 접두어(id)를 붙여서 컴파일시점에 문자열 변환을 처리하게 할 수 있음
- 기본적으로 제공하는 접두어는 `s`, `f`, `raw` 세가지가 있습니다. 사용자가 접두어를 생성할 수도 있음

#### 접두어s
- 접두어s는 `${변수명}`을 이용하여 문자열안의 변수를 값으로 치환하여 줌. 계산식, 함수도 사용할 수 있음
~~~scala
// 문자열 치환 
val name = "David"

// ${name}이 David로 변환 
scala> println(s"Hello! ${name}")
Hello! David

// 계산 값 치환 안됨 
scala> println("${ 1 + 1 }")
${ 1 + 1 }

// s 접두어가 있으면 계산식 처리 
scala> println(s"${ 1 + 1 }")
~~~

#### 접두어 f
- 접두어f는 문자열 포맷팅을 처리. 
- 자바의 printf() 와 같은 방식으로 처리되고, 타입이 맞지 않으면 오류가 발생
~~~scala
val height:Double = 182.3 
val name = "James"

// f접두어를 이용한 값 변환 테스트 
scala> println(f"$name%s is $height%2.2f meters tall")
James is 182.30 meters tall
~~~

#### 접두어 raw
- 접두어raw은 특수 문자를 처리하지 않고 원본 문자로 인식합니다. 특수문자를 그대로 입력해야 할 때 사용할 수 있음
~~~scala
// \n으로 개행 처리 
scala> s"가\n나"
res1: String =
가
나

// \n을 문자 그대로 인식 
scala> raw"가\n나"
res3: String = 가\n나
~~~

## 4. 변수
- 스칼라의 변수는 두가지 유형이 있습니다. 가변 변수와 불변 변수 
- 가변 변수(variable)는 `var`, 불변 변수(value)는 `val`로 선언합니다.
- 가변 변수는 재할당이 가능하지만 불변 변수는 재할당이 불가능
- 불변 변수는 한번 값이 정해지면 변경되지 않기(immutable) 때문에 데이터 처리에 있어 단순하게 처리할 수 있는 장점이 있음
~~~scala
var variable = 10
val value = 20

// var 값의 재할당은 처리 가능 
scala> variable = 30
variable: Int = 30

/// value 값의 재할당은 오류 발생 
scala> value = 40
<console>:12: error: reassignment to val
       value = 40
             ^
~~~
- 스칼라는 동시처리를 중요하게 생각하기 때문에 변경 불가능한 데이터를 중요하게 생각함 
- 데이터의 변경이 가능하면 동시처리시에 데이터에 대한 고민이 많아지게 됨. 
- 따라서 불변 변수로 설정하는 것을 선호

## 5. 함수
- 함수는 `def`로 선언
- 함수를 선언할 때 리턴문과 리턴 타입은 생략이 가능
- 매개변수의 파라미터 타입은 생략할 수 없음
- 리턴값이 없는 함수를 선언할 때는 `Unit`을 이용
- 함수의 매개변수는 불변 변수이기 때문에 재할당 할 수 없음
- 리턴 타입을 생략하면 컴파일러가 반환값을 이용하여 자동으로 추론
- 리턴문이 생략되고, 리턴 타입이 Unit이 아니면 함수의 마지막 값을 리턴
~~~scala
// 함수 선언 
def add(x: Int, y: Int): Int = {
  return x + y
}

// x는 val 이기 때문에 변경 불가 
def add(x: Int): Int = {
  x = 10 
}

// 리턴 타입 생략 가능 
def add(x: Int, y: Double) = {
  x + y
}

// 리턴 타입이 Unit 타입도 생략 가능 
def add(x: Int, y: Int) = {
  println(x + y)
}

// 리턴 데이터가 없는 경우 Unit을 선언  
def add(x: Int, y: Int): Unit = {
  println(x + y)
}
~~~

### 함수의 축약형
- 함수가 1라인으로 처리 가능한 경우에는 중괄호({}) 없이 선언할 수도 있음
~~~scala
// 중괄호 없이 선언 
def printUpper(message:String):Unit = println(message.toUpperCase())

// 반환타입도 생략 
def printLower(message:String) = println(message.toLowerCase())
~~~

### 파라미터의 기본값
~~~scala
// y에 기본값 선언 
def add(x: Int, y: Int = 10): Unit = {
  println(x + y)
}

// 기본값 이용
scala> add(1)
11

// 전달값 이용 
scala> add(10, 3)
13
~~~

### 가변 길이 파라미터
- 같은 타입의 개수가 다른 가변 길이 파라미터를 입력할 경우 *를 이용하면 `Seq`형으로 변환되어 입력됨
~~~scala
// 여러개의 Int 형을 입력받아서 합계 계산 
def sum(num:Int*) = num.reduce(_ + _)

scala> sum(1, 2, 3)
res22: Int = 6

scala> sum(1)
res23: Int = 1
~~~

### 변수에 함수 결과 할당
- 함수를 `def`로 선언하지 않고 `var`, `val`로 선언할 수도 있습니다. 이렇게 하면 함수를 실행하여 그 반환값이 변수에 입력됩니다. 따라서 함수의 결과를 여러곳에서 이용할 때만 사용하는 것이 좋습니다.
~~~scala
val random1 = Math.random()
var random2 = Math.random()
def random3 = Math.random()

// random1, random2는 호출할 때마다 같은 값 반환
// 선언 시점에 값이 확정됨 
scala> random1
res19: Double = 0.1896318278308372
scala> random1
res20: Double = 0.1896318278308372

scala> random2
res21: Double = 0.817421180386978
scala> random2
res22: Double = 0.817421180386978

// random3은 실행시점에 값이 확정됨 
scala> random3
res24: Double = 0.4491518929189594
scala> random3
res25: Double = 0.7644113222566244
~~~

### 함수의 중첩
- 함수 안에 함수를 중첩하여 선언하고 사용할 수 있음
~~~scala
  def run() {
    def middle() {
      println("middle")
    }

    println("start")
    middle()
    println("end")
  }

scala> run
start
middle
end
~~~

## 6. 람다 함수
- 스칼라는 익명의 람다 함수를 선언할 수 있습니다.
- 람다함수는 언더바 _를 이용하여 묵시적인 파라미터를 지정할 수 있음. 
- 묵시적인 파라미터를 이용할 때는 언더바의 위치에 따라 파라미터가 선택됨
~~~scala
// exec는 3개의 파라미터(함수 f, x, y)를 받음
def exec(f: (Int, Int) => Int, x: Int, y: Int) = f(x, y)

// 람다 함수를 전달하여 처리. x+y 작업을 하는 함수 전달 
scala> exec((x: Int, y: Int) => x + y, 2, 3)
res12: Int = 5

// 선언시에 타입을 입력해서 추가적인 설정 없이 처리 가능 
scala> exec((x, y) => x + y, 7, 3)
res13: Int = 10

// 함수에 따라 다른 처리 
scala> exec((x, y) => x - y, 7, 3)
res14: Int = 4

// 언더바를 이용하여 묵시적인 처리도 가능 
scala> exec(_ + _, 3, 1)
res15: Int = 4
~~~

## 7. 커링
- 스칼라에서 커링은 여러 개의 인수 목록을 여러 개의 괄호로 정의할 수 있습니다. 함수를 정해진 인수의 수보다 적은 인수로 호출하면 그 리턴 값은 나머지 인수를 받는 함수
~~~scala
// x를 n으로 나누어 나머지가 0인지 확인하는 함수 
def modN(n:Int, x:Int) = ((x % n) == 0)     // 1번
def modN(n: Int)(x: Int) = ((x % n) == 0)   // 2번
~~~
- 위의 예제에서 함수의 파라미터를 표현하는 방법은 다르지만 같은 함수
- 1번의 n, x 인수를 2번에서는 여러개의 괄호로 받은 것
- 2번 함수는 커링을 이용해서 n 값을 미리 바인딩 하는 다른 함수로 선언하거나, 다른 함수의 파라미터로 전달 가능
~~~scala
def modN(n: Int)(x: Int) = ((x % n) == 0)

// modN함수를 커링을 이용하여 n 값이 정해진 변수로 호출 
def modOne:Int => Boolean = modN(1)
def modTwo = modN(2) _

println(modOne(4))  // true
println(modTwo(4))  // true
println(modTwo(5))  // false
~~~
- 커링을 이용한 스칼라의 샘플예제를 살펴보면 다음과 같음
~~~scala
object CurryTest extends App {
  // 재귀를 이용하여 xs를 처음부터 끝까지 순환하는 filter
  // 리스트를 head, tail로 구분하여 시작부터 끝까지 순환 
  // head : list의 첫 번째 값 리턴
  // tail : list의 첫 번째 값을 제외한 나머지 list 리턴
  def filter(xs: List[Int], p: Int => Boolean): List[Int] =
    if (xs.isEmpty) xs
    else if (p(xs.head)) xs.head :: filter(xs.tail, p)  // :: 는 리스트를 연결하는 함수 
    else filter(xs.tail, p)

  def modN(n: Int)(x: Int) = ((x % n) == 0)
  val nums = List(1, 2, 3, 4, 5, 6, 7, 8)

  println(filter(nums, modN(2)))    // List(2, 4, 6, 8)
  println(filter(nums, modN(3)))    // List(3, 6)
}
~~~

## 8. 클로저
- 함수형 언어에서 클로저는 내부에 참조되는 모든 인수에 대한 묵시적 바인딩을 지닌 함수
- 이 함수는 자신이 참조하는 것들의 문맥을 포함
- 클로저는 지연 실행의 좋은 예
- 클로저 블록에 코드를 바인딩함으로써 그 블록의 실행을 나중으로 연기할 수 있습니다. 
- 예를 들어 클로저 블록을 정의할 때는 필요한 값이나 함수가 스코프에 없지만, 나중에 실행시에는 있을 수가 있음
- 실행 문맥을 클로저 내에 포장하면 적절한 때까지 기다렸다가 실행할 수 있음
~~~scala
// divide는 x/n인 함수를 반환 
def divide(n:Int) = (x:Int) => {
  x/n
}

// n에 5를 바인딩, 호출될 때마다 사용
def divideFive = divide(5)
println(divideFive(10)) // 2

// 함수 외부의 값 factor를 바인딩 
var factor = 10
def multiplier = (x:Int) => x * factor
println(multiplier(4))  // 40

factor = 100
println(multiplier(4))  // 400
~~~

## 9. 함수 타입
- 스칼라에서 타입을 이용해서 클래스와 함수를 제네릭하게 생성

### 클래스 적용
- 다음의 예제는 클래스에 타입을 적용하여 제네릭하게 데이터를 처리하는 예제
- 스칼라는 기본적으로 불변(immutable) 데이터를 이용하기 때문에 변경가능한 스택처리를 위해 import 문을 추가
~~~scala
import scala.collection.mutable

trait TestStack[T] {
  def pop():T
  def push(value:T)
}

class StackSample[T] extends TestStack[T] {

  val stack = new scala.collection.mutable.Stack[T]

  override def pop(): T = {
    stack.pop()
  }

  override def push(value:T) = {
    stack.push(value)
  }
}

val s = new StackSample[String]
~~~

## 10. 클래스
- 클래스는 class를 이용하여 생성 할 수 있음
- 클래스를 선언할 때는 멤버 변수를 선언 가능
- 또한 멤버 변수 생략도 가능 
~~~scala
// 클래스 선언 
class Person(name:String, age:Int)
// 클래스 생성 
val p = new Person("David", 30)

// 멤버 변수 생략 가능 
class A
~~~  

### 클래스 멤버변수
- 멤버 변수를 선언할 때 가변 변수와 불변 변수를 명시적으로 선언할 수 있습니다
- 가변 변수는 컴파일러가 클래스 내부에 자동으로 getter, setter 메소드를 생성
- 가변 변수로 선언된 값은 읽고, 쓰는 것이 가능합니다. 불변 변수는 컴파일러가 getter만 생성합니다. 불변 변수로 선언된 값은 읽는 것만 가능
- 가변 변수, 불변 변수로 선언되지 않은 변수는 getter, setter 가 생성되지 않기 때문에 클래스 내부에서만 사용
~~~scala
// 기본형 
class Animal(name: String) {
  println(s"${name} 생성")
}

// 가변
class Dog(var name: String) {
  println(s"${name} 생성")
}

// 불변 
class Cat(val name: String) {
  println(s"${name} 생성")
}

var ani = new Animal("동물")
var dog = new Dog("개")
var cat = new Cat("고양이")

// 기본 멤버 변수는 접근 불가 
scala> println(ani.name)
<console>:27: error: value name is not a member of Animal
       println(ani.name)
                   ^

scala> println(dog.name)
개

scala> println(cat.name)
고양이

scala> dog.name = "바둑이"
dog.name: String = 바둑이

// 불변 변수는 재할당 불가 
scala> cat.name = "나비"
<console>:26: error: reassignment to val
       cat.name = "나비"
~~~

### 멤버변수의 디폴트 값
- 클래스 멤버변수는 변수의 종류에 상관없이 디폴트 값을 입력할 수 있습니다. 멤버변수의 값을 입력하지 않으면 디폴트 값을 이용
~~~scala
// 기본 멤버 변수, 변수, 상수 모두 기본값 설정 가능 
class Person1(name:String, age:Int)
class Person2(var name:String, var age:Int = 10)
class Person3(val name:String="Ted", val age:Int)

var p1 = new Person1("David", 12)
var p2 = new Person2("Tomphson")
var p3 = new Person3("David", 12)

// name 에 기본값을 입력할 수 없어서 오류 발생 
var p3 = new Person3(12)  
scala> var p3 = new Person3(12)
<console>:13: error: not enough arguments for constructor Person3: (name: String, age: Int)Person3.
Unspecified value parameter age.
       var p3 = new Person3(12)
~~~

### 클래스의 메소드
- 클래스의 메소드는 함수의 선언과 동일하게 `def`로 선언
~~~scala
class Person(name:String, age:Int) {
    def greeting() = println(s"${name}님은 ${age}살 입니다.")
}
~~~

### 메소드 오버라이드(override)
- `override` 선언자를 이용하여 메소드를 오버라이드 할 수 있음
- `new`를 이용하여 클래스 생성시에 오버라이드하여 메소드를 재정의 할 수 있음
~~~scala
class Person(name:String, age:Int, val job:String) {
    def greeting() = println(s"${name}님은 ${age}살 입니다.")
    def work() = println(s"직업은 ${job}입니다. ")
}

// work 함수 오버라이딩 
class Writer(name:String, age:Int) extends Person(name, age, "") {
    override def work() = println(s"직업은 작가입니다.")
}

val w = new Writer(name = "David", age = 15)

scala> w.greeting
David님은 15살 입니다.

scala> w.work
직업은 작가입니다.

// 클래스 생성시 메소드 오버라이드. public 변수를 사용하는 메소드만 오버라이드 가능 
val p = new Person("David", 15, "학생") {
  override def work() = println(s"job is ${job}. ")
}

scala> p.greeting
David님은 15살 입니다.

scala> p.work
job is 학생. 
~~~

### 생성자
- 스칼라는 따로 생성자가 존재하지 않음. 
- 클래스 바디부분에 있는 코드가 바로 실행되기 때문에 이 부분에 처리 로직을 넣어주면 됨.
~~~scala
class Person(name:String, age:Int) {
    def greeting() = println(s"${name}님은 ${age}살 입니다.")
    println("초기화 완료")
}

scala> val p = new Person("David", 15)
초기화 완료
p: Person = Person@522c2a19
~~~

### 상속과 추상 클래스
- 상속은 `extends`를 이용합니다. 일반 클래스와 추상클래스 모두 상속할 수 있습니다.
- 추상클래스는 `abstract`를 이용합니다. 추상클래스는 매개변수를 가질 수 있습니다. 
- 메소드를 선언만 하고 구현은 자식 클래스에 맡길 수 도 있고 기본 메소드를 구현할 수도 있습니다.
~~~scala
// 추상클래스 선언 
abstract class Person(name: String, age: Int) {
  def work
  def status(str: String)
  def greeting() = println(s"${name}님은 ${age}살 입니다.")
}

// Person 추상 클래스를 상속 
class Player(name: String, age: Int) extends Person(name, age) {
  def work = { println("일합니다.") }
  def status(str: String) = println(s"$str 상태 입니다.")
}

scala> var p = new Player("칼", 30)
scala> p.work
일합니다.

scala> p.status("깨있는")
깨있는 상태 입니다.
~~~

### 봉인 클래스(Sealed class)
- 스칼라는 봉인 클래스를 지원
- 봉인 클래스는 하위 타입이 모두 한파일에 있어야 함
- 관련 클래스를 한파일에 모두 입력하게 강제할 수 있기 때문에 관리의 효율성이 높아짐
-  봉인 클래스는 `sealed`를 이용하고 트레잇도 봉인할 수 있음
- 다음과 같이 `file1.scala`에 다음과 같이 봉인 추상클래스 `Furniture`를 생성하고, `file2.scala`에서 `Desk` 클래스를 선언하면 "illegal inheritance from sealed class Furniture" 오류가 발생합니다.
~~~scala
// file1.scala
sealed abstract class Furniture
case class Couch() extends Furniture
case class Chair() extends Furniture

// file2.scala
case class Desk() extends Furniture
  -> illegal inheritance from sealed class Furniture
~~~

## 11. 케이스 클래스
- 스칼라에는 케이스 클래스라는 특수 클래스가 존재
- `case`를 이용하여 선언
- 일반 클래스와 달리 인스턴스를 생성할 때 new를 사용하지 않음
~~~scala
// 케이스 클래스 Person 선언 
case class Person(name:String, age:Int)
var p = Person("철수", 15)
~~~

### 특징 1. 불변 데이터
- 케이스 클래스는 불변 데이터
- 케이스 클래스의 멤버변수는 기본적으로 불변 변수로 선언
~~~scala
case class Person(name:String, age:Int)

var p = Person("A", 10)

scala> p.name
res6: String = A

// 케이스 클래스의 데이터는 불변 변수로 자동 선언 
scala> p.name = "B"
<console>:27: error: reassignment to val
       p.name = "B"
~~~

### 특징 2. 패턴 매칭
- 스칼라의 패턴매칭은 자바의 case문과 유사하지만 더 강력한 기능을 제공

### 특징 3. 데이터 비교
- 케이스 클래스의 비교는 참조값을 이용하지 않고, 클래스의 멤버변수의 데이터를 이용하여 처리
~~~scala
var p1 = Person("A", 10)
var p2 = Person("A", 10)
var p3 = Person("B", 10)

scala> p1 == p2
res7: Boolean = true

scala> p2 == p3
res8: Boolean = false
~~~

### 특징 4. 초기화가 간단
- 케이스 클래스는 `new`를 이용하지 않고 초기화 할 수 있습니다. 
- 키워드를 이용하지 않기 때문에 문법적으로 더 간단하고 효율적으로 표현됩니다.
~~~scala
var p = Person("A", 10)
~~~

### 특징 5. 자동 코드 생성
- 케이스 클래스는 컴파일러가 toString, hashCode, equals를 자동으로 생성해 줌
- 컴파일러가 불변 객체를 지원하기 위해서, 새로운 복제 객체를 리턴하는 `copy()` 메서드를 자동으로 생성해 줌
~~~scala
var p = Person("A", 10)

scala> p.toString
res9: String = Person(A,10)

scala> p.hashCode
res10: Int = 649684425
~~~

## 12. 패턴 매칭
- 스칼라의 패턴 매칭은 자바의 switch 문과 유사하지만, 자바와는 달리 기본 자료형외에 케이스 클래스를 이용한 처리가 가능
- 패턴매칭의 기본 문법은 다음과 같습니다. 
- param의 값이 value1의 값과 비교되어 일치하는 값의 결과를 반환합니다. 
- 언더바(_)는 어떤 값도 일치하지 않을 때 처리 결과를 출력합니다.
~~~scala
param match {
  case value1 => "value1"
  case _ => "default value"
}
~~~
- 패턴매칭을 이용하면 if문을 중첩하여 연달아 사용하지 않고 깔끔하게 선택조건을 구분

### 기본 자료형 매칭
- 기본 자료형을 이용한 패턴 매칭의 예제
- 매칭을 이용하는 방법은 다음과 같습니다. 
- `matching` 함수는 파라미터 `x`와 `case` 문의 값들을 비교하여 일치하는 값을 반환
~~~scala
def matching(x:Int): String = x match {
  case 0 => "zero"
  case 1 => "one"
  case 2 => "two"
  case _ => "many"
}

scala> matching(1)
res8: String = one

scala> matching(4)
res9: String = many

scala> matching('a')
res10: String = many
~~~

### 케이스 클래스 매칭
- 케이스 클래스를 이용한 패턴 매칭의 예제를 살펴 보겠습니다. 
- 먼저 케이스 클래스 멤버 변수의 값을 이용하여 매칭을 처리
- `Notification` 추상 클래스를 상속한 `Email`, `SMS`, `VoiceRecording` 케이스 클래스를 생성
~~~scala
abstract class Notification

case class Email(sender: String, title: String, body: String) extends Notification
case class SMS(caller: String, message: String) extends Notification
case class VoiceRecording(contactName: String, link: String) extends Notification

def showNotification(notification: Notification): String = {
  notification match {
    // body 는 반환값에 사용하지 않기 때문에 _로 처리 가능 
    case Email(email, title, _) =>
      s"You got an email from $email with title: $title"
    case SMS(number, message) =>
      s"You got an SMS from $number! Message: $message"
    case VoiceRecording(name, link) =>
      s"you received a Voice Recording from $name! Click the link to hear it: $link"
  }
}

val email = Email("to@gmail.com", "Help Me", "I have a Question")
val someSms = SMS("12345", "Are you there?")
val someVoiceRecording = VoiceRecording("Tom", "voicerecording.org/id/123")

scala> println(showNotification(email))
You got an email from to@gmail.com with title: Help Me

scala> println(showNotification(someSms))
You got an SMS from 12345! Message: Are you there?

scala> println(showNotification(someVoiceRecording))
you received a Voice Recording from Tom! Click the link to hear it: voicerecording.org/id/123
~~~

### 케이스 클래스 매칭 패턴 가드
- 패턴 가드는 `if <논리 표현>` 문을 이용하여 패턴 처리를 구체화하는 방법
- 다음은 `showNotification`과 유사하지만 특정 데이터가 존재하면 다른 메시지를 출력하는 예제
~~~scala
 def showImportantNotification(notification: Notification, importantPeopleInfo: Seq[String]): String = {
    notification match {
      // importantPeopleInfo에 같은 이메일이 존재 
      case Email(email, _, _) if importantPeopleInfo.contains(email) => "You got an email from special someone!"
      // importantPeopleInfo에 같은 번호가 존재 
      case SMS(number, _) if importantPeopleInfo.contains(number) => "You got an SMS from special someone!"
      case other => showNotification(other) // 일치하지 않으면 showNotification 호출 
    }
  }

  // 패턴 체크를 위한 중요한 인물 정보 
  val importantPeopleInfo = Seq("867-5309", "jenny@gmail.com")

  val someSms = SMS("867-5309", "Are you there?")
  val someVoiceRecording = VoiceRecording("Tom", "voicerecording.org/id/123")
  val importantEmail = Email("jenny@gmail.com", "Drinks tonight?", "I'm free after 5!")
  val importantSms = SMS("867-5309", "I'm here! Where are you?")

scala> println(showImportantNotification(someSms, importantPeopleInfo))
You got an SMS from special someone!

scala> println(showImportantNotification(someVoiceRecording, importantPeopleInfo))
you received a Voice Recording from Tom! Click the link to hear it: voicerecording.org/id/123

scala> println(showImportantNotification(importantEmail, importantPeopleInfo))
You got an email from special someone!

scala> println(showImportantNotification(importantSms, importantPeopleInfo))
You got an email from special someone!
~~~

### 케이스 클래스 패턴 매칭 예제
- 다음은 색상을 출력하는 케이스 클래스 패턴 매칭 예제
-  Color를 상속한 Red, Green, Blue 클래스를 이용하여 색상의 값을 보여줌
~~~scala
package sdk.scala.sample

object PatternMatchingSample2 extends App {

  class Color(val red:Int, val green:Int, val blue:Int)

  case class Red(r:Int) extends Color(r, 0, 0)
  case class Green(g:Int) extends Color(0, g, 0)
  case class Blue(b:Int) extends Color(0, 0, b)

  def printColor(c:Color) = c match {
    case Red(v) => println("Red: " + v)
    case Green(v) => println("Green: " + v)
    case Blue(v) => println("Blue: " + v)
    case col:Color => {
      printf("Red: %d, Green: %d, Bluee: %d\n", col.red, col.green, col.blue)
    }
    case _ => println("invalid color")
  }

  printColor(Red(10))
  printColor(Green(20))
  printColor(Blue(30))
  printColor(new Color(10, 20, 30))
  printColor(null)
}
~~~

### 클래스 타입 매칭
- 케이스 클래스의 타입을 이용한 패턴 매칭의 예제를 살펴 보자
- 케이스 클래스의 타입을 자동으로 확인하여 함수를 호출
~~~scala
abstract class Device
case class Phone(model: String) extends Device {
  def screenOff = "Turning screen off"
}

case class Computer(model: String) extends Device {
  def screenSaverOn = "Turning screen saver on..."
}

// Device를 받아서 클래스의 타입을 자동으로 확인하여 다른 처리 
def goIdle(device: Device) = device match {
  case p: Phone    => p.screenOff
  case c: Computer => c.screenSaverOn
}

  val phone = Phone("Galaxy")
  val computer = Computer("Macbook")

scala> println(goIdle(phone))
Turning screen off

scala> println(goIdle(computer))
Turning screen saver on...
~~~

### 값을 이용한 패턴 매칭
- 숫자와 문자열 형식의 값을 전달하여 성적을 표현하는 패턴 매칭 예제를 확인해 보자
- 아래와 같은 방식으로 패턴 매칭을 사용하면 if 문을 열거하지 않고도 깔끔하게 문제를 표현할 수 있음
~~~scala
object PatternMatchingSample extends App {
  val VALID_GRADES = Set("A", "B", "C", "D", "F")

  def letterGrade(value:Any):String = value match {
    case x if (90 to 100).contains(x) => "A"
    case x if (80 to 90 ).contains(x) => "B"
    case x if (70 to 80 ).contains(x) => "C"
    case x if (60 to 70 ).contains(x) => "D"
    case x if ( 0 to 60 ).contains(x) => "F"
    case x:String if VALID_GRADES(x.toUpperCase()) => x.toUpperCase()
  }

  println(letterGrade(91))
  println(letterGrade(72))
  println(letterGrade(44))
  println(letterGrade("B"))
}
~~~

## 13. 믹스잇 컴포지션
- 믹스인 컴포지션은 클래스와 트레잇을 상속할 때 서로 다른 부모의 변수, 메소드 를 섞어서 새로운 정의를 만드는 것
- 추상 클래스 A는 message 변수를 가지고 있음
- 클래스 B는 추상 클래스 A를 상속 하면서 변수 message를 초기화
- 트레잇 C는 추상 클래스 A를 상속하면서 loudMessage 함수를 선언
- 클래스 D는 클래스 B와 트레잇 C를 믹스인하여 클래스 B의 message 를 이용하는 loudMessage 함수를 생성할 수 있음
~~~scala
abstract class A {
  val message: String
}

class B extends A {
  val message = "I'm an instance of class B"
}

trait C extends A {
  def loudMessage = message.toUpperCase()
}

class D extends B with C

// 이 클래스를 생성하고 실행하면 다음과 같이 확인
scala> val d = new D
d: D = D@2c674d58

scala> println(d.message) 
I'm an instance of class B

scala> println(d.loudMessage)
I'M AN INSTANCE OF CLASS B
~~~

## 14. trait(트레잇)
- 트레잇(trait)은 자바의 인터페이스와 유사
- 메소드를 정의만 해놓을 수도 있고, 기본 구현을 할 수도 있음
- 추상 클래스와 달리 생성자 파라미터는 가질 수 없음(ex) trait name(val S:String, val T:String) --> X)
- 트레잇에서는 가변 변수, 불변 변수 모두 선언 가능
- 트레잇을 구현하는 클래스에서 가변 변수는 수정이 가능하지만, 불변 변수는 수정할 수 없음
- 트레잇의 기본 메소드는 상속되고, `override` 키워드를 이용하여 메소드를 재정의 할 수도 있음
- 트레잇은 extends로 상속하고 여러개의 트레잇을 with 키워드로 동시에 구현할 수 있음
- <b>상속하여 구현할 수 있기 때문에 추상클래스와 유사하지만 생성자 멤버변수를 가질 수는 없음</b>
- 추상클래스는 하나만 상속할 수 있지만, 트레잇은 여러개를 상속 할 수 있음
- <b>생성자 멤버변수가 필요하면 추상클래스를 이용하는 것이 좋고, 생성자 멤버변수가 필요 없다면 트레잇을 이용하는 것이 좋음</b>
~~~scala
// Machine 트레잇 
trait Machine {
  val serialNumber: Int = 1
  def work(message: String)
}

// KrMachine 트레잇 
trait KrMachine {
  var conturyCode: String = "kr"
  def print() = println("한글 출력")    // krprint 기본 구현 
}

// Computer 클래스는 Machine, KrMachine를 둘다 구현합니다. 
class Computer(location: String) extends Machine with KrMachine {
  this.conturyCode = "us"   // code 값 변경 
  def work(message: String) = println(message)
}

// Car 클래스는 Machine, KrMachine를 둘다 구현합니다.
class Car(location: String) extends Machine with KrMachine {
  def work(message: String) = println(message)
  override def print() = println("운전중입니다.") // print 재정의
}

var machine = new Computer("노트북")
var car = new Car("포르쉐")

scala> machine.work("computing...")
computing...

scala> machine.print()
한글 출력

scala> println(machine.conturyCode)
us

scala> car.work("driving...")
driving...

scala> car.print() 
운전중입니다.

scala> println(car.conturyCode)
kr
~~~

## 15. 싱글톤 객체  
- 스칼라에서 obejct 선언자로 싱글톤 객체를 생성
- 싱글톤 객체의 메서드는 전역적으로 접근하고 참조 가능
- 싱글톤 객체는 직접적으로 접근해서 사용할 수도 있고, `import` 를 선언하여 이용할 수도 있음
~~~scala
object Bread {
  val name: String = "기본빵"
  def cooking() = println("빵만드는 중...")
}

// import 이용
scala> import Bread.cooking
scala> cooking
빵만드는 중...

// 직접 접근 
scala> Bread.cooking
빵만드는 중...
~~~

### 컴패니언
- 싱글톤 객체와 클래스가 같은 이름을 사용하면 컴패니언(Companions)이라고 함
- 컴패니언은 정적 메소드의 보관 장소를 제공
- 자바의 `static`을 이용한 정적 데이터는 스칼라에서는 컴패니언을 이용하여 처리하는 것을 추천
- 팩토리 메소드 같은 정적 메소드는 컴패니언을 이용하여 작성하고, 일반적인 데이터는 클래스를 이용하여 정적 데이터와 일반 데이터를 분리하여 관리할 수 있음
~~~scala
class Dog

object Dog {
    def bark = println("bark")
}

scala> Dog.bark
bark
~~~
- 컴패니언을 이용한 팩토리 예제
~~~scala
class Email(val username: String, val domainName: String)

object Email {
  def fromString(emailString: String): Option[Email] = {
    emailString.split('@') match {
      case Array(a, b) => Some(new Email(a, b))
      case _ => None
    }
  }
}

val scalaCenterEmail = Email.fromString("scala.center@epfl.ch")
scalaCenterEmail match {
  case Some(email) => println(
    s"""Registered an email
       |Username: ${email.username}
       |Domain name: ${email.domainName}
     """)
  case None => println("Error: could not parse email")
}

scala> 
Registered an email
Username: scala.center
Domain name: epfl.ch
~~~

## 16. 콜렉션(collection)
- 스칼라는 여러가지 유용한 기본 콜렉션 자료구조를 제공
- 여러가지 자료구조 중에서 자주 사용하는 배열, 리스트, 셋, 튜플, 맵의 사용방법에 대해서 알아보자

### 배열
- 배열은 길이가 고정된 고정된 자료구조 
- 배열의 데이터에 접근하는 법과 다른 배열을 연결하는 법을 알아보자
~~~scala
val array1 = Array(1, 2, 3)

// 배열의 데이터 접근 
scala> array1(0)
res0: Int = 1

// 배열의 데이터 변경 
scala> array1(1) = 10
scala> array1(1)
res5: Int = 10

val array2 = Array(3, 4, 5)

// 배열 연결하기 ++
val array3 = array1 ++ array2

// 배열의 앞에 데이터 추가 
val array4 = 0 +: array3

// 배열의 뒤에 데이터 추가 
val array5 = array3 :+ 100
~~~

### 리스트
- 리스트 가변적인 길이의 데이터를 저장하기 위한 자료구조 
- 리스트를 생성하는 방법과 리스트의 데이터에 접근하는 방법
~~~scala
val list1 = List(10, 20, 30, 40)
val list2 = (1 to 100).toList
val list3 = array1.toList

scala> list1
res19: List[Int] = List(10, 20, 30, 40)

scala> list1(0)
res11: Int = 10

scala> list1(3)
res12: Int = 40

scala> list1.head
res17: Int = 10

scala> list1.tail
res18: List[Int] = List(20, 30, 40)
~~~

### 셋(Set)
- 셋은 중복을 허용하지 않는 자료 구조 
- <b>셋은 전달된 값이 존재하는지 여부를 반환</b>
~~~scala
val s1 = Set(1, 1, 2)

scala> s1
res23: scala.collection.immutable.Set[Int] = Set(1, 2)

// 셋은 전달된 값이 존재하는지 여부를 반환 
scala> s1(1)
res26: Boolean = true

scala> s1(2)
res27: Boolean = true

scala> s1(3)
res28: Boolean = false
~~~

### 튜플
- 튜블은 불변의 데이터를 저장하는 자료구조 
- 여러가지 값을 저장할 수 있습니다. 
- 값에 접근할 때는 `_1`, `_2`와 같은 형태로 접근
- 튜플은 <b>패턴매칭</b>에 이용할 수 있음
~~~scala
// 튜플 선언 
val hostPort = ("localhost", 80)

scala> hostPort._1
res29: String = localhost
~~~
- 튜플을 이용한 패턴 매칭은 다음과 같이 사용
~~~scala
// 패턴매칭 선언 
def matchTest(hostPort:(String, Int)) = hostPort match {
  case ("localhost", port) =>  println(s"localhost, $port")
  case (host, port) => println(s"$host, $port")
}

val hostPort1 = ("localhost", 80)
val hostPort2 = ("localhost", 8080)
val hostPort3 = ("127.0.0.1", 8080)

scala> matchTest(hostPort1)
localhost, 80

scala> matchTest(hostPort2)
localhost, 8080

scala> matchTest(hostPort3)
127.0.0.1, 8080
~~~

### 맵
- 맵은 사전 형식으로 데이터를 저장하는 구조
- 맵을 생성하고 사용하는 방법은 다음과 같음
- 맵의 데이터를 반환하면 `Option` 타입을 반환
- `getOrElse`를 이용하거나, `get`의 반환값 `Option`을 패턴매칭을 이용하는 것이 좋음
~~~scala
val map1 = Map(1 -> 2)
val map2 = Map("foo" -> "bar")

// Option 타입 반환 
scala> map1.get(1)
res43: Option[Int] = Some(2)

// getOrElse를 이용하여 키와 일치하는 데이터가 없으면 기본값을 반환하도록 설정 
scala> map1.getOrElse(1, 0)
res41: Int = 2

scala> map1.getOrElse(10, 0)
res45: Int = 0
~~~

## 17. 반복문
- 스칼라의 반복문은 for, do while, while 문이 있음

### for
- for 문을 사용하는 방법은 다음과 같음
- `to`와 `until`은 간단하게 시퀀스를 생성하게 도와주는 스칼라의 문법
- `to`는 이하의 리스트를 생성하고, `until`은 미만의 시퀀스를 생성합니다.
~~~scala
// 0에서 3이하의 시퀀스 
for (num <- 0 to 3)
    println(num)
0
1
2
3

// 0에서 3미만의 시퀀스 
for (num <- 0 until 3)
    println(num)
0
1
2

// 콜렉션의 데이터 출력 
val strs = Array("A", "B", "C", "D", "E")
for (str <- strs)
    println(str)
A
B
C
D
E
~~~

### 배열의 인덱스와 함께 for 문 처리
- 인덱스와 함께 처리하는 방법은 배열의 길이만큼 인덱스를 호출하여 처리하거나, `zipWithIndex` 함수를 이용
~~~r 
// 콜렉션의 데이터를 인덱스와 함께 출력 
for(index <- 0 until strs.length)
    println(index, strs(index))
(0,A)
(1,B)
(2,C)
(3,D)
(4,E)

// 콜렉션의 데이터를 zipWithIndex를 이용하여 인덱스와 함께 출력 
for((value, index) <- strs.zipWithIndex)
    println(value, index)
(A,0)
(B,1)
(C,2)
(D,3)
(E,4)
~~~

### 맵의 키, 밸류를 for문 처리
- 맵의 키, 밸류는 튜플로 전달됩니다. for문 처리시에 이 값들을 이용하면 됩니다
~~~scala
// 맵 데이터 출력 
val map = Map("k1" -> "v1", "k2" -> "v2", "k3" -> "v3", "k4" -> "v4", "k5" -> "v5")
for ((k, v) <- map)
  println(k, v)

(k2,v2)
(k5,v5)
(k1,v1)
(k4,v4)
(k3,v3)
~~~

### 중첩 for 문
- 중첩 for문은 자바처럼 for문을 두개를 이용해서 생성할 수도 있지만, 
- 세미 콜론(;)을 이용하여 생성할 수도 있습니다. 간단한 중첩문 생성 방법은 다음과 같습니다.
~~~scala
for (x <- 0 to 2; y <- 0 to 2)
    println(x, y)

(0,0)
(0,1)
(0,2)
(1,0)
(1,1)
(1,2)
(2,0)
(2,1)
(2,2)
~~~

### 조건식 추가
- `for`문에 데이터 생성을 위한 조건식을 이용할 수도 있음
- 세미콜론으로 `if`문을 분할하여 여러개의 조건을 추가할 수도 있음
~~~scala
for (num <- 0 to 3; if num != 2)
    println(num)
0
1
3

for (x <- 0 to 2; y <- 0 to 2; if x < 1)
    println(x, y)
(0,0)
(0,1)
(0,2)

for (x <- 0 to 2; y <- 0 to 2; if x < 1; if y < 1)
    println(x, y)
(0,0)
~~~

### yield를 이용한 시퀀스 컴프리헨션
- 스칼라의 시퀀스 컴프리헨션은 for문의 끝에 yield를 이용해서 for문에서 생성한 값들의 시퀀스를 반환
~~~scala
/ 0에서 n이하의 시퀀스에서 5의 배수로 구성된 시퀀스 생성 
def fives(n: Int) = {
    for( x <- 0 to n; if x % 5 == 0)
      yield x
}

for(num <- fives(100))
    println(num)
~~~
~~~scala
def checkSum(num: Int, sum: Int) =
    for (
      start <- 0 until num;
      inner <- start until num if start + inner == sum
    ) yield (start, inner); 

checkSum(20, 32) foreach {
    case (i, j) =>
      println(s"($i, $j)")
}
~~~

### do..while 문
- do while문은 조건이 참일동안 반복됨
~~~scala
var i = 0;

do{
    println(i)
    i += 1
} while(i < 3)

0
1
2
~~~

### while 문
- while문은 조건이 참일동안 반복됩니다.
~~~scala
  var num = 0;
  while (num < 3) {
    num += 1
    println(num);    
  }

1
2
3
~~~

## 18. 정렬, 그룹핑, 필터링 등 함수
- 콜렉션을 이용해서 주로 처리하게 될 작업에서 효율적으로 활용할 수 있는 주요 함수

### map
- `map` 함수는 콜렉션의 각 아이템에 대해서 동일한 작업을 해야할 때 사용할 수 있음
- 다음은 리스트의 각 아이템에 대해서 1을 추가하는 작업과 대문자로 변환하는 작업
~~~scala
// 각 값에 1 추가 
val list = (1 to 10)
list.map(_ + 1)
// 결과 
Vector(2, 3, 4, 5, 6, 7, 8, 9, 10, 11)

// 각 문자를 대문자화 
val strs = List("david", "kevin", "james")
strs.map(_.toUpperCase)
// 결과 
List(DAVID, KEVIN, JAMES)
~~~

### reduce, fold
- `reduce`, `fold` 함수는 콜렉션의 데이터를 집계할 때 사용합니다. 
- 두 개의 함수는 동작은 비슷하지만 `fold` 함수는 기본값을 제공할 수 있습니다. 
- 각각의 함수모두 left, right 방향을 가질 수 있습니다. 
- 더하기(+) 연산의 경우 양쪽 방향이 동일한 결과를 나타내지만 
- 빼기(-) 연산의 경우 방향에 따라 다른 결과를 나타냅니다.

### groupBy
- `groupBy`는 데이터를 키를 기준으로 병합할 때 사용
- 결과를 `Map` 형식으로 반환하고, 전달된 키와 리스트 형태의 데이터로 반환
~~~scala
var datas = List(("A", 1), ("B", 2), ("C", 6), ("B", 2), ("A", 8), ("C", 2))
datas.groupBy(_._1).foreach({ case (k, v) => printf("key: %s, value: %s\n", k, v) })

// 결과 
key: A, value: List((A,1), (A,8))
key: C, value: List((C,6), (C,2))
key: B, value: List((B,2), (B,2))
~~~

### filter
- `filter`는 콜렉션의 데이터를 필터링하여 없애거나 분류할 때 사용할 수 있음
- `partition`은 콜렉션을 분류할 때 사용
-  `find`는 데이터를 검색할 때 사용
- `takeWhile`과 `dropWhile`을 이용하여 원하는 부분까지 데이터를 선택할 수 있음
~~~scala
val list = (1 to 10)
list.filter(_ > 5)  // 5 이상의 데이터 분류 
// 결과: Vector(6, 7, 8, 9, 10)

list.partition(_ % 3 == 0)  // 2로 나누어 나머지가 0인 데이터를 분류 
// 결과: (Vector(3, 6, 9),Vector(1, 2, 4, 5, 7, 8, 10))

list.find(_ == 3)   // 3을 검색 
// 결과: Some(3)

val list2 = List(1, 2, 3, -1, 4, 5, 6)
list2.takeWhile(_ > 0)
// 결과: List(1, 2, 3)

val list3 = List(1, 2, 3, 4, 5, 6)
list3.dropWhile(_ < 2)
// 결과: List(2, 3, 4, 5, 6)
~~~

### zip
- `zip`은 두개의 콜렉션은 같은 인덱스의 데이터를 묶을 수 있음
-  길이가 일치하지 않으면 작은 갯수 만큼만 반환
~~~scala
for( item <- List(1,2,3).zip(List(1,2,3)))
      println(item)
(1,1)
(2,2)
(3,3)

for( item <- List(1,2,3).zip(List(1,2,3,4)))
      println(item)
(1,1)
(2,2)
(3,3)
~~~

### mapValues
- `mapValues`는 Map 타입의 데이터에서 밸류만 map 함수 처리를 하고 싶을 때 사용하는 함수
~~~scala
// value를 제곱
var maps = Map("A" -> 1, "B" -> 2, "C" -> 3, "D" -> 4, "E" -> 5)
maps.mapValues(x => x*x).foreach( x => x match { case (k, v) => printf("key: %s, value: %s\n", k, v) })

// 결과
key: E, value: 25
key: A, value: 1
key: B, value: 4
key: C, value: 9
key: D, value: 16

// value인 List의 sum을 구함 
var maps = Map("A" -> List(1, 2, 3), "B" -> List(4, 5, 6), "C" -> List(7, 8, 9))
maps.mapValues(_.sum).foreach({ case (k, v) => printf("key: %s, value: %s\n", k, v) })

// 결과 
key: A, value: 6
key: B, value: 15
key: C, value: 24
~~~

### sort
- 정렬은 `sorted`, `sortWith`, `sortBy` 세가지 메소드를 이용할 수 있음
~~~scala
// sorted 사용방법 
val list = List( 4, 6, 1, 6, 0)
val l_sort = list.sorted
val r_sort = list.sorted(Ordering.Int.reverse)

scala> println(l_sort)
List(0, 1, 4, 6, 6)
scala> println(r_sort)
List(6, 6, 4, 1, 0)

// sortBy 사용방법 
val sList = List("aa", "bb", "cc")
val l_sortBy = sList.sortBy(_.charAt(0))
scala> println(l_sortBy)
List(aa, bb, cc)

// sortWith 사용방법 
val l_sortWith = list.sortWith(_ <= _)
val r_sortWith = list.sortWith(_ >= _)
scala> println(l_sortWith)
List(0, 1, 4, 6, 6)
scala> println(r_sortWith)
List(6, 6, 4, 1, 0)


// 케이스 클래스의 데이터를 정렬 
// correct가 같으면 index로 정렬 
case class Person(index:Int, var correct:Int)

val persons = Array(Person(1, 3),Person(2, 4), Person(3, 4))
val list = persons.sortWith((x:Person, y:Person) => {
  if(x.correct == y.correct)
      x.index >= y.index

    x.correct > y.correct
}).toList

scala> println(list)
List(Person(2,4), Person(3,4), Person(1,3))
~~~

## lazy val
- 지연 변수로써 선언되는 순간에 변수에 RHS를 계산해서 LHS로 할당하는 것이 아니라  
  호출되는 시점에 계산함

## Option class
- `Option` 클래스는 어떤 값이 있을 수도, 없을 수도 있는 상황에 쓸 수 있는 간단한 형태의 컨테이너 클래스
- 해당 클래스는 abstract class 이므로 인스턴스를 만들 수 없고 코드에서는 이 클래스의 하위 클래스인 `Some`이나 `None`을 사용
- 


## 용어 정리
- trait
  - Scala 는 interface 가 없으며 대신  trait 을 사용한다.
  - Scala 의 trait 는 자바의 interface 와 달리 구현 가능하다. (자바8 부터는 자바인터페이스도 디폴트메소드등 구현)
  - 하나의 부모클래스를 갖는 클래스의 상속과 달리 트레이트는 몇개라도 조합해 사용 가능하다. 
  - Scala 의 trait 는 자바의  인터페이스와 추상클래스의 장점을 섞었다. 
  - 트레이트를 믹스인 할때는 extends 키워드를 이용한다. 
  - extends 를 사용하면 trait 의 슈퍼클래스를 암시적으로 상속하고 , 본래 trait 를 믹스인한다. 
  - trait 는 어떤 슈퍼클래스를 명시적으로 상속한 클래스에 혼합할 수 있다. 
    그때 슈퍼클래스는 extends 를 사용하고, trait 는 with 로 믹스인한다. 

