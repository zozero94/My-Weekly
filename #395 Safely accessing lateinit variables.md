# [Safely accessing lateinit variables](https://okkotlin.com/lateinits/) (Android Weekly #395)

<u>[Main Issue]</u>

- Reflection API를 사용하여 lateinit check를 할 수 있다. 

  ```kotlin
  lateinit var fullName: String
      
  if (::fullName.isInitialized) {
      print("Hi, $fullName")
  }
  ```

  - Reflection API 적용 가능 범위
    1. 같은 클래스에서 사용 가능
    2. outer class 안에서 사용 가능
    3. 같은 파일의 최 상위 레벨에서 사용 가능

<hr>

[Sub Issue]

- Reflection 이란? 

  - 프로그램 내부 속성을 조작 할 수 있는 API
  - JVM에 로딩되어있는 클래스와 메소드 정보를 가져 올 수 있다.

- ``::`` 을 붙이는 이유?

  - 일급 객체 프로퍼티에 접근할때 사용

  - **런타임시**에 초기화가 되었는지 체크하기 위해서 사용

  - 런타임시 체크하는 이유 -> 자바에는 일급객체가 없기 때문에 runtime시에 reflection 시키기 위해

    [(참조)](https://kotlinlang.org/docs/reference/reflection.html#class-references)

  - ``::`` 는 레퍼런스를 가져오기 위함 (kotlin -> java로 변경시 결국에는 reflection을 해야 함)

- 일급 객체란?

  1. 변수나 데이터에 할당 할 수 있어야 함
  2. 객체의 인자로 넘길 수 있어야 함
  3. 객체의 리턴값으로 리턴 할 수 있어야 함

