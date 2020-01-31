# [Understanding Higher-Order Functions and Lambdas in Kotlin](https://blog.mindorks.com/understanding-higher-order-functions-and-lambdas-in-kotlin) (Kotlin Weekly #178)

### 자바에는 왜 High-Order Function이 없을까?

- Java에는 일급객체의 개념이 존재하지 않는다.
- 일급 객체란?
  1. 변수나 데이터에 할당 할 수 있어야 함
  2. 객체의 인자로 넘길 수 있어야 함
  3. 객체의 리턴값으로 리턴 할 수 있어야 함

### 람다식이란?

- 람다는 값을 다룰 수 있는 익명함수 표현방법

### 고차함수란?

- 고차함수는 **함수**를 인자(Parameters)로 삼거나 반환(return)하는 함수이다.
  1. Can take functions as parameters : 함수를 파라미터로 취할 수 있다.
  2. Can return a function : 함수를 반환 할 수 있다.

- 즉 코틀린의 함수는 일급객체이기 때문에 High-Order Function을 사용 할 수 있다.

### 그렇다면 High-Order Function은 자바에서는 어떻게 동작할까??

1. 코틀린에서의 High-order Function

![image-20200131190140429](C:\Users\USER\AppData\Roaming\Typora\typora-user-images\image-20200131190140429.png)

2. Java 코드로의 변환 (kotlin code -> kotlin bytecode -> decompile java)

![image-20200131190236149](C:\Users\USER\AppData\Roaming\Typora\typora-user-images\image-20200131190236149.png)


3. 일급객체의 개념이 존재하지 않기 때문에 interface를 활용하여 invoke하여 사용하는 것을 볼 수 있다.

![image-20200131190255271](C:\Users\USER\AppData\Roaming\Typora\typora-user-images\image-20200131190255271.png)
