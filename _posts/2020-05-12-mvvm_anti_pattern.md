---
layout: post
title: MVVM Anti Pattern
date: 2020-05-12
comments: true 
thumbnail: "assets/img/thumbnails/mvvm_anti_pattern.jpeg"
categories: Androidgi
image: "assets/img/thumbnails/mvvm_anti_pattern.jpeg"
tags: [Android, MVVM, Architecture]

---

> MVVM Architecture을 적용할 때 피해야할 패턴에 대하여 다뤄보도록 하겠습니다.



#### 1. ViewModel안에 Android Framework가 있어도 될까?

![image1_1](/assets/img/mvvm_anti_pattern/image1_1.png)

**ViewModel**에는 **Android Framework** 존재를 알아서는 안됩니다.

Android Framework에 종속성을 가지지 않아야 **lightweight unit test**를 진행할 수 있게 되어 **leak safety**와 **modularity**가 향상됩니다.

종속성을 가지고 있는지에 대해 쉽게 확인 할 수 있는 방법은 ViewModel 안에 **android.*** 에 관한 패키지의 import 여부에 대해 확인하시면 됩니다.

------

#### 2. ViewModel안에 View를 참조해도 될까?

ViewModel은 Activity/Fragment 보다 lifecycle이 깁니다.

![image1_2](/assets/img/mvvm_anti_pattern/image1_2.png)

Activity/Fragment가 **destroy** 되고 **recreate** 되었을 경우 기존에 존재하던 ViewModel을 가지게 됩니다.

ViewModel은 새로운 Activity/Fragment를 참조하고있는게 아니라 

**destroyed된 Activity/Fragment를 참조**하고 있기 때문에 **memory leak**이 일어나거나 **crash**가 발생하게 됩니다.

View 뿐만 아니라 **Activity의 context를 참조하고 있는 class**도 참조해서는 안됩니다.

그러면 ViewModel에서 View랑 통신하고 싶을때는 어떻게 해야할까요?

ViewModel안에 정의된 LiveData를 활용하여 **observer pattern**을 이용하면 됩니다.

![image1_3](/assets/img/mvvm_anti_pattern/image1_3.png)



밑에 코드는 ViewModel에 정의 된 **reviewList(LiveData)**를 **observe**하여 데이터 값이 변할 때 마다

Adapter 값을 update 하여 View가 upate 되는것을 확인할 수 있습니다.

```kotlin
viewModel.reviewList.observe(this, Observer {
    (adapter as? ReviewAdapter)?.apply {
        items = it
        notifyDataSetChanged()
    } ?: init(this, it, viewModel)
})
```

------

#### 3. ViewModel에서 MutableLiveData의 접근 제한자를 public으로 설정해도 될까?

우선 **MutableLiveData**와 **LiveData**의 차이점은 값을 변경할 수 있는지의 차이 입니다.

ViewModel에 정의된 MutableLiveData가 public으로 선언된다면

```kotlin
class TestViewModel : ViewModel() {
  private val _reviewList = MutableLiveData<Review>()
  
  val reviewList : LiveData<Review>
  	get() = _reviewList
  
  fun loadData() {
    _reviewList.value = loadReviews()
  }
}
```

Activity/Fragment에서 LiveData 값을 변경 할 수 있기 때문에 이러한 행동은 MVVM Architecture에 위배되는 행동입니다.

View는 오로지 LiveData를 observe만 해야하기 때문에 

ViewModel에 선언된 MutableLiveData를 **getter**나 **backing properties**를 이용하여 **캡슐화**를 해야합니다.

------

#### 4. Activity/Fragment에 로직이 있어도 될까?

Activity/Fragment에 if문이라던지 for문 등 복잡한 로직이 존재하면 안됩니다.

이러한 로직은 ViewModel안에 아니면 다른 layer에 존재해야 합니다.

view는 일반적으로 unit test가 아니므로 view에 존재하는 로직의 유효성을 검사하기 어렵습니다.

그러므로 **Activity/Fragment에는 최소한의 로직**만 있어야합니다. 

------

#### 5. Activity/Fragment에 여러개의 ViewModel을 가져도 될까?

[우선 Google에서 권장하는 방식은 **하나의 ViewModel만을 사용하는것**을 권장](https://www.youtube.com/watch?v=Ts-uxYiBEQ8&feature=youtu.be&t=8m40s)하고 있습니다.

상황에 따라서 여러개의 ViewModel을 사용할 수 있지만 

하나의 View에 여러개의 ViewModel이 존재했을 경우 복잡도가 높아집니다.

하나의 ViewModel에는 여러 LiveData를 가지는게 더 효율적으로 코드를 관리 할 수 있습니다.

![image1_5](/assets/img/mvvm_anti_pattern/image1_5.png)

특히 RecyclerView에서 여러 **LiveData object**를 적용하면 아주 쉽게 데이터를 관리할 수 있습니다. 

------

#### 6. 하나의 ViewModel에는 여러개의 View를 가져도 될까?

하나의 ViewModel에 여러개의 View를 가질 수 있지만 

여러개의 View의 로직들이 하나의 ViewModel에 존재한다면 

ViewModel안에 코드들이 길어지게 되거나 많은 **responsibilities**를 가지게 됩니다.

일부 로직들을 ViewModel과 동일한 범위의 **Presenter Layer**로 이동시키면 됩니다.

예로들어 Preference를 통해 데이터를 저장하고 싶을때

```kotlin
class TestViewModel(private val context: Context) : ViewModel() {
  fun saveTestData(value: String) {
    val pref = context.getSharedPreferences("pref", Context.MODE_PRIVATE)
    pref.putString("TEST", value).apply()
  }
}
```

ViewModel의 생성자로 Context를 받아서 바로 SharedPreference 이용하여 데이터를 저장하는게 아니라

![image1_6](/assets/img/mvvm_anti_pattern/image1_6.png)

다음과 같이 ViewModel의 생성자로 **Repository**를 받고

```kotlin
class TestViewModel(private val repository: Repository) : ViewModel() {
  fun saveTestData(value: String) {
    repository.saveData(value: String)
  }
}
```

Repository에서 SharedPreference 관련된 작업을 진행하면 됩니다.

```kotlin
class Repository(private val context: Context) {
  fun saveData(value: String) {
    val pref = context.getSharedPreferences("pref", Context.MODE_PRIVATE)
    pref.putString("TEST", value).apply()
  }
}
```

이렇게 코드를 작성했을 경우 testable해지고 architecture를 유지 가능하게 합니다.

------

#### 7. BindingAdapter, BindingMethod 중 어느것을 써야하나요?

**layout file**에서 지정된 이름의 **attribute**가 없을 경우 

**BindingAdapter**나 **BindingMethod**를 사용하여 **사용하고 싶은 attribute를 정의**할 수 있습닌다.

```xml
<com.kotlin.view.component.badge.BadgeStackView
    android:id="@+id/badgeStackView"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:items="@{vm._operationGroupNames}" // 사용하고 싶은 attribute 정의
		/>	
```

BindingAdapter는 주로 extension 형식으로 사용되고

```kotlin
fun BadgeStackView.setData(data:List<String>?) {
  setData(data)
}
```



BindingMethod는 CustomView에서 정의한 setter 메서드를 layout file에서 attribute로 사용할 수 있습니다.

```kotlin
// Binding Method 정의
@BindingMethods(value = [
  BindingMethod(
    type = BadgeStackView::class,
    method = "setData", 
    attribute = "app:items")])

// CustomView
class BadgeStackView(context: Context, attributeSet: AttributeSet? = null) : FrameLayout(context, attributeSet) {

    fun setData(data: List<String>?) {
        ...
    }
}
```

**CustomView를 사용하지 않는 경우**에는 **BindingAdapter**를 사용하고 

**CustomView를 사용할 경우**에는 최대한 **BindingMethod**를 사용하는것을 추천합니다.

BindingAdapter를 남발하게되면 setter 메서드를 찾는데 어려움을 겪게 됩니다.

추가적으로 CustomView에서 setter 메서드의 parameter가 2개 이상일 경우에는 BindingMethod를 사용할 수 없습니다.

이때는 BindingAdapter를 사용해야합니다.

------

### 에필로그

MVVM Architecture에 대한 글이 생각보다 많지 않아서 실무에 적용하는데 어려움을 많이 겪었습니다.

단순한 예제 글들은 많았지만 MVVM 동작원리, anti pattern 등 deep 한 글들이 거의 없어서 

MVVM Anti Pattern이라는 글을 작성하게 되었습니다.

이 글을 작성하는 데 도움을 주신 [태환님](https://github.com/taehwandev), [준호 형](https://github.com/pjhjohn), 그리고 [승민이 형](https://github.com/maryangmin)에게 감사의 인사를 드립니다. 



> [참조 블로그] : https://medium.com/androiddevelopers/viewmodels-and-livedata-patterns-antipatterns-21efaef74a54
>
> [참조 블로그] : https://medium.com/androiddevelopers/livedata-with-snackbar-navigation-and-other-events-the-singleliveevent-case-ac2622673150



 