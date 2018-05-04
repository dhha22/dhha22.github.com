---
layout: post
title: Kotlin 안티패턴#1
date: 2018-03-05
comments: true 
thumbnail: "assets/img/thumbnails/kotlin.png"
categories: Android
tags: [Android, Kotlin]
---

## Kotlin 안티 패턴 #1

> 해당 글은 Droid Kaigi 2018에서 발표했던 Kotlinアンチパターン을 번역한 글입니다.

------



### Kotin 안티패턴 #1 에서 다뤄 볼 주제

- lateinit 과 null 초기화


- Scope 함수


- Nullable 과 NonNull

------

### lateinit 과 null 초기화

#### lateinit에 대해서

Java에서는 초기값을 설정하지 않고 변수 선언을 했을 경우 **null** 이 들어갑니다.  

```java
TextView message;	// null 이 들어갑니다.
```

   

Kotlin에서는 기본적으로 초기값을 설정하지 않으면 안됩니다.  

```kotlin
var message: TextView	// error가 발생됩니다.
var message: TextView? = null	// OK
```

 

Android에서는 onCreate() 이후로 초기화 하는것이 많습니다. 

Kotlin에서 **초기값을 설정하는게 필수**입니다.

아래 예시는 TextView를 null 로 초기화 하였기 때문에

실질적으로 **NonNull**이었던 부분이 전부 **Nullable**로 되어버립니다.  

```kotlin
var message: TextView? = null
fun clear() {
  message!!.text =""
  message?.text = ""
}
```



 Kotlin에서 초기값을 설정하는게 필수지만 **lateinit**을 사용하면 선언할 때 초기값을 넣지 않아도 됩니다.  

```kotlin
lateinit var message: TextView	// OK
```

단 값을 초기화 하지 않은 상태에서 해당 값을 참조하게 되면 **UninitializedPropertyAccessException**이 발생됩니다.

------



#### lateinit을 사용하는 곳

인스턴스 생성할 때 값이 정해지지 않았지만 onCreate(), onCreateView()에 대입해야할 경우 사용하면 됩니다.

예시)

- findViewById를 한 View
- DataBinding의 binding
- Dagger등의 DI에 의해 inject 될 것

단 **원시 타입**(int, float, long, ...)이나 **Nullable**에는 **lateinit**을 사용할 수 없습니다.

------



#### lateinit 안티패턴

통신 후 얻는 정보를 **lateinit**로 할 경우

```kotlin
lateinit var profile: Profile

fun init() {
    fetchProfile().subscribe { profile ->
        this.profile = profile
    }
}
```



미리 listener를 설정하면 

통신 중이거나 통신 에러시에 접속해 **UninitializedPropertyAccessException**이 발생됩니다.

```kotlin
button.setOnClickListener {
    textView.text = profile.name
}
```

------



#### 해결방법

**Nullable**로 하고 항상 "값을 못받았을 경우 어떻게 할까?" 라는 생각을 하는게 좋습니다.

```kotlin
var profile: Profile? = null	// Nullable 설정
fun init() {
    fetchProfile().subscribe { profile ->
        this.profile = profile
    }
}
```

onCreate() / onCreateView() 에서 초기화가 가능한다면 **lateinit**을 사용해도 됩니다.

onCreate() / onCreateView() 이후에 값이 정해지면 Nullable로 하거나 멤버 변수를 사용하는것을 피하시는게 좋습니다.

------



#### isInitialized

Kotlin 1.2부터 **isInitialized**가 추가되었습니다.

값이 대입되었는지 여부를 확인 할 수 있습니다.

```kotlin
lateinit var str: String

fun foo() {
    val before = ::str.isInitialized	// false
    str = "hello"
    val after = ::str.isInitialized		// true
}
```



isinitialized는 원래 테스트 코드에서 이용하려고 추가되었습니다.

isInitialized를 일상적으로 사용하면 더이상 null이 안전하지 않으므로 프로덕션 코드에서 사용 안하시는게 좋습니다.

```kotlin
private lateinit var file: File

@After
fun tearDown() {
    // 파일 만들기 전에 fail하면 에러가 발생
    file.delete()
}
```



------

### Scope 함수

scope 함수 종류에는 let/ run/ also/ apply/ with 5개가 있습니다.

with를 제외하면 대략적 다음과 같이 분류 할 수 있습니다.

```kotlin
반환값 = 자신.scope함수 {
    리시버.method()
    결과
}
```



![scope_method_1](/assets/img/kotlin/scope_method_1.png)

------



#### Scope 함수를 사용하는 곳 #1

null 관련 제어에 유용합니다.

```kotlin
str?.let {
    //	str이 null이 아닐 경우
}

val r = str ?: run {
    // str이 null인 경우
}
```



#### Scope 함수를 사용하는 곳 #2

초기화 처리를 정리하고 싶을 경우 사용하면 좋습니다.

```kotlin
val intent = Intent().apply {
    putExtra("key", "value")
    putExtra("key", "value")
    putExtra("key", "value")
}
```

------



#### Scope 함수 안티패턴

apply 범위 내에서 프로퍼티 접근 형식을 사용할 때

```kotlin
val button = Button(context)
button.text = "hello"	// Java의 setText(..)가 호출됨
// ... button의 설정이 이어짐 ...

↓ apply를 사용하여 설정을 정리

val button = Button(context).apply {
    text = "hello"
    // ... button의 설정이 이어짐 ... 
}
```



로컬 변수를 정의하면 접근 범위가 바뀝니다.

```kotlin
fun init() {
    var text = ""	// text 라는 변수가 추가 될 경우
    val button = Button(context).apply {
        text = "hello"
    }
    button.text	// "hello" 가 되지 않음
}
```

------



#### 해결방법 #1

apply 범위 내에서는 프로퍼티 접근 형식을 사용하지 않고 일반 함수를 호출하는 방법

```kotlin
val button = Button(context).apply {
    setText("hello")
}
```

단 로컬 함수(setText와 이름이 동일한 함수) 가 정의되어 있을 경우 같은 문제가 발생합니다.



#### 해결방법 #2

this를 필수로 사용하는 방법 (also와 비슷한 느낌)

```kotlin
val button = Button(context).apply {
    this.text = "hello"
}
```



#### 해결방법 #3

apply를 사용하는 대신 also를 사용하는 방법 

```kotlin
val button = Button(context).also {
    it.text = "hello"
}
```



------

### Nullable 과 NonNull

Nullable/ NonNull은 Kotlin의 큰 매력 중 하나 입니다.

```kotlin
val nullable: String? = null	// OK
val nonNull: String = null	// 잘못됨

nullable.length	// 잘못됨
nullable!!.length	// OK
nullable?.length	// OK
nonNull.length	// OK
```

------



#### Nullable 과 NonNull 안티패턴 #1

Nullable 그대로 데이터를 사용하게 되면 

```kotlin
data class User(
    val id: Long = null,
    val name: String? = null,
    val age: Int? = null
)
```

Nullable 데이터를 사용할 때 곳곳에서 null 체크를 하거나 safe call를 하는 처지가 됩니다.

```kotlin
return team?.user?.name?.length
```

→ 실질적으로 조건 분기가 늘어나 파악하기 어려워 집니다.

→ "뭐가 null이 였지?"에 대해 매번 생각하게 됩니다.



#### Nullable 과 NonNull 안티패턴 #2

모든 변수를 NonNull로 하기 위해 잘못된 데이터를 집어 넣으면

```kotlin
val response = ...

return User(
    id = respon.id ?: 0L,
    name = response.name.orEmpty(),
    age = response.age ?: 0
)

if(user.name.isNotEmpty()) {
    // 골치 아픔 or 잊어버림
}
```

→ 잘못된 데이터인지 확인하는 처지가 됩니다.

→ 체크를 잊어 오류가 생깁니다.

------



#### 해결방법

"null이 무엇을 나타내는지" 로 처리하는 레이어를 정해야합니다.

- API에서 전달해주지 않았기 때문 → API 레이어에서 처리
- 사용자가 설정하지 않았기 때문 → UI 레이어에서 처리
- 명백하지 않고 레이어를 넘었을 경우 → 클래스에서 명확화

데이터 표현에 대해 곤란했을 경우 null에 의존하지 않습니다. (예외 던지기, 다른 클래스로 하기 ,...)



> [출처] : https://www.slideshare.net/RecruitLifestyle/kotlin-87339759

