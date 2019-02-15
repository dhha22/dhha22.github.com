---
layout: post
title: How to improve your MVP architectrue and tests#1
date: 2019-02-14
comments: true 
thumbnail: "assets/img/thumbnails/2018_droid_kaigi_2.png"
categories: Android
tags: [Android, MVP, Architecture]
---



## How to improve your MVP architectrue and tests

> 해당 글은 Droid kaigi 2018에서 발표했던 How to improve your MVP architectrue and tests 를 번역한 글입니다.

------

### 이번 주제에서 다뤄 볼 내용

- MVP 아키텍처를 적용했던 프로젝트의 안티패턴 소개
- **안티패턴의 개선 방법**과 **Presenter**에 대한 테스트 작성 및 역할 설명

------

### paymo 의 아키텍처

[^paymo]: 스마트 폰에서 더치페이 정산이 가능한 결제 앱

![image_2.1](/assets/img/2018_droid_kaigi/image_2.1.png)

------

### MVP 사용하고 있습니까?

- Android에서 2년 정도 전에 화제가 되었음
- Controller 대신에 **Presenter**라고 부르는 UI 로직을 담당하는 클래스를 이용하고 있음
- **Activity**와 **Fragment**는 View로 취급하고 **최대한 로직을 가지지 못하게 함**
- Presenter의 개념을 적용하고 있는 프로젝트가 상당히 많음

------

### MVP의 개요

![image_2.2](/assets/img/2018_droid_kaigi/image_2.2.png)

------

### Clean Architecture

![image_2.3](/assets/img/2018_droid_kaigi/image_2.3.png)

------

### 자주 듣는 MVP의 혜택

- View에서 Presenter로 **UI 로직을 분리** 시키는 것으로 테스트를 하기 쉬워짐
- 역할의 명확화에 의한 코드의 가독성 품질 향상
- 설계가 정해져 있기 때문에 리뷰 비용이 감소
- Activity의 비대화를 방지

---



### MVP 적용 프로젝트 안티패턴 

→ 실제로 일부 프로젝트에서 경험했던 부분

1. 테스트를 작성하지 않은 경우
2. Presenter가 Activity와 Fragment를 직접 접근하여 조작하고 있는 경우
3. View에 로직이 있고 Presenter에서 UI를 직접 조작하고 있거나 일관성이 없는 경우

이러한 안티패턴이 발생한 이유: **View 와 Presenter의 책임 분류가 되어 있지 않기 때문**

------

### 안티패턴 개선방안 의제

- **View 와 Presenter를 분리**
- Presenter에 대한 테스트 작성
- 테스트 코드를 리팩토링 하기

---

### View 와 Presenter를 분리하기

- 책임을 결정해도 Activity와 Fragement를 직접 Presenter에서 참조되어 버리면 결국 뭐든지 생겨버립니다.
- Presenter에 직접 Android에 의존한 View 조작이나 데이터 접근 처리가 있으면 테스트를 하는게 어려워 집니다.
- 우선 **View를 추상화** 하고 **강제로 책임을 분리**하면 됩니다.

![image_2.4](/assets/img/2018_droid_kaigi/image_2.4.png)



Android에 의존한 **프로세싱 및 데이터 접근 프로세싱을 Presenter에서 분리**하여 Presenter를 순수하게 유지합니다.



![image_2.5](/assets/img/2018_droid_kaigi/image_2.5.png)



테스트 할 때 분리된 부분을 Mock 화 하기 때문에 Presenter에서 테스트를 진행하기 쉬워집니다.

------

### Sample Code

- GitHub 클라이언트 앱
- GitHub ID를 입력하면 해당 유저가 표시됨
- 싱글 페이지의 간단한 앱
- 쉽게 설명하기 위해 Dagger등은 미사용

![image_2.6](/assets/img/2018_droid_kaigi/image_2.6.png)

---

### View(Activity) 안티패턴 

![image_2.7](/assets/img/2018_droid_kaigi/image_2.7.png)

---

### Presenter 안티패턴

![image_2.8](/assets/img/2018_droid_kaigi/image_2.8.png)



------

### View(Activity) 개선 후

![image_2.9](/assets/img/2018_droid_kaigi/image_2.9.png)



![image_2.10](/assets/img/2018_droid_kaigi/image_2.10.png)

![image_2.11](/assets/img/2018_droid_kaigi/image_2.11.png)

---

### Presenter 개선 후

![image_2.12](/assets/img/2018_droid_kaigi/image_2.12.png)



![image_2.13](/assets/img/2018_droid_kaigi/image_2.13.png)



![image_2.13](/assets/img/2018_droid_kaigi/image_2.13.png)

---

### 요약

- View를 추상화하여 Presenter는 미리 인터페이스에 정의 된 메서드를 통해서만 View를 접근 할 수 있는 강제력이 일어남
- View는 이벤트 통지 및 그리기만 하고 표출판정 등의 논리는 Presenter 측에서 실행



> [출처] : https://speakerdeck.com/kirimin/how-to-improve-your-mvp-architecture-and-tests

