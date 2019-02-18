---
layout: post
title: How to improve your MVP architectrue and tests#2
date: 2019-02-18
comments: true 
thumbnail: "assets/img/thumbnails/2018_droid_kaigi_2.png"
categories: Android
tags: [Android, MVP, Architecture]
---



## How to improve your MVP architectrue and tests #2

> 해당 글은 Droid kaigi 2018에서 발표했던 How to improve your MVP architectrue and tests 를 번역한 글입니다.

------

### 이번 주제에서 다뤄 볼 내용

- MVP 아키텍처를 적용했던 프로젝트의 안티패턴 소개
- 안티패턴의 개선 방법과 **Presenter**에 대한 테스트 작성 및 역할 설명

> [How to improve your MVP architectrue and tests #1] : https://dhha22.github.io/android/2018/03/05/kotlin-aniti-pattern-1.html

---

### MVP 안티 패턴 개선 의제 

- View 와 Presenter를 분리
- **Presenter에 대한 테스트를 작성**
- 테스트 코드를 리팩토링 하기

---

### 테스트의 분류

![image_3.1](/assets/img/2018_droid_kaigi/image_3.1.png)



- 테스트를 크게 나눠 보면 UI 테스트, 통합 테스트, 유닛 테스트의 3가지가 있음
- 세 피라미드의 관계에서 위로 갈수록 구현 및 실행 비용이 높은 프로덕션 환경에 가까움

---

### Presenter에 대한 테스트

- Presenter에 대한 테스트는 이 중에서 통합 테스트에 해당
- 유닛테스트 보다 넓은 범위를 테스트 할 수있고, UI 테스트보다 비용이 낮음

---

### Presenter 테스트를 작성하는 장점

- View의 표출판정 등의 화면 전체 논리의 정확성을 UI 테스트에 가까울 정도로 확인할 수 있음
- 넓은 범위에서 버그를 감지하기 쉬워짐
- Presenter에 로직을 모아 테스트를 작성하는 것으로, Presenter의 처리의 리팩토링과 모델 클래스에 분리되어 하기 쉬워짐
- 로컬 유닛 테스트로 에뮬레이터 없이 재빠르게 실행 가능하기 때문에 부담없이 실행 확인 할 수 있음

---

### Presenter 에 테스트 작성하는 방법

- Mock라이브러리에서 View와 UseCase를 Mock화
- Presenter의 이벤트 메서드를 호출하여, 처리결과 View 와 UserCase의 메서드가 정상적으로 호출되어 있는지를 확인하는것으로 테스트를 시행

---

### 복습

![image_2.4](/assets/img/2018_droid_kaigi/image_2.4.png)



- Android에 의존한 처리 및 데이터 접근 처리를 Presenter에서 분리하여 Presenter를 순수하게 유지



![image_2.5](/assets/img/2018_droid_kaigi/image_2.5.png)



- 테스트 할 때 는 분리된 부분을 Mock화 하여 Presenter에 테스트를 하는게 쉬워짐 



![image_3.2](/assets/img/2018_droid_kaigi/image_3.2.png)

----

### Mockito 를 사용

- Java의 주요 모의 라이브러리
- 클래스나 인터페이스의 모의 인스턴스를 생성할시, 메서드에 임의의 반환값을 지정하거나 메서드가 호출된 횟수를 확인할 수 있음

![image_3.3](/assets/img/2018_droid_kaigi/image_3.3.png)

---

### 샘플코드

![image_3.4](/assets/img/2018_droid_kaigi/image_3.4.png)

![image_3.5](/assets/img/2018_droid_kaigi/image_3.5.png)

---

### 요약

- Presenter 에서 Android 프레임 워크에 의존한 처리 및 데이터 접근 처리를 분리하여 Presenter에 대해 로컬 유닛 테스트를 작성할 수 있음
- Presenter 에 대한 테스트는 전체 처리의 정확성을 검증하는 통합 테스트
- Mockito 의 verify 메서드 등을 활용하여, Presenter 의 처리결과, View 와 UseCase의 메서드가 예상대로 불러지는것인가를 검증함

---

### 실제로 프로덕트에서 Presenter 테스트를 쓰기 시작해서 느낀 단점

- 의미있는 단위 테스트를 작성하려면 나름대로 비용이 생김, 점차 귀찮음이 많아지고 엉성하게 됨
- 새로 구현할때는 정성스럽게 테스트를 작성하지만 유지보수할때에는 어떻게 해도 엉성하게 된 테스트를 고치는것만으로도 만족하게됨
- 복잡한 화면과 테스트 코드가 점점 커져버림, 깊이 생각하지 않고 쓰면 중복 별로 의미가 없는 테스트코드가 되버림

----

### MVP 안티 패턴 개선 의제 

- View 와 Presenter를 분리
- Presenter에 대한 테스트를 작성
- **테스트 코드를 리팩토링 하기**

---

### 테스트 코드를 리팩토링 하기

- 중복 된 테스트 코드가 있으면 사양 변경시 변경사항이 늘어남
- 코드량도 증가하므로 쓰는것도 읽은것도 힘들어 내용이 엉성하게 됨
- 코스트가 높은 테스트는 쓰지 않아도 됨

---

### 테스트 코드를 다시 보면

![image_3.6](/assets/img/2018_droid_kaigi/image_3.6.png)

---

### 어떻게 하면 좋을까

- 가능한 효율적이고 변화에도 잘 적용할수 있는 테스트 코드를 사용하고 싶음
- 공통 부분은 동인한 테스트를 통해, 케이스마다 차이만을 확인하고 싶음

---

### 테스트 코드를 구조화 해봄

- Kotlin 용 테스트 프레임 워크 라이브러리 Spek 등을 참고로 하여 구조화 된 테스트를 생각함
- Spek 등의 라이브라리를 이용해도 좋지만, Spek 는 대형 업데이트를 앞두고 있기때문에 이번에는 Kotlin의 표준기능의 로컬 함수를 사용하여 구현해봄



![image_3.7](/assets/img/2018_droid_kaigi/image_3.7.png)

![image_3.8](/assets/img/2018_droid_kaigi/image_3.8.png)



![image_3.9](/assets/img/2018_droid_kaigi/image_3.9.png)



![image_3.10](/assets/img/2018_droid_kaigi/image_3.10.png)



---

### 요약

- Kotlin 의 로컬 함수를 이용한 테스트 코드를 구조화 해본 시도를 하고 있음
- 읽기 어려워지기 쉬운 Mockito 를 이용한 테스트를 정리하고 가독성을 높이고 있음
- 검증하고싶은 패턴이나 케이스가 증가해도 코드가 비대해지거나 수정 비용이 증가하지 않도록 하고 있음

---

### Presenter 의 처리를 도메인 모델에 넘겨주기

#### 테스트의 피라미드를 기억

- 통합 테스트는 강력하고 편리하지만, 모델에 대한 간단한 assert 테스트 보다 쓰기 편하고 정밀도가 높음
- 본래는 유닛 테스트의 비중이 큰것이 건전
- Presenter 에서 모델 처리 및 테스트를 옮겨서 실행

![image_3.1](/assets/img/2018_droid_kaigi/image_3.1.png)



![image_3.11](/assets/img/2018_droid_kaigi/image_3.11.png)



![image_3.12](/assets/img/2018_droid_kaigi/image_3.12.png)

![image_3.13](/assets/img/2018_droid_kaigi/image_3.13.png)



![image_3.14](/assets/img/2018_droid_kaigi/image_3.14.png)



---

### 요약

- Mockito 를 사용한 통합 테스트 보다 assert 에 의한 유닛 테스트의 방법이 비용이 낮고 정확도가 높음
- Presenter 에서 도메인 모델로 분리 시키는 처리를 찾아 내자
- 조금씩 유닛 테스트를 늘려 나감으로써, Presenter 에 대한  테스트가 비대화 되지 않도록 함
- 정말로 핵심적인 부분의 테스트는 Espresso 등을 이용한 UI 테스트를 사용하는 방법도 있음



#### UI 테스트 보다 유닛 테스트를 우선시 하고, 유닛 테스트에서 부족한 부분을 통합테스트에서 커버 함



> [출처] : https://speakerdeck.com/kirimin/how-to-improve-your-mvp-architecture-and-tests

