# [The Power of Types for Errors](https://androidweekly.us2.list-manage.com/track/click?u=887caf4f48db76fd91e20a06d&id=f29b7f4474&e=7b5d1e4925)

## (Android Weekly #399)



```kotlin
fun onError(type: ErrorType, details: String? = null) {
  // ...
}
```

우리는 보통 에러를 처리하는 함수를 이렇게 만든다

```kotlin
enum class ErrorType {
   IO,
   PASSWORD_INVALID,
   RECAPTCHA_FAILED
// ...
}
```

그리고 enum class를 만들어 에러타입을 지정하고 그 에러의 상세 내용을 details에 엮게 된다.

두 Parameter은 독립적이며 굉장히 강력히 엮여있고 (tight coupling), 두 객체사이 관계를 공유할 수 없다.



### 이 관계의 표현

두 객체로 나뉘어진 파라미터를 하나로 합쳐보자.

```kotlin
data class Error(val type: ErrorType, val details: String?)
Our tracking method now looks much nicer:
fun onError(error: Error) {
  // ...
}
```

이렇게 된다면 한가지 파라미터만 있을 것이고, 읽는자로 하여금 무엇을 넘겨야 하는지 분명히 해준다.

그리고 두개의 독립된 객체를 가졌을 때 처럼, 잘못된 무언가를 통과시킬 수 없다.



### 더 잘 만들어 보기

두 파라미터는 간단하게 Error class로 이동되었지만 아직 숨겨진 관계가 존재한다.

그래서 이것을 개발자가 언제 무엇을 통과해야 하는지 **추측하게 해서는 안되도록** 표현해야 한다.



enum class는 이런 개인적인 세부사항을 포함할 수 없다. 정의상 불변의 상수일 뿐이다.

하지만, sealed class는 보다 정확하게 모델링이 가능하다.



```kotlin
sealed class Error {
  object InvalidPassword : Error()
  object RecaptchaFailed : Error()
  class IOError(val details: String): Error()
// ...
}
```



### 다음 스텝

enum class는 계층구조를 지원하지 않는다. 

enum 값들은 추상화 한 수준에서 평평하다.

보상하기 위해 우리는 일반적으로 이러한 이름을 사용한다.

```kotlin
enum class ErrorType {
   IO,
   EMAIL_DENIED,
   EMAIL_EXISTING,
   EMAIL_INVALID,
   PASSWORD_INVALID,
   RECAPTCHA_FAILED
}
```

>  하지만 이름을 작성할 때, 어떤 타입이 될수 있는 이름을 작성하지 마라.

이제 enum 을 무시하면 계층구조를 설명하는것은 쉽다.

```kotlin
sealed class Error {
  sealed class SignInError {
     sealed class EmailError : SignInError() {
        object Denied : EmailError()
        object Invalid : EmailError()
        object Existing : EmailError()
     }
     class IOError(val message: String): SignInError()
  }
//...
}
```

이메일 관련 오류를 로그인 사용 사례로 제한할 수도 있다.

이러면 한 지점에서 하나의 사용 사례와 관련된 모든 오류를 처리할 수 있다.



```kotlin
with(BehaviorSubject.create<mError>()) {
        observeOn(Schedulers.newThread())
            .subscribe {
                when (it) {
                    is mError.SignInError.SocialError.GoogleError ->
                        println(it.message)
                    is mError.SignInError.SocialError.FacebookError ->
                        println(it.message)
                }
            }
    }
```



### 결과

- 파라미터가 함께 속하지만 별도로 처리되는 파라미터가 보인다면 숨겨진 타입이 있는지 확인한다.
- 특정한 경우에만 사용되는 필드가 있는 클래스를 보면 숨겨진 유형의 계층이 있을 수 있다.
- 특정 조합만 허용하는 방법이 보이면 유형으로 타입으로 고정하라.

