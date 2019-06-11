---
layout: post
title: Privacy Changes in Android Q #1
date: 2019-06-17
comments: true 
thumbnail: "assets/img/thumbnails/io_2019_android.jpeg"
categories: Android
tags: [Android]
---



## Privacy Changes in Android Q#1



> 해당 글은 제가 Google I/O 19 Extended in Korea Android 에서 발표했던 부분을 정리한 글입니다.

------



안드로이드에서 새로운 버전이 나올때 마다 개인정보를 보호하기 위하여 다양한 기능을 추가하고 있습니다.

![Google I:O extended Android.003](/assets/img/2019_google_i:0/Google I:O extended Android.003.jpeg)



Lockdown Mode, Secure DNS 그리고 마시멜로우 부터 추가되었던 Runtime permissions 등이 있었습니다.



---

![Google I:O extended Android.004](/assets/img/2019_google_i:0/Google I:O extended Android.004.jpeg)



Android Q 에서도 개인정보를 보호하기 위하여 많은 기능이 추가되었습니다.



---

![Google I:O extended Android.005](/assets/img/2019_google_i:0/Google I:O extended Android.005.jpeg)



그중 가장 변화가 큰 부분은 **Location Settings**, **App Storage**, **Background App Launching**, **Device ID Restrictions** 이 있습니다.



---

![Google I:O extended Android.007](/assets/img/2019_google_i:0/Google I:O extended Android.007.jpeg)



오늘 다뤄볼 내용은 **백그라운드 앱에 대한 새로운 제한사항** 입니다.



---

![Google I:O extended Android.008](/assets/img/2019_google_i:0/Google I:O extended Android.008.jpeg)



**Android Q Beta4** 에서 몇가지 변경사항이 있습니다.

- 사용자 상호작용 없이 백그라운드에서 액티비티를 실행시킬 수 없음

- 백그라운드에서 액티비티를 실행시킬 때는 notification을 활용해야함
- 개발자 옵션에서 **Allow background activity starts** 기능을 꺼야지 백그라운드 제한사항을 실행시킬 수 있음
  (정식 release 버전에서는 on/off 상관없이 백그라운드 제한사항이 적용됨)



---

![Google I:O extended Android.009](/assets/img/2019_google_i:0/Google I:O extended Android.009.jpeg)



Android Q Beta에서 백그라운드 제한사항을 실행시키고 싶으면 

개발자 옵션에서 앱 카테고리에 있는 백그라운드 활동 시작 허용 기능을 off 하면 적용됩니다. 



---

![Google I:O extended Android.010](/assets/img/2019_google_i:0/Google I:O extended Android.010.jpeg)



Android Q 이전에는 사용자 상호작용 없이 백그라운드에서 액티비티를 실행시킬수있었는데

이러한 동작은 사용자에게 불편함을 일으키게 됩니다.



---

![Google I:O extended Android.011](/assets/img/2019_google_i:0/Google I:O extended Android.011.jpeg)



앱이 현재 백그라운드 상태에 있습니다.



---

![Google I:O extended Android.012](/assets/img/2019_google_i:0/Google I:O extended Android.012.jpeg)



서버에서 push messsage를 받았을 경우 사용자 상호작용 없이 

**FLAG_ACTIVITY_NEW_TASK** 플래그를 이용하여 액티비티가 실행되는 모습을 볼 수 있습니다.

---

![Google I:O extended Android.014](/assets/img/2019_google_i:0/Google I:O extended Android.014.jpeg)



소스코드를 보게 되면 push message가 왔을 경우 onMessageReceived 함수가 호출되고

onMessageReceived 함수 안에서는 액티비티를 실행시키기 위하여 **FLAG_ACTIVITY_NEW_TASK** 를 사용하고 있습니다.



---

![Google I:O extended Android.015](/assets/img/2019_google_i:0/Google I:O extended Android.015.jpeg)



Android Q에서는 백그라운드에서 액티비티 실행이 제한됩니다.

사용자 상호작용없이 실행이 불가능하며 앱 타겟이 Android Q가 아니더라도 Android Q 디바이스에서는 영향을 미칩니다.

지금 현재 베타에서는 백그라운드에서 액티비티가 실행되지만 토스트 메시지와 로그켓에 나타납니다. 

추후 정식버전에서는 백그라운드에서 액티비티 실행이 제한됩니다.



---

![Google I:O extended Android.016](/assets/img/2019_google_i:0/Google I:O extended Android.016.jpeg)



그러면 앱이 백그라운드 상태일때 사용자에게 중요한 정보를 전달하고 싶을때는 어떻게 해야할까요?



---

![Google I:O extended Android.018](/assets/img/2019_google_i:0/Google I:O extended Android.018.jpeg)



백그라운드 상태일때 액티비티를 실행시킬수 있는 몇가지 사항이 있습니다.

- 시스템에서 PendingIntent을 통하여 실행시키는 경우
- 시스템 브로드 캐스트를 받아서 실행시키는 경우 (ACTION_NEW_OUTGOING_CALL, SECRET_CODE_ACTION 등)
- 사용자로부터 SYSTEM_ALERT_WINDOW 권한을 받아서 실행시키는 경우

특별한 상황이 아닌 이상은 PendingIntent을 통하여 액티비티를 실행시키는것을 권장합니다.



---

![Google I:O extended Android.019](/assets/img/2019_google_i:0/Google I:O extended Android.019.jpeg)



앱이 백그라운드 상태에 있을때 사용자에게 중요한 정보를 전달하고 싶으면 바로 액티비티를 실행시키는것이 아니라 notification을 통해 전달해야합니다.



---

![Google I:O extended Android.021](/assets/img/2019_google_i:0/Google I:O extended Android.021.jpeg)



Android Q에서 서버에서 push message를 받고 바로 액티비티를 실행시키면 다음과 같이 경고문구가 나옵니다.

이러한 액티비티 실행 방식은 Android Q가 정식 출시되면 지원되지 않습니다.



---

![Google I:O extended Android.023](/assets/img/2019_google_i:0/Google I:O extended Android.023.jpeg)





다음과 같이 Notification을 이용하여 **full-screen intent**를 이용하면 사용자에게 액티비티를 실행시킬수 있는 권한을 줄 수 있습니다.



---

![Google I:O extended Android.027](/assets/img/2019_google_i:0/Google I:O extended Android.027.jpeg)



소스코드를 보면 **setFullScreenIntent** 메서드를 사용한것을 확인할 수 있는데요. 파라미터로 PendingIntent가 들어가는것을 확인할 수 있습니다. setFullScreenIntent에 대하여 좀 더 자세히 알아보도록 하겠습니다.



---

![Google I:O extended Android.029](/assets/img/2019_google_i:0/Google I:O extended Android.029.jpeg)



setFullScreenIntent를 사용하면 Heads-up notification이 나타나고 자동으로 사라지지 않습니다.

만약 NotificagionChannel의 중요도를 설정하지 않으면 Heads-up notification은 나타나지 않습니다.



---

![Google I:O extended Android.031](/assets/img/2019_google_i:0/Google I:O extended Android.031.jpeg)



여기서 중요한 부분은 NotificaionChannel에서 중요도를 설정할때 **IMPORTANCE_HIGH**로 설정하지 않았을 경우에도 Heads-up notification이 나타나지 않습니다.



---

![Google I:O extended Android.033](/assets/img/2019_google_i:0/Google I:O extended Android.033.jpeg)



마지막으로 앱 타겟이 Android Q 이상 일 경우 **USE_FULL_SCREEN_INTENT** 권한을 설정해야합니다.

PendingIntent에 대하여 좀 더 자세히 살펴보도록 하겠습니다.



---

![Google I:O extended Android.034](/assets/img/2019_google_i:0/Google I:O extended Android.034.jpeg)



백그라운드에서 push message가 왔을 경우 notification을 띄우는데 notification manager가 시스템 서비스에서 관리합니다. 

notification에서 앱에 있는 액티비티를 실행 시키려면 intent을 통하여 실행시켜야 하지만 앱과 전혀 다른 프로세스이므로 실행시킬수 있는 권한이 없습니다.

이때 PendingIntent를 이용하면 intent를 wrapping하고 notification manager에게 액티비티를 실행시키는 권한을 전달할 수 있습니다.



---

![Google I:O extended Android.035](/assets/img/2019_google_i:0/Google I:O extended Android.035.jpeg)

그러면 RemoteViews에서 버튼들의 클릭액션은 어떻게 구현해야할까요?



---

![Google I:O extended Android.036](/assets/img/2019_google_i:0/Google I:O extended Android.036.jpeg)



RemoteViews는 다른 프로세스에서 동작하고 View를 상속받은 실제 View가 아닙니다. 

이렇기 때문에 몇가지 제약사항이 있습니다. 사용할 수 있는 View가 제한되어있기 때문에 커스텀 뷰를 이용할 수 없습니다.



---

![Google I:O extended Android.039](/assets/img/2019_google_i:0/Google I:O extended Android.039.jpeg)



버튼의 클릭을 구현할 때는 **setOnClickListener** 방식이 아닌 **setOnClickPendingIntent**를 이용하여 구현해야 합니다.

해당 메서드의 파라미터는 버튼의 id 값과 PendingIntent가 있습니다.

버튼 클릭 로직을 쉽게 관리하기 위하여 바로 액티비티를 실행시키지 않고 브로드캐스트를 이용하였습니다.



---

![Google I:O extended Android.042](/assets/img/2019_google_i:0/Google I:O extended Android.042.jpeg)



브로드캐스트 리시버 부분을 보면 버튼을 클릭했을 경우 notification이 사라져야 하기 때문에 notification manager의 cancel을 호출하였습니다.

버튼을 클릭했을 경우 intent 정보에 Boolean 값을 추가하였는데요. 수락버튼을 눌렀을 경우 acceptCall 값이 true이기 때문에 라이브 상담 액티비티를 PendingIntent를 이용하여 실행시켰습니다.



---

![Google I:O extended Android.043](/assets/img/2019_google_i:0/Google I:O extended Android.043.jpeg)



요약을 하면 백그라운드 상태일때 액티비티는 실행시킬 수 없고 사용자에게 notification을 통하여 액티비티를 실행시킬 수 있는 권한을 줘야합니다.

다음 글에서는 **안드로이드 기기 고유 식별자 제한** 에 대하여 다뤄보도록 하겠습니다.