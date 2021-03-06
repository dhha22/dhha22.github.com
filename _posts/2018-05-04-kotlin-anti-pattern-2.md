---
layout: post
title: Kotlin Anti Pattern #2
date: 2018-05-04
comments: true 
thumbnail: "assets/img/thumbnails/2018_droid_kaigi_1.png"
image: "assets/img/thumbnails/2018_droid_kaigi_1.png"
categories: Android
tags: [Android, Kotlin]
---



> 해당 글은 Droid Kaigi 2018에서 발표했던 Kotlinアンチパターン을 번역한 글입니다.

------



### Kotin Anti Pattern #2 에서 다뤄 볼 주제

- Data Class


- interface 와 abstract class


- lazy 와 custom getter
- Fragment 와 lazy
- custom getter
- custom setter

> [Kotlin Anti Pattern #1] : https://dhha22.github.io/android/2018/03/05/kotlin-aniti-pattern-1.html

------



### Data Class

- equals/ hashCode 및 toString이 override 되어있음
- componentN 이나 copy등의 메서드를 제공

```kotlin
data class User(val name: String, val age: Int)

val alice = User("Alice", 27)
alice == User("Alice", 27)	// true
alice.toString()	// "User(name=Alice, age=27)"
val (name, age) = alice
val nextYear = alice.copy(age = 28)
```

------



#### Data Class Anti Pattern

인스턴스 생성을 위한 메서드에서 제약을 보증하고 싶을 경우

```kotlin
data class Range(val min: Int, val max: Int) {
  companion object {
    fun getRange(a: Int, b: Int) : Range {
      return Range(minOf(a, b), maxOf(a, b))
    }
  }
}
```

data class에는 **copy 메서드**가 제공됩니다.

→ 임의의 데이터를 가진 인스턴스 생성 가능

```kotlin
val range = Range.getRange(3, 5)
val illegal = range.copy(max = 0)
// ↑ Range (min = 3, max = 0)로 되고, 제약이 깨짐
```

------



#### 해결 방법

값이 결정되었다라고 해서 data class를 사용하지 않습니다.

 **「값의 내부 표현」 == 「클래스에 예상되는 행동」**

→ 값의 정리에 이름을 붙이고 있을 뿐

→ data class



**「값의 내부 표현」 != 「클래스에 예상되는 행동」**

→ 일반 클래스로 사용하는 편이 좋음

------



### interface 와 abstract class

Kotlin에서 interface의 디폴트 메서드를 사용할 수 있습니다.

Java8에도 있었던 기능이지만 안드로이드에서 사용하기 위해서는 **minSdkVersion을 24 이상**으로 할 필요가 있습니다.

```kotlin
interface Downloadable {
  fun download() {
    // 공통 처리 등
  }
}
```

------



#### abstract class 와 뭐가 다를까?

|          |  interface   |   abstract class   |
| :------: | :----------: | :----------------: |
|   상태   | 가질 수 없음 |    가지고 있음     |
|   상속   |  interface   | class 와 interface |
| 다중상속 |     가능     |       불가능       |

|           | default method | class method |
| :-------: | :------------: | :----------: |
|   final   |     불가능     |     가능     |
| protected |     불가능     |     가능     |

------



#### interface 와 abstract class Anti Pattern

Java 때부터 잘 알려져 있던 Anti Pattern 중 하나입니다.

처리를 공통화 하기 위해 Base 클래스에 여러 메서드를 선언하다 보니 클래스가 점점 비대해 집니다.

```kotlin
abstract class BaseActivity : AppCompatActivityO() {
  protected fun showNetworkError() {
    // 에러표시 (에러가 없는 페이지도 있음)
  }
}
```

결국 수천 줄을 가진 클래스가 되고 더이상 걷잡을수 없게 됩니다.

------



#### 해결방법

Java 에서는 상속보다 **위임**이 더 좋은 방법이라고 말하고 있습니다.

Kotlin에서는 interface의 **디폴트 메서드**를 사용하는것이 좋습니다.

```kotlin
interface NetworkErrorView {
  // 오류에 대한 View 및 새로고침 처리는 자식 클래스에서 제공
  val networkErrorView: View
  fun onClickReload()
  
  fun showNetworkError() {
    // 리스너 세팅, 오류 표시 등의 일반적인 처리
  }
}
```

물론 abstract 클래스가 올바르게 사용되면 「Base 클래스 = 나쁨」 은 아닙니다.

##### abstract 클래스 사용법의 올바른 예

- onCreate 등의 상속 부분에서 처리를 공통화 하고 싶을 경우
- 클래스 외부 부터 호출하지 않고 싶을 경우
- 자식 클래스에서 override 당하고싶지 않은 경우



##### 디폴트 메서드 구현이 올바른 예

- 전술 이외의 처리를 공통화 시키고 싶은 경우
- 다중 상속하고 싶은 경우, 상태를 가지고 싶은 경우도 **class delegation**을 사용하면 가능합니다.

```kotlin
class UserModel() : DefaultImpl by State() { ... }
class UserModel(s: State) : DefaultImpl by s { ... }
```



```kotlin
interface IState {
  var state: String
}

interface DefaultImpl : IState {
  fun abstract() : String
  fun default() { ... }
}

class Staet : IState {
  override var staet: String = ""
}

class UserModel : DefaultImpl, IStaet by State() {
  override fun abstract() : String {
    return "concrete"
  }
}
```

------



### 최상위 함수 및 확장 함수

#### 최상위 함수

파일에 직접 함수를 쓸 수 있습니다.

```kotlin
fun throwOrLog(t: Throwable) {
  // 개발은 throw, 릴리즈는 log 전송
}
```



그러면 어디서나 호출 할 수 있습니다.

```kotlin
fun foo() {
  throwOrLog(e)
}
```

------



#### 확장함수

클래스에 함수를 추가 하는것이 가능합니다.

```kotlin
fun Any?.log() {
  Log.d("DEBUG", this.toString())
}
```



여기서도 어디서든 호출 할수 있게 됩니다.

```kotlin
fun foo() {
  123.log()
  "hello".log()
}
```

------



#### 최상위 함수 및 확장 함수 Anti Pattern #1

일반성이 없거나 별로 사용하지 않은 메서드를 추가할 경우

```kotlin
fun String.decorate() = "david"+$this+"ha"
// ↑"decorate "의 의미가 넓고, 다양한 상황에 고려되지 않음
```



#### 최상위 함수 및 확장 함수 Anti Pattern #2

공식으로 있음직한 이름인데 엉성하게 구현되어 있을 경우

```kotlin
fun String.isInteger() = this.all { it in '0'..'9' }
// ↑ 비어있는 문자열, 음수가 고려되지 않음
```



- 확장 기능이 편리해서 남발하는 경향
- 라이브러리 보다 버그가 있을 가능성이 높음
- Java로 Util 클래스를 만드는 것과 같은 문제지만 Kotlin 이라고 통상의 함수처럼 보이기에는 피해가 큼
- 같이 일하는 팀에 개발자가 많으면 특히 문제가 됨

------



#### 해결 방법

팀에서 확장 함수를 만드는 기준을 정렬하고 interface의 기본 구현 범위를 한정합니다.

```kotlin
interface BookmarkableActivity {
  fun bookmark() {
    // 일반적인 bookmark 처리
  }
   
  fun String.toLabel() = "★ $this"
}
```

------



### lazy 와 custom getter

#### lazy에 대해서

lazy는 처음 해당 변수에 접근했을때 값이 계산되고 이후 그 값을 반환합니다. (delegated 속성 중 하나)

```kotlin
private val userId by lazy {
  intent.getStringExtrat("USER_ID")
}
```

------



#### 일반적인 표현식의 custom getter

- 일반적인 대입은 **인스턴스 생성시**에 계산

  ```kotlin
  val isAdmin = userId == ADMIN_ID
  ```

  

- lazy는 **처음 접근 했을 시**에 계산

  ```kotlin
  val isAdmin by lazy { userId == ADMIN_ID}
  ```

  

- custom getter은 **접근 할 때** 마다 계산

  ```kotlin
  val isAdmin get() = userId == ADMIN_ID
  ```

------



#### lazy와 custom getter Anti Pattern

일반적인 대입, lazy, custom getter를 모호하게 구분할 경우

```kotlin
// 충돌한 경우
private val button = findViewById<Button>(R.id.button)

// 오래된 값이 돌아온 경우
val area by lazy { width * height }

// 불필요한 계산을 여러번 실행할 경우
val user: User get() = User(userID)
```

------



#### 해결 방법

값의 성질에 따라 적절히 클래스를 나눈다

- 클래스 초기화 시에 값이 확정되고 불변 → 일반 대입
- 어느 시점을 넘기전 까지 값이 없지만 불변 → lazy
- 상태가 바뀌면 값이 바뀔 경우 → custom getter

------



#### delegated 속성

속성의 get과 set을 다른 클래스에 위임이 가능할 때

```kotlin
var userId: String by MyClass()
```

→ 일반적인 값으로 보이기 쉬움

- Intent의 extra
- Fragement의 arguments
- savedInstaneState
- SharedPreferences
- FirebaseRemoteConfig



delegated 속성의 사용 예

```kotlin
class Extra<out T> : ReadOnlyProperty<Activity, T> {
  override fun getValue(
  	thisRef:Activity,
    property: KProperty<*>
  ): T = thisRef.intent.extras.get(property.name) as T
}

fun Intent.put(
  prop: KProperty1<*, String>, value: String
): Intent = this.putExtra(prop.name, value)
```



```kotlin
class ProfileActivity : AppCompatActivity() {
  val userId: String by Extra()
  
  companion object {
    fun createIntent(context: Context, userId: String): Intent {
      return Intent(context, ProfileActivity::class.java)
      			.put(ProfileActivity::userId, userId)
    }
  }
}
```

------



### Fragment 와 lazy

lazy는 처음 접근할때에 값이 계산되고, 이후 그 값을 캐시하고 반환합니다.

```kotlin
private val userId by lazy {
  intent.getStringExtra("USER_ID")
}
```

------



#### Fragment 와 lazy Anti Pattern

Fragment의 View를 lazy 하는것은 Anti Pattern입니다.

```kotlin
val button by lazy {
  view!!.findViewById<Button>(R.id.button)
}
```

Fragment는 같은 인스턴스에서 **onCreateView**가 다시 불립니다.

lazy를 사용하면 처음의 view 상태를 계속 유지하기 때문에 **onCreateView**가 되어도 값이 업데이트 되지 않습니다.

------



#### 해결 방법

lateinit을 사용하여 onCreateView에서 대입합니다.

```kotlin
lateinit var button: Button

override fun onCreateView(...): View? {
  val view = ...
  button = view.findViewBtId(R.id.button)
  return view
}
```

Data Binding의 경우에도 binding 자체는 lateinit가 좋습니다.

------



### custom getter

val 과 var의 반환 값을 커스텀하는것이 가능합니다.

문법상, 인수가 없는 함수는 모두 custom getter로 됩니다.

```kotlin
var userId: String = ""
val isAdmin get() = userId == ADMIN_ID
```

------



#### custom getter Anti Pattern #1

값을 가져오는 도중 사이드 이펙트가 발생합니다.

```kotlin
private val itemCount: Int
get() {
  if(recyclerView.adapter == null) {
    initXXX()	// 사이드 이펙트
  }
  return recyclerView.adapter.itemCount
}
```



#### custom getter Anti Pattern #2

계산량이 많습니다.

```kotlin
private val total: Int
get() = countView(root)

fun countView(view: View): Int = if(view is ViewGroup) {
  (0 until view.childCount).map {
    countView(view.getChildAt(it))
  }.sum()
} else {
  1
}
```



호출 측면에서 일반 변수처럼 보입니다.

따라서 상태가 변화하거나 계산량이 많은것으로 예상할 수 없어서 버그와 성능 저하를 일으킬 수 있습니다.

```kotlin
if (this.itemCount > 20) {
  // ↑ 위에 처럼 순진하게 접근할 수 도 있음
}
```

------



#### 해결방법

사이드 이펙트가 없고 계산 과정이 간단할 경우 → custom getter

사이드 이펙트가 있거나 계산량이 많을 경우 →  함수

위의 분류대로 구현하면 함수 이름보다 명확하게 호출 측이 사이드 이펙트의 유무와 계산량을 예상할 수 있습니다.

------



### custom setter

Kotlin에서 일반 var를 커스텀화 할 수 있습니다.

```kotlin
var text: String = ""
set(value) {
  field = value
  notifyDataSetChanged()
}
```

------



#### custom setter를 사용하는 곳

- 값을 설정하는 김에 갱신 처리 등을 실시
- 변수의 위임

------



#### 변수의 위임

멤버 변수에 위임을 간단하게 사용할 수 있습니다.

(이 경우에 backing field가 생성되지 않습니다.)

```kotlin
private val owner = Person()

var ownerName: String
get() = owner.name
set(value) {
  owner.name = value
}
```

※ backing field : Java로 변화 했을 경우 생성되는 일반적인 field 입니다.



다소 중복이 있을 수 있지만 다음과 같이 쓸 수 있습니다. 

```kotlin
private val owner = Person()

var ownerName: String by object : ReadWriteProperty<Any, String> {
  override fun getValue(thisRef: Any, property: KProperty<*>): String = owner.name
  override fun setValue(thisRef: Any, property: KProperty<*>, value: String) {
    owner.name = value
  }
}
```

------



#### 변수의 위임 Anti Pattern

예: 값이 업데이트 되면 XXX를 업데이트 하고 YYY에 통지하고 ZZZ 값도 업데이트 해야하는 경우

```kotlin
var person: Person = Person()
set(value) {
    field = value
    updateXXX()
    notifyYYY()
    ZZZ = value.name
}
```

값만 바꿀 수 없습니다 (심지어 클래스 안에서도)

반영과 계산을 나중에 한꺼번에 하는것은 불가능합니다.

값을 대입했을 뿐이라고 생각하면 예외의 상황이 발생합니다.

------



#### 해결 방법

당연하지만 var의 책임은 값의 유지에 두어야 합니다.

var는 일반적으로 private으로 가지고 있어야합니다.

갱신용 함수로 공개하는 편이 var로 표현하는 것 보다 혼란이 적습니다.



> [출처] : https://www.slideshare.net/RecruitLifestyle/kotlin-87339759

