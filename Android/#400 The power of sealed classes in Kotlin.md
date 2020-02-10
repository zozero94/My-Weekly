# [The power of sealed classes in Kotlin](https://www.lordcodes.com/articles/the-power-of-sealed-classes-in-kotlin)

## (Android Weekly #400)

- 열거형인 Enums는 여러 프로그래밍 언어로 존재해왔다. 그러면서 제한된 값 집합에서 값을 얻는 유형으로 나타낼 수 있다.
- 코틀린은 이 개념을 받아 제한된 계급으로 알려진 훨씬 더 강력한 클래스로 진화했다.



> Sealed classes는 편지봉투와 같다. 



- Enum class는 상수 값의 집합으로 구성되며 인스턴스에는 이러한 상수 중 하나가 할당됨.
- Sealed class는 대신 하위 클래스들의 집합에 의존하여 각 하위 클래스들의 여러 인스턴스를 가질 수 있고, 그자체 상태를 가질 수 있다.


 > Sealed class sub-classes can carry their own state
 >
 > Sealed class의 하위 클래스는 그들 자신의 상태를 가질 수 있음



## TODO

- 사용자 계정이 어떤 상태에 있는지 볼 수 있는 ``AuthenticationState``가 있다고 가정하자.

1. 사용자 식별로 로그인
2. 저장된 자격 증명을 사용하여 로그아웃
3. 완전히 로그아웃

```kotlin
sealed class AuthenticationState {
  data class SignedIn(val userGuid: UUID) : AuthenticationState()
  data class StoredCredentials(val credentials: Credentials) : AuthenticationState()
  object SignedOut : AuthenticationState()
}
```



- enum과 다르게 sealed class는 body안에 하위 클래스들을 유지 할 필요가 없다.

오직 같은 파일이면 된다.

```kotlin
// AuthenticationState.kt

sealed class AuthenticationState
data class SignedIn(val userGuid: UUID) : AuthenticationState()
data class StoredCredentials(val credentials: Credentials) : AuthenticationState()
object SignedOut : AuthenticationState()
```



## When

Sealed class는 When 조건문이랑 사용될때 가장 강력하다.

컴파일러는 배정받거나 반환되는 결과를 결정하여 핸들링한다. 따라서 else 분기가 필요없다.

```kotlin
fun onAuthenticationStateChanged(newState: AuthenticationState) = when (newState) {
  is AuthenticationState.SignedIn -> showSignedIn(newState.userGuid)
  is AuthenticationState.StoredCredentials -> showSignedIn(newState.credentials)
  AuthenticationState.SignedOut -> showSignedOut()
}
```

> The compiler can determine if all cases have been handled
>
> 컴파일러는 모든 케이스가 처리되었는지 결정 할 수 있다.



