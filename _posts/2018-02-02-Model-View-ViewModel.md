---
layout: post
title: MVVM
date: 2018-02-02
thumbnail: "assets/img/thumbnails/mvvm.png"  
categories: Android
tags: [Android, Architecture]
---

## Model - View - ViewModel

MVVM(Model-View-ViewModel)은 Microsoft의 Ken Cooper와 Ted Peters가 개발한 아키텍처입니다.
데이터 바인딩 메커니즘을 사용한 UI부분의 추상화가 특징입니다.

MVVM의 개념은 세 가지 역할로 구성되어 있습니다, 사용자 인터페이스 (UI)의 분리를 목적으로 한 아키텍처 입니다. (그림 1.6)

응용 프로그램의 UI와 비즈니스 로직을 연관 데이터 바인딩을 이용하여 관계를 분리합니다. 
View의 추상화 행위, 비즈니스 로직, 도메인 모델 중에서 View 관련된 글루 코드를 없애는 메리트가 있습니다.
![그림_1.6](/assets/img/architecture-pattern/image_1.6.png)

> 글루코드 : 컴퓨터 프로그래밍 에서 프로그램의 요구 사항의 실현에는 일절 기여하지 않지만, 원래 호환되지 않는 부분끼리를 결합하는 데에만 작동 코드 이다. 예를 들어 함수 인터페이스가 있다.



#### Model

Model은 어플리케이션 도메인 모델을 말합니다. 데이터 구조 외에 비즈니스 로직을 표현하는 수단도 모델에 포함되어 있습니다.

#### View

View는 사용자가 보는 구조, 레이아웃, 모양을 정의하는 역할을 담당합니다. 비즈니스에 관련된 로직에서 분리되어 있습니다.

#### View Model

ViewModel은 Model과 View의 중재자입니다. 표시에 관련된 로직을 담당합니다.
Model을 조작하고 View가 사용하기 쉬운 형태로 데이터를 제공하는 역할을 합니다.



ViewModel은 View를 Model로부터 격리하고 Model과 View를 독립적으로 진화시키기 위한 것입니다. 
View는 ViewModel을 알고 있으며, ViewModel은 Model을 파악하고 있습니다. 
반대로 Model은 ViewModel을 인식하지 못하고 ViewModel도 View에 대해 알지 못합니다. 
각각독립성을 높이고 (캡슐화), 관심사를 분리하고 효율적인 개발을 목표로하고 있습니다.



### 데이터바인딩

MVVM 등장 배경에는 데이터 바인딩 (Data Binding) 구조의 발전이 있습니다.
데이터 바인딩은 데이터 표현과 사용자 인터페이스를 정적 또는 동적으로 바이딩 할 수있는 구조입니다.
데이터 표현은 XML, XAML 등의 마크 업 언어가 사용됩니다. 기능에 따라 2 종류로 분류 할 수 있습니다. (그림 1.7)

![그림_1.7](/assets/img/architecture-pattern/image_1.7.png)



#### 단방향 데이터 바인딩

표시 할 데이터가 변경되면 그에 따라 사용자 인터페이스에 변화를 반영화하는 단방향 바인딩



#### 양방향 데이터 바인딩

단방향 이외에, 사용자 인터페이스의 변경 또는 조작에 따라 데이터도 변경해야하는 양방향 데이터 바인딩



MVVM의 도입은 데이터 바인딩 또는 유사한 구조를 가진 플랫폼이 전제입니다. 
MVVM이 ViewModel에 의해 Model과 View를 분리하는 특성상 
ViewModel에서는 View에 표시되는 데이터를 조립하고 반영해야합니다.
복잡한 UI 작업을 서포트하는 구조가 데이터 바인딩입니다.
플랫폼의 적용 예로는 Android에서 XML 및 Java 코드의 바인딩, WPF와 Silverlight 플랫폼을 기반으로 한 XAML과 C # 소스 코드의 바이딩을 들 수 있습니다. JavaScript의 세계에서 DOM (Document Object Model)과 JavaScript 소스 코드를 바인딩하는 형태로 MVVM의 적용 예가 있습니다.
MVVM 아키텍처에서 데이터 바인딩의 역할은 큰 것입니다.



### MVVM의 기반 개념 : Presentation Model

MVVM은 Martin Fowler의 Presentation Model을 기반으로하고 있습니다.

MVVM의 기반으로 하고 있는 Presentation Model은 View 표시에 관한 정보를 관리하는
Presentation Model의 상태를 그대로 반영합니다. 
Presentation Model에서 데이터에 변화가 있으면 View로 설정하고 View 측에서 사용자 이벤트가 발생하면 신속하게 PresentationModel에 통지합니다.
이 설정 및 알림 작업은 View의 상태를 도메인 모델에서 분리하는 데 필수적입니다.
이 View (가 제공하는 UI)가 복잡할수록 설정 및 알림 시간이 개발자의 부담입니다.
(바인딩을위한 코드를 손으로 쓰는 것은 생각보다 귀찮은 것입니다).
소프트웨어 개발의 진화함에 따라 Presentation Model과 View 동기화하는 PresentationModel 성가신 부분을 데이터 바인딩 메커니즘으로 실현할 수있게 된 것은 기술적인 성과였습니다.

MVVM 아키텍처는 데이터 바인딩 메커니즘을 사용하여 Presentation Model을 개발시켜 (국소 적으로 자동 생성을 사용하여 최적화), View와 Model의 분리를 추진하고 있다고 합니다.

