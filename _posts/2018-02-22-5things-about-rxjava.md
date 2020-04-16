---
layout: post
title: RxJava를 잘 다룰수 있는 5가지 Tip
date: 2018-02-22
comments: true 
thumbnail: "assets/img/thumbnails/rxjava_tip.jpeg"
image: "assets/img/thumbnails/rxjava_tip.jpeg"
categories: Android
tags: [Android, RxJava]
---

RxJava를 처음 사용하거나 알고 있어도 항상 배워야 할 새로운 것이 있습니다.
해당 5가지 Tip을 알게 되면 RxJava를 좀 더 능숙하게 다룰 수 있게 됩니다.

> 해당 글은 RxJava 1.2.6 에서 사용할 수 있는 API를 참조합니다.

### 1. map 과 flatmap을 사용하는 시점

**map**과 **flatmap**은  ReactiveX 에서 자주 사용 되는 연산자 입니다.
하지만 해당 연산자들을 언제 써야할지 잘 모르는 경우가 있습니다.
**map**과 **flatmap**은 **Observable**에서 방출된 각각의 아이템들을 변환시키기 위한 연산자 입니다.
**map**은 항목 하나만 내보내는 반면 **flatmap**은 0개 이상의 항목을 내보냅니다.

![image_1.1](/assets/img/rx-java/image_1.1.png)



위 예제를 보면 **map**연산자는 **split**함수를 이용하여 각 문자열 마다 적용하여 문자열 배열을 포함하는 하나의 항목을 내보냅니다. 하나의 항목을 다른 항목으로 변환하고 싶을 경우 이 연산자를 사용하면 됩니다.

하지만 위 문자열들을 하나의 단일 스트림에 추가하고 싶을 경우에는 **flatmap**을 사용하면 됩니다.
**flatmap**연산자는 single sequence로 방출된 단어의 배열을 flattens 하게 만듭니다.

------

### 2. Observable.create(…)를 사용하여 observables를 생성하지 않도록 한다

개발하다 보면 기존의 동기식 또는 비동기식 API를 반응식 API로 변환해야할 경우가 있습니다.
**Observable.create**를 사용하여 개발할 수 있지만 다음과 같은 방식을 따라야 합니다.

- Observable 구독 취소 시 callback 등록 취소 (그렇게 하지 않을 경우 메모리 누수가 발생할 수 있음)
- 구독자가 구독하고 있을 경우에만 onNext 또는 onCompleted를 사용하여 이벤트를 보내야함
- onError를 사용하여 upstream으로 오류를 전파해야함
- backpressure 조절

위와 같은 요구사항을 따르며 직접 구현하는것은 어렵지만 다행히도 직접 구현하지 않아도 됩니다. RxJava에서 이를 위해 몇가지 **static helper method**를 제공합니다.

#### SyncOnSubscribe

`OnSubscribe<T>` 구현은 끌어오기(pull) 기반이며 **backpressure**를 명로하게 내장했습니다. 
**SyncOnSubscribe** 는 **backpressure**이 가능한 **Observable**을 제작할 수 있게 도와주는 편리한 유틸리티입니다.

>  backpressure가 이해가 안되신 분들은 **'3. backpressure를 다루는 방법'**을 읽고 오시길 바랍니다.

```java
public Observable<byte[]> readFile(@NonNull FileInputStream stream) {
  final SyncOnSubscribe<FileInputStream, byte[]> fileReader = SyncOnSubscribe.createStateful(
    () -> stream,
    (stream, output) -> {
      try {
        final byte[] buffer = new byte[BUFFER_SIZE];
        int count = stream.read(buffer);
        if (count < 0) {
          output.onCompleted();
        } else {
          output.onNext(buffer);
        }
      } catch (IOException error) {
        output.onError(error);
      }
      return stream;
    },
    s -> IOUtil.closeSilently(s));
  return Observable.create(fileReader);
}
```



#### fromCallable

단순한 동기 API를 wrapping 하여 반응형 API로 변환하는데 유용한 **static helper**입니다. 추가로 **fromCallable**는 예외도 처리합니다.

```java
public Observable<Boolean> enablePushNotifications(boolean enable) {
  return Observable.fromCallable(() -> sharedPrefs
    .edit()
    .putBoolean(KEY_PUSH_NOTIFICATIONS_PREFS, enable)
    .commit());
}
```



#### fromEmitter

**Observable**이 **unsubscribed** 되었을 때 비동기 API를 wrapping 하고 리소스를 관리하는데 유용한 **static helper**입니다. **fromCallable**과 달리 여러 항목을 내보낼 수 있습니다.

```java
import android.bluetooth.le.BluetoothLeScanner;
import android.bluetooth.le.ScanCallback;
import android.bluetooth.le.ScanResult;
import android.support.annotation.NonNull;
import rx.Emitter;
import rx.Observable;

import java.util.List;

public class RxBluetoothScanner {
    public static class ScanResultException extends RuntimeException {
        public ScanResultException(int errorCode) {
            super("Bluetooth scan failed. Error code: " + errorCode);
        }
    }
    
    private RxBluetoothScanner() {
    }

    @NonNull
    public static Observable<ScanResult> scan(@NonNull final BluetoothLeScanner scanner) {
        return Observable.fromEmitter(scanResultEmitter -> {
            final ScanCallback scanCallback = new ScanCallback() {
                @Override
                public void onScanResult(int callbackType, @NonNull ScanResult result) {
                    scanResultEmitter.onNext(result);
                }

                @Override
                public void onBatchScanResults(@NonNull List<ScanResult> results) {
                    for (ScanResult r : results) {
                        scanResultEmitter.onNext(r);
                    }
                }

                @Override
                public void onScanFailed(int errorCode) {
                    scanResultEmitter.onError(new ScanResultException(errorCode));
                }
            };
            
            scanResultEmitter.setCancellation(() -> scanner.stopScan(scanCallback));
            scanner.startScan(scanCallback);
        }, Emitter.BackpressureMode.BUFFER);
    }
}
```

------

### 3. Backpressure를 다루는 방법

**Observable**가 이벤트를 너무 빨리 생성했을 경우 **Observer downstream**은 이벤트를 처리할 수 없게 됩니다.
이럴때 우리들은 **MissingBackpressureException**을 만나게 됩니다.



![image_1.2](/assets/img/rx-java/image_1.2.png)

RxJava는 **backpressure**를 다룰 수 있는 몇가지 방법을 제공하지만 상황에 따라 선택하는게 좋습니다.

#### Cold vs Hot Observables

**Cold Observable**는 **subscribe** 할 경우 아이템들을 방출하기 시작합니다.
**Cold Observable**의 예로는 File 읽기, 데이터베이스 쿼리, 웹 요청, 정적 iterable를 Observable 변환 등 이 있습니다.

**Hot Observable**는 **subscribe**과 관계 없이 연속적으로 이벤트를 방출합니다. 
Observer가 **Hot Observable**를 **subscribe**하면 다음중 하나를 수행할 수 있습니다.

- 방출되었던 모든 이벤트의 일부분을 받습니다.
- 방출되었던 모든 이벤트를 받습니다.
- 새로운 이벤트가 발생할 때 받습니다.

**Hot Observable**의 예로는 터치 이벤트, 알림, 업데이트 진행 등 이 있습니다.

**Hot Observable**은 이벤트를 연속적으로 방출하기 때문에 사용자는 속도를 제어 할 수 없습니다.
예를들어 터치이벤트가 발생했을 경우 사용자는 터치이벤트의 속도를 제어할 수 없습니다. 따라서 아래 전략중 하나를 사용하는것이 가장 좋습니다.

#### BackpressureMode.NONE 과 BackPressureMode.ERROR

이 두개의 Mode에서 방출된 이벤트는 **backpressure**이 되지 않습니다.
observeOn의 내부 16 element 크기의 buffer가 overflow가 발생했을 경우 **MissingBackpressureException**가 나타납니다.

![image_1.3](/assets/img/rx-java/image_1.3.png)



#### BackpressureMode.BUFFER

이 모드에서는 초기 크기가 128인 무한 buffer가 만들어집니다. 너무 빨리 방출된 항목은 무제한으로 버퍼링됩니다. buffer가 소모되지 않으면 항목은 메모리가 모두 소모 될 때 까지 계속 누적됩니다. 결국 **OutOfMemoryException**을 만나게 됩니다.

![image_1.4](/assets/img/rx-java/image_1.4.png)



#### BackpressureMode.DROP

이 모드에서는 크기가 1인 고정 buffer를 사용합니다. **downstream observable**는 유지되지 않으면 첫번째 항목이 버퍼링되고 그 다음 방출된 이벤트들이 삭제됩니다. **consumer**가 다음 값을 받을 수 있는 상태가 되면 **Observable **이 방출한 첫번째 값을 받습니다.

![image_1.5](/assets/img/rx-java/image_1.5.png)





#### BackpressureMode.LATEST

이 모드는 **BackpressureMode.DROP**과 비슷합니다. 이 모드도 역시 크기가 1인 고정 버퍼를 사용합니다. 그러나 첫번째 항목을 버퍼링 하고 그 다음 방출된 이벤트를 삭제하는게 아니라 **BackpressureMode.LATEST**는 버퍼의 항목을 최신 방출로 바꿉니다. **consumer**가 다음 값을 받을 수 있는 상태가 되면 **Observable**이 방출한 최신 값을 받습니다.

![image_1.6](/assets/img/rx-java/image_1.6.png)

------

### 4. 실수로 스트림을 종료하는 실수를 방지하는 방법

RxJava는 Observable sequence에 오류가 발생했을 경우 sequence를 종료하고 sequence에 알림을 보내 해당 오류를 전달합니다.

하지만 sequence의 종료를 원하지 않을 수도 있습니다. 이러한 경우 RxJava는 sequence를 종료하지 않고도 오류를 처리 할 수 있는 다양한 방법을 제공합니다.

#### onErrorResumeNext

**onErrorResumeNext** 를 사용하게 되면 해당 오류를 가로챌 수 있고 다른 **Observable**값으로 반환할 수 있습니다. 

오류를 추가 정보로 wrap 하고 새 오류를 반환하거나 수신할 새로운 이벤트로 반환할 수 있습니다.

```java
public Observable<SearchResult> search(@NotNull EditText searchView) {
  return RxTextView.textChanges(searchView) 
    .map(CharSequence::toString)
    .debounce(500, TimeUnit.MILLISECONDS)   
    .filter(s -> s.length() > 1)            
    .observeOn(workerScheduler)             
    .switchMap(query -> searchService.query(query))   
    .onErrorResumeNext(Observable.empty()); // 에러가 발생할 경우 텍스트 뷰 변경을 중단
}
```



**onErrorResumeNext** 연산자를 사용하면 **down stream sequence**를 복구하지만 **onError** 알림이 생성되었으므로 **upstream sequence**가 종료됩니다. 만약 **Subject**를 사용할 때 **onError** 알림이 발생했을 경우 **Subject**는 종료됩니다.

**upstream**을 계속 실행하고 싶다면 **Observable**를 **onErrorResumeNext** 연산자의 **flatMap** 또는 **switchMap** 연산자 내부에 선언하면 됩니다. 

```java
public Observable<SearchResult> search(@NotNull EditText searchView) {
  return RxTextView.textChanges(searchView) 
    .map(CharSequence::toString)
    .debounce(500, TimeUnit.MILLISECONDS)   
    .filter(s -> s.length() > 1)            
    .observeOn(workerScheduler)             
    .switchMap(query -> searchService.query(query)   
    			.onErrorResumeNext(Observable.empty())); 
}
```

------

### 5. Observable을 공유하는 방법

때때로 **Observable**의 결과물을 여러 **Observer**과 공유해야할 때가 있습니다. **Observable**에서 RxJava로 방출 된 이벤트를 **multicast** 하는 두가지 방법은 **share** 과 **publish** 이 있습니다.

#### Share

**share** 연산자를 사용하면 여러 **Observer**가 **Observable**에 연결하여 방출된 이벤트를 공유할 수 있습니다 .

아래 예제에서 **Observable**는 MotionEvent 아이템들을 방출합니다. 그런 다음 **DOWN** 및 **UP** 터치 이벤트를 걸러낼 두개의 **Observable**들을 만듭니다. 

**Down**이벤트들은 **빨강색 원**을 그리고 **UP**이벤트들은 **파란색 원**을 그립니다.

```java
public void touchEventHandler(@NotNull View view) {
  final Observable<MotionEvent> motionEventObservable = RxView.touches(view).share();
  // Capture down events
  final Observable<MotionEvent> downEventsObservable = motionEventObservable
    .filter(event -> event.getAction() == MotionEvent.ACTION_DOWN);
  // Capture up events
  final Observable<MotionEvent> upEventsObservable = motionEventObservable
    .filter(event -> event.getAction() == MotionEvent.ACTION_UP);

  // Show a red circle at the position where the down event ocurred
  subscriptions.add(downEventsObservable.subscribe(event ->
      view.showCircle(event.getX(), event.getY(), Color.RED)));
  // Show a blue circle at the position where the up event ocurred
  subscriptions.add(upEventsObservable.subscribe(event ->
      view.showCircle(event.getX(), event.getY(), Color.BLUE)));
}
```



**Observer**가 **Observable**를 **subscribe** 했을 경우 이벤트가 방출되기 시작됩니다. 이벤트가 방출되고나서 **subscribe** 했을 경우 앞에 방출되었던 이벤트를 놓치게 되기 때문에 문제가 됩니다.



![image_1.7](/assets/img/rx-java/image_1.7.gif)

위 예제에서 파란색 **Observer**는 첫번째 값이 방출되고 난 이후 **subscribe** 했기 때문에 해당 이벤트를 놓치게 됩니다. 이벤트를 놓치고 싶지 않다면 **publish** 연산자를 사용하면 됩니다.



#### Publish

**publish** 연산자를 호출 했을 경우 **Observable**는 **ConnectedObservable**로 변화합니다. 

아래 예제는 **publish**연산자를 사용한 부분을 제외하고 위 예제와 동일합니다.

```java
public void touchEventHandler(@NotNull View view) {
  final ConnectedObservable<MotionEvent> motionEventObservable = RxView.touches(view).publish();
  // Capture down events
  final Observable<MotionEvent> downEventsObservable = motionEventObservable
    .filter(event -> event.getAction() == MotionEvent.ACTION_DOWN);
  // Capture up events
  final Observable<MotionEvent> upEventsObservable = motionEventObservable
    .filter(event -> event.getAction() == MotionEvent.ACTION_UP);

  // Show a red circle at the position where the down event ocurred
  subscriptions.add(downEventsObservable.subscribe(event ->
      view.showCircle(event.getX(), event.getY(), Color.RED)));
  // Show a blue circle at the position where the up event ocurred
  subscriptions.add(upEventsObservable.subscribe(event ->
      view.showCircle(event.getX(), event.getY(), Color.BLUE)));
  // Connect the source observable to begin emitting events
  subscriptions.add(motionEventObservable.connect());
}
```

 **Observable**를 **subscribe** 해도 바로 이벤트가 방출되지 않고 **ConnectedObservable**의 **connect**를 호출하면 이벤트가 방출되기 시작합니다.





![image_1.8](/assets/img/rx-java/image_1.8.gif)

**connect**가 호출되고 난 뒤에는 같은 이벤트의 동일한 순서가 녹색 및 파란색 **Observer**들에게 모두 방출됩니다.



> [출처] : https://medium.com/@jagsaund/5-not-so-obvious-things-about-rxjava-c388bd19efbc