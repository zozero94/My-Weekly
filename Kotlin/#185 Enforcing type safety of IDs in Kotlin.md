# [Enforcing type safety of IDs in Kotlin](https://www.lordcodes.com/articles/enforcing-type-safety-of-identifiers-in-kotlin)

## Kotlin Weekly #185



1. 일반적인 타입 (String, Long)
2. Tiny Type (UUID 등 기본적인 타입, 제네릭)
3. data class
4. type aliases



객체에 식별자, ID 형식이 필요한 경우는 매우 흔하다. 

원격 API나 데이터베이스 통신을 할 때 엔티티를 참조해야 하는 경우가 많다.

String, UUID, Long 과 같은 적절한 형태의 ID 속성을 클래스에 제공함으로써 쉽게 구현 할 수 있다.



**ID에 그들의 타입을 줌으로써 타입안정성을 강화하는 아이디어를 탐구해보자.**



### General types

ID의 옵션중 하나는 String 타입을 사용하는 것이다. (특히 기대되지 않은 포멧을 사용할때)

또 다른 옵션 중 하나는 UUID이다. 구현은 Java의 표준 라이브러리에 의해 제공된다.

UUID는 ID 저장을 위해 설계되었고, 128bit의 값으로 문자열과 쉽게 변환할 수 있다.



```kotlin
data class Team(val id: UUID, val size: Int)

data class TeamMember(val id: String, val name: String)
```



### Tiny Type (작은 타입)

오히려 일반적인 유형에 의존하기 보다는 쉽게  전용 유형을 만들 수 있다.

```kotlin
data class Identifier(val rawValue: UUID)
```



다른 원시타입을 필요로 한다면, 다른 원시 타입을 포함하는 추가 ID 타입을 만들거나 일반 타입 매개변수를 가진 단일 ID 유형을 사용 할 수 있다.

**제네릭**을 사용하면 특정 유형의 ID를 필요로 하는 속성이나 함수 인자를 가질 수 있다.

```kotlin
data class Identifier<RawT>(val rawValue: RawT)
```



우리는 일반적인 원시타입에 ``Identifier`` extentions 를 사용해서 일반적인 String 값을 뽑아 낼 수 있다.

```kotlin
val Identifier<UUID>.uuidString: String
    get() = rawValue.toString()
```



### Ensuring the correct type (올바른 타입 확인)

여기에 일반적인 타입을 내부적으로 사용할때 나타날 수 있는 몇가지 이슈가 있다.

Say Team은 ``UUID`` ID 를 가지고 있으며, ``Member``는 ``String`` ID를 가지고 있다.

- 모든 팀 ID는 UUID 이지만 모든 UUID가 인증된 팀을 가리키는 것은 아니다.
- 멤버 ID는 서로 다른 엔티티를 참조하더라도 팀 ID를 지원하는 원시 문자열과 동일한 값을 가질 수 있다.

동일한 원시 유형을 사용하는 두개의 서로 다른 ID를 혼합하고 일치시키는 것은 완벽하게 유효하며

컴파일러에 의해서 방지되지 않는다.



``Identifier``의 두 번째 제네릭 변수 파라미터를 추가함으로써 특정 엔티티에 대한 사용을 제한 할 수 있다.



```kotlin
data class Identifier<EntityT, RawT>(
    val rawValue: RawT
)

data class Room(val id: Identifier<Room, UUID>)
data class Meeting(val id: Identifier<Meeting, UUID>)

fun bookMeeting(id: Identifier<Meeting, UUID>) {}

// ❌ Compile error: Type mismatch.
bookMeeting(room.id)
```

Entity T 는 ``Identifier`` 내에서 실제로 사용되지 않기 때문에, ``@Support("Unused")``를 사용하여 무시할 수 있는 사용중이라는 경고를 받을 가능 성이 있다.



### Type aliases (타입 별칭)

우리의 codebase에 다양한 엔티티들이 있을 때, ID의 수는 증가 할 것이고 우리는 식별자 서명을 타이핑하는 것에 화가 난다.

이 작업을 단순화 하기 위하여 타입 별칭에 의존 할 수 있다.

좋은 조직의 탑은 그들의 identify의 엔티티에 속한 것들을 보관함으로써, 쉽게 찾을 수 있도록 하는 것이다.

```kotlin
// Team.kt

typealias TeamId = Identifier<Team, UUID>

data class Team(val id: TeamId)
```

함수와 프로퍼티에 ID를 지정하는것이 훨씬 더 간단해졌다.

```kotlin
fun Team.inviteMember(id: MemberId) {}

// ❌ Compile error: Type mismatch.
team.inviteMember(team.id)

// ✅ Compiles.
team.inviteMember(member.id)
```



### Alternatives (대책)

만약 우리가 제네릭 타입의 매개변수를 피하는것을 원한다면, 우리는 코틀린의 데이터 클래스의 간결성에 의존할 수 있고, 각 ID에 대한 분리 된 작은 타입을 가질 수 있다.

```kotlin
data class MessageId(val raw: UUID)
data class ChatId(val raw: String)
data class PersonId(val raw: Long)
```

- 장점은 기본적인 데이터 클래스이기 때문에 훨씬 더 간단하다는 것이다. 심지어 inline class 일수도 있다.

- 단점은 우리는 Identifier를 사용하여 자동으로 다른 타입으로 변환하는 코드를 쓸 수 있다.
  예를들어 Bundle에 저장하여 액티비티로 보내거나, Room이 SQLite 데이터베이스에 저장할수 있는 형식으로 바꿀 수 있다.



### Wrap up (감싸다)

우리의 ID엔티티의 안정성을 강화하여 사용 하는것은 우리는 일함에 있어 코드를 더 안전하게 작업할 수 있고, 잘못된 ID를 사용함으로써 생기는 버그를 피할 수 있다.

우리의 코드 또한 엔티티가 언급하는 ID를 볼때 더 읽기 쉽다.

게다가 이 문제를 더욱 강력하게 만들기 위해 확장하는 방법도 있다.

