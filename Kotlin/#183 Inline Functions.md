# [**Inline Functions**](https://100androidquestionsandanswers.us12.list-manage.com/track/click?u=f39692e245b94f7fb693b6d82&id=3bedc7da4b&e=056158c869)

## Kotlin Weekly #183

### 핵심 

1. 코드를 bytecode로 바로 변환한다.

```kotlin
inline fun example1(data: (Int) -> Unit) {
    data(1)
}
// kotlin to bytecode
    LINENUMBER 7 L2
    ALOAD 0
    ICONST_1
    INVOKESTATIC java/lang/Integer.valueOf (I)Ljava/lang/Integer;
    INVOKEINTERFACE kotlin/jvm/functions/Function1.invoke (Ljava/lang/Object;)Ljava/lang/Object; (itf)
    POP
   L3
    LINENUMBER 8 L3
    RETURN
   L4

```

```kotlin
fun example2(data: (Int) -> Unit) {
    data(2)
}
// kotlin to bytecode
    LINENUMBER 11 L1
    ALOAD 0
    ICONST_2
    INVOKESTATIC java/lang/Integer.valueOf (I)Ljava/lang/Integer;
    INVOKEINTERFACE kotlin/jvm/functions/Function1.invoke (Ljava/lang/Object;)Ljava/lang/Object; (itf)
    POP
   L2
    LINENUMBER 12 L2
    RETURN
   L3

```

- 위 두개와 같이 일반적인 함수가 변환되는 과정은 똑같다.



```kotlin
example1 { println(it) }
//kotlin to bytecode
    LINENUMBER 2 L0
   L1
    ICONST_0
    ISTORE 0
   L2

```

```kotlin
example2 { println(it) }
//kotlin to bytecode
    LINENUMBER 3 L11
    GETSTATIC NotePadKt$main$2.INSTANCE : LNotePadKt$main$2;
    CHECKCAST kotlin/jvm/functions/Function1
    INVOKESTATIC NotePadKt.example2 (Lkotlin/jvm/functions/Function1;)V
   L12

```

- 위 예제에서 보면 실제 함수를 사용할 때, 인라인되지 않은 함수는 더 많은 코드를 가져오는 것을 볼 수 있다.

- inlines 는 컴파일러가 직접 함수가 불린 지점에 bytecode 로 가져다 놓는것을 의미한다.

```kotlin
inline fun <T> log(name: String, action: () -> T) {
    println("Before $name")
    action.invoke()
    println("After $name")
}

log("foo") { println("foo") }

// compiler generates
println("Before foo")
println("foo")
println("After foo")
```

2. 지역 스코프에서 반환이 가능하다.

   - 함수안의 함수에서 inline된 함수는 바이트 코드화 되기 때문에 반환값이 그 상위 함수로 종속된다.

3. reified type parameters [참고자료](https://sungjk.github.io/2019/09/07/kotlin-reified.html)

   - reified type이란 inline과 함께 쓰인다.

   - reified : 추상화된 개념을 구체화 하다 (사전적 의미)

   - 일반적인 제네릭 함수의 타입 T는 컴파일시에는 존재하지만 런타임에는 존재하지 않음

     > Type erasure
     >
     > 런타임시 클래스가 여러개 생성되는것을 방지하기 위해
     >
     > 혹은 하위 호환성을 위해 제네릭 타입을 모두 Object 타입으로 변경한다.
     >
     > <T extends Comparable<T>> -> Comparable 타입으로 변경 됨

   - **reified는 컴파일러가 함수의 바이트코드를 함수가 사용되는 모든 곳에 복사하도록 만든다.**
   -  reified type과 함께 인라인 함수가 호출되면 컴파일러는 type argument로 사용된 실제 타입을 알고 만들어진 바이트코드를 직접 클래스에 대응되도록 바꿔준다.
   - reified 타입 파라미터로 작성된 인라인 함수는 **자바 코드에서 호출 할 수 없다.**



## [본문]

### 언제 inline 함수를 사용할까?

#### 	성능

- 우리는 Collection operation을 사용할때 항상 inline 함수를 사용한다. (map, filter와 같은 함수)
- 비용없이 고차함수를 사용하는것을 허락하는 좋은 소식이다. - 깨끗하고 빠른 함수형 프로그래밍이 가능하게 한다.

```kotlin
val newList = listOf(1, 2, 3, 4).map { it * 2 }

// compiles to
val temp: MutableList<Int> = mutableListOf()
for (x in listOf(1, 2, 3, 4)) {
	temp.add(x * 2)
}

val newList = temp.toList()
```

> Collection을 직접적으로 다루는것 대신 Collection 함수를 사용하는건은 더 깨끗하고 에러가 덜 발생한다.
>
> 그리고 그것은 성능면에서 동등하다.

### 왜 inlining은 성능을 가져오나?

​	인라인 함수가 성능을 향상시키는 것을 이해하기 위해서 우리는 **함수를 인라인하지 않았을 때**, 코틀린이 무엇을 하는지 이해해야 한다.

- 함수의 코드를 가져오기 위해서, 메모리의 다른 공간으로 점프시킨다.
- 만약 함수가 람다 argument를 가지고 있으면 모든 정보를 담을 수 있는 추가적인 코드가 생성된다.

인라인을 사용하면 위 두가지를 해결 할 수 있다.



### 지역 스코프가 아닌 반환(Non-local returns)

Collection에 대해 말하자면, 컬렉션에서 return 할수 있다는것을 알고 있는가?

```kotlin
fun findIndex(value: String, list: List<String>): Int? {
    list.forEachIndexed { index, item ->
        if (item == value) return index
    }

    return null
}
```

일반적인 함수에서는 컴파일러는 return을 허락하지 않는다!

```kotlin
fun bar(): Int {
    normalFunction {
        return 5   // `return` is not allowed here!
    }
}
```

즉, 첫번 째 findIndex 함수의 forEachIndexed 안에서 findIndex의 반환값을 선언 할 수 있다.

그 이유는 컴파일러는 코드를 인라인화 하기 때문에 리턴이 가능하다.

> 컴파일러는 인라인 기능으로 인해 함수를 벗겨냈다.

하지만 두번 째, bar() 함수 안에서 다른 일반적인 함수를 사용하고 거기에서 bar()함수의 반환값을 리턴한다면 컴파일러는 허락하지 않는다.



### 수정된 유형의 매개변수 (Reified Type Parameters)

가끔, 제네릭을 사용하는 함수들은 타입정보를 필요로 한다. (Ex : ``filterIsInstance``)

```kotlin
abstract class Animal(name: String)

data class Dog(val name: String): Animal(name)
data class Cat(val name: String): Animal(name)

val pets: List<Animal> = listOf(Cat("Cleatus"), Cat("Jim"), Dog("Morgana"))

pets.filterIsInstance<Dog>()  // [Dog(name=Morgana)]
```

함수가 인라인되면, 우리는 어떤 리플렉션도 할 필요가 없다.

우리는 ``is`` 혹은 ``as`` 연산을 그대로 사용 할 수 있다.

``filterIsInstance`` 함수가 간단하게 ``Iterable``의 extention 이라 가능하다.

```kotlin
inline fun <reified T> kotlin.collections.Iterable<*>.filterIsInstance(): List<T> {
    val newList = mutableListOf<T>()

    this.forEach { if (it is T) newList.add(it) }

    return newList.toList()
}
```



### 인라인 함수에서 타입 삭제가 적용되지 않는 이유 

### (Why doesn't type erasure apply to inlined functions?)

컴파일러가 함수의 바이트코드를 인라인화 시키기 때문에 인자로 사용되는 실제 타입을 알고 있다.

컴파일러가 인라인된 코드를 붙여넣으면 제네릭에 관한 정보가 없다.

컴파일러는 함수가 불린 위치에서 사용된 실제 타입을 사용한다.

그 타입은 우리가 접근할수 있고 정보를 가져올 수 있다.



이것은 일반적인 함수와 굉장히 다른 행동이다. 타입 제거가 있는 곳에서도 타입 정보를 수집할 수 있을 정도로 다르다.



### inline 함수의 제한 (Limitations of `inline` functions)

그렇다면 왜 모든 함수를 인라인시키지 않는가?

1. 큰 바이트 코드
2. 스택 추적기능 상실
3. 가시성 제한 불가능
4. 재귀 함수 불가능

큰 인라인 함수를 만든다면 (예를들어 인라인 함수를 부르는 인라인 함수) 코드는 빠르게 커질것이다.

컴파일러는 모든 함수 호출부분에 코드를 붙일것이다.

또, 디버깅을 해야 할 경우 스택 추적기능을 상실한다.



인라인 기능은 가시성을 제한할수 없다. 인라인 기능이 다른 모듈의 클래스에 붙여질 수 있으며, 이 모듈에서 사용하고 있는 private 클래스는 접근할 수 없게 된다.



마지막으로 재귀기능을 할 수 없다. 그 이유는 함수스택이 쌓이지 않기 때문이다.





Inline 은 성능향상을 위한 좋은 도구이지만 적절한 때에 사용해야 한다.