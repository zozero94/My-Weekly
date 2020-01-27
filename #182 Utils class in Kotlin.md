# [Utils class in Kotlin](https://medium.com/@nhoxbypass/utils-class-in-kotlin-387a09b8d495) (Kotlin Weekly #182)

##### 기존 JAVA 사용자들은 Static Method에  class name을 사용하여 직접 접근하는것에 익숙하다.



- 자바와 다르게 코틀린에는 static이라 불리는 것이 없다.
> Unlike Java, there is nothing called static in Kotlin.



#### Kotlin에는 세 가지 일반적인 타입의 객체들이 존재한다.

1. Object Declaration

2. Companion Object
3. Object Expression



### 1.  Object Declarations

- Kotlin은 매우 쉬운 방법으로 Singleton을 선언한다.

```kotlin
object SaberFactory {
    fun makeLightSaber(powers: Int): LightSaber {
        return LightSaber(powers)
    }
}
```

- Object의 인스턴스는 단 한가지이며, 처음 접근할 때 안전한 방식으로 만들어진다.

> Like singleton, there is only one instance of an object, which is created the first time it’s accessed, in a thread-safe manner.
- Object 클래스를 호출할 때, 우리는 이름으로 바로 접근 할 수 있다.
```kotlin
val saber = SaberFactory.makeLightSaber(150)
```

<hr>

### 2. Companion Objects

- companion object는 companion keyword 로 **클래스에 묶인** 객체이다.

```kotlin
class Saber(val powers: Int) {
    companion object Factory {
        fun makeLightSaber(powers: Int): LightSaber {
            return LightSaber(powers)
        }
    }
}
```

- Companion object는 메소드가 클래스의 인스턴스보다는 클래스에 묶여있다고 선언하는 데 사용될 수 있다.
- 즉, ```fun makeLightSaber``` 은 메소드에 한정되지 않고 클래스에 한정된다고 보면 된다.

> Companion object can be used to declare method to be tied to a class rather than to instances of it.

- Companion object도 싱글톤이다, 그래서 메소드 또한 바로 접근 할 수 있다.

```kotlin
val saber = Saber.makeLightSaber(150)
```



### 3. 일반 클래스와의 차이

초기화 시점에 ``new ``키워드를 사용 할 필요가 없다.

> In fact, object is just a data type with a single instance. So if we want to find something similar in Java, that would be the Singleton pattern.

사실, object는 단지 하나의 인스턴스를 가진 데이터 타입이다.

그래서 만약 너가 자바에서 비슷한것을 찾는다면 , 그것은 싱글톤 패턴일것이다.

```kotlin
object SaberFactory {
    fun makeLightSaber() { /*...*/ }
}

class SaberFactory {
    fun makeLightSaber() { /*...*/ }
}
```

오브젝트 클래스와 일반 클래스를 자바로 디컴파일 하면 아래와 같이 나타난다.

```kotlin
// Generated class

public final class SaberFactory {
   public static final SaberFactory INSTANCE = new SaberFactory();

   private SaberFactory() { }

   public final void makeLightSaber() { /*...*/ }
}

public final class SaberFactory {
   public final void makeLightSaber() { /*...*/ }
}
```

#### 왜 코틀린은 object를 소개하는가?

주된 이유는 코틀린은 객체지향언어에서 statics에서 벗어나려고 하기 때문이다. 

코틀린은 개발자로 하여금 static을 사용하는것을 못하게 한다.

대신에 싱글톤 object로 static을 대체한다.



자바에 싱글톤은 구현하기 쉽지 않다. 만약 두개의 스레드가 싱글톤에 동시에 접근한다면, 두개의 인스턴스가 생성이 될것이다.



### Package-level functions

> [Package-level functions](https://kotlinlang.org/docs/reference/java-to-kotlin-interop.html#package-level-functions) are all functions declared in a source file without an enclosing class or object.

패키지 레벨의 함수들은 클래스 혹은 오브젝트로 감싸진 소스파일 안에서 선언되어진다.



``org.example`` 패키지 안의 ``app.kt`` 파일에서 불리는 모든 함수는 자바 클래스에서 ``org.example.Appkt``이름으로된 static 함수로 컴파일 된다.

```kotlin
package org.example

fun makeLightSaber() { /*...*/ }
```
이렇게 된다
```java
package org.example

// Generated class
class AppKt {
    public static void makeLightSaber() { /*..*/ }
}
```



자바에서 우리는 Appkt를 불러 메소드를 사용한다.

``org.example.AppKt.makeLightSaber();``

과거에 동일한 패키지의 모든 고차함수들이 패키지의 이름을 딴 단일 Java의 클래스가 되었다.

그러나 JetBrains 는 사용파일이름을 생성된 클래스를 위해 대신 사용한다. (``app.kt`` -> ``AppKt.class``)



만약 우리가 AppKt를 다른 이름으로 바꾸고 싶다면?

##### @File:JvmName

자바클래스에서 생성된 이름은 ``@JvmName`` 어노테이션으로 바뀔수 있다.

```kotlin
@file:JvmName("SaberUtils")

package org.example

fun makeLightSaber() { /*...*/ }
```

그러면 자바에서 ``Appkt `` 대신에 ``SaberUtils`` 이름으로 메소드를 호출 할 수 있다.

``org.example.SaberUtils.makeLightSaber();``



동일한 Java 클래스 이름(동일한 패키지 & 파일 이름, 혹은 동일한 @JvmName)을 가진 여러 파일을 가지는 것은 오류다.

이것은 어떻게 고칠까?

##### @file:JvmMultifileClass

같은 패키지 안의 다른 파일 

(Package : org.example)

(File : lightsaber.kt, darksaber.kt)

```kotlin
// lightsaber.kt
@file:JvmName("SaberUtils")
@file:JvmMultifileClass

package org.example

fun makeLightSaber() { /*...*/ }
```



```kotlin
// darksaber.kt
@file:JvmName("SaberUtils")
@file:JvmMultifileClass

package org.example

fun makeDarkSaber() { /*...*/ }
```



안에서 ``@JvmMultifileClass`` 어노테이션을 사용하면 SaberUtils를 사용하여 접근 할 수 있다.

``org.example.SaberUtils.makeLightSaber();
org.example.SaberUtils.makeDarkSaber();``



코틀린은 정적 방법으로 패키지 레벨의 기능을 나타낸다.

하지만 @JvmStatic을 사용하면 객체의 정의된 정적 방법을 생성 할 수 있다.

이 주석을 사용하면 컴파일러는 객체를 둘러싸는 클래스에 정적 메소드와 객체 자체에서 인스턴스 메소드를 모두 생성한다.



```kotlin
object SaberFactory {
    @JvmStatic fun makeStaticSaber() { /*..*/ }
    
    fun makeNonStaticSaber() { /*..*/ }
}
```

Java에서 너는 두가지를 모두 사용 할 수 있다.



```java
SaberFactory.makeStaticSaber(); // works fine
SaberFactory.makeNonStaticSaber(); // error

SaberFactory.INSTANCE.makeNonStaticSaber(); // works, call through the singleton INSTANCE
SaberFactory.INSTANCE.makeStaticSaber(); // works too
```



# Kotlin 안에서의 Utils Class

### 첫번째 방법

**Object**를 사용한다.

```kotlin
object SaberUtils {
    fun makeLightSaber(powers: Int): LightSaber {
        return LightSaber(powers)
    }
}
```

코틀린에서는 잘 동작한다. 하지만 자바에서 호출할때 ``INSTANCE``를 사용해야 한다.

왜냐하면 이것은 static method가 아닌 싱글톤 인스턴스를 가진 일반 메소드이기 때문이다.

```kotlin
// Call from Kotlin
SaberUtils.makeLightSaber(150)
// Call from Java
SaberUtils.INSTANCE.makeLightSaber(150)
```

### 두번째 방법

**패키지 레벨의 함수(package-level functions)**들을 사용한다.

```kotlin
@file:JvmName("SaberUtils")
@file:JvmMultifileClass

fun makeLightSaber(powers: Int): LightSaber {
    return LightSaber(powers)
}
```

너는 자바코드에서도 ``SaberUtils.makeLightSaber()``로 호출 할 수 있다.

하지만 코틀린 코드에서 ``makeLightSaber()``메소드를 Prefix로 ``SaberUtils`` 없이 사용할 수 있다.

왜냐하면 어떠한 클래스에도 속하지 않기 때문이다.



### 무엇이 더 좋은 접근일까?

두번째 방법이 더 범용적이다.

하지만 첫번째 방법을 선택하는 이유들이 있다.

- object를 사용하여 타이핑을 시작할 때 IDE의 추천 항목에서 모든 Utils가 보여지는것을 피한다.

- Java 개발자로써, 대부분의 메소드들을 ``foo()`` 보다``Utils.foo()``  를 사용하는것을 선호한다.



### 세번째 방법 

object 안에서 @JvmStatic을 사용한다.