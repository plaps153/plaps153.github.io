---
layout: post
title:  "[RxSwift] Driver"
description: "RxSwift Driver에 대한 간단한 설명"
date:   2020-01-06
category: RxSwift
tags: [RxSwift, Driver, Development]
comments: true
---

사실 Driver의 태생 목적을 정확히 모르고 사용해 왔습니다. 그저 Main thread에서만 동작하는 observable, 그래서 UI 작업에 많이 쓰이는 Observable로만 알고 있습니다.
그리고 subscribe 와 짝을 이루는 drive 등, 좀 정확히 알 필요가 있어서 정리합니다. 물론 이곳 저곳 많이 참고하였습니다.

Driver는 RxSwift 뿐만 아니라 RxCocoa까지 import해야 사용할 수 있습니다. 이유는 간단한데 RxCocoa [Traits](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Traits.md)의 한 종류이기 때문 입니다. 

# Driver 정의
Rx사이트에서 Driver에 대해 [정의한 내용](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Traits.md#why-is-it-named-driver)은 다음과 같습니다.
> This is the most elaborate trait. Its intention is to provide an intuitive way to write reactive code in the UI layer, or for any case where you want to model a stream of data Driving your application.

>일단 UI layer에서 직관적으로 reactive code를 짜기 위해 제공되며, **당신의 application을 driving 하기(구동시키기)위해 stream of data를 modeling하기 원할 때 사용할 수 있다.**

우리 windows 사용할 때를 생각해 보면 이해하기 쉬울 것 같습니다. 그래픽 카드를 새로 샀다고 했을 때 제대로 사용하기 위해서는 반드시 해당 그래픽 카드에 맞는 driver를 설치해야 하지요? 마찬가지로 어떤 UI를 구동시키기 위해 필요한 일련의 데이터가 바로 그 UI를 위한 driver겠네요. 그래서 이따 설명드리겠지만 아래의 소스를 위의 설명으로 설명하자면

```swift
let idEvent = idTextField.rx.text.orEmpty.asDriver()
```

**idTextField**의 text값을 driver로 만든다는 것은 이 **idEvent** driver 가 구동시킬 UI가 있다는 것으로 이해하면 되겠죠? 결국에 아래와 같이 button을 enable하는데 driver로 쓰이게 됩니다.

```swift
buttonEnableEvent
    .drive(button.rx.isEnabled)
    .disposed(by: disposeBag)
```

# Driver 특징
또한 Driver.swift에 다음과 같은 특징이 있습니다.

> Trait that represents observable sequence with following properties:
> -- it never fails
> -- it delivers events on `MainScheduler.instance`
> -- `share(replay: 1, scope: .whileConnected)` sharing strategy
> 
> Additional explanation:
> -- all observers share sequence computation resources
> -- it's stateful, upon subscription (calling subscribe) last element is immediately replayed if it was produced
> -- computation of elements is reference counted with respect to the number of observers
> -- if there are no subscribers, it will release sequence computation resources

요약하자면 이렇습니다.
1. Error가 없다. (error를 emit하지 않는다는 의미겠지요.)
2. Event은 무조건 MainScheduler.instance로 전달된다.
3. share(reply: 1, scope: .whileConnected) shairing strategy를 가진다.
    - share에 대해서는 [다음 사이트](https://plaps153.github.io/rxswift/2020/01/06/RxSwift-Operator-Share.html) 를 참고
	- .whileConnected는 1개 이상 subscriber가 존재하는 동안에만, 버퍼가 유지됩니다. 따라서 subscription이 0개가 된 뒤에 발생하는 subscription은 새로운(latest) 결과를 갖게 됩니다.
	- subscriber가 없을 경우에는 결과를 
4. 모든 observer들은 일련의 computation resources를 공유한다.
5. Stateful (이전 상태를 기록)하다. 즉 subscribe 동시에 직전의 결과가 있다면 반환한다.
6. 계산된 결과들은 (elements) observer의 갯수 만큼 referenced count됩니다.

# When to use?
그렇다면 과연 driver를 언제 사용해야 하는지 아래의 [예제](https://mrgamza.tistory.com/497)를 보면서 설명하겠습니다.

```swift
let idEvent = idTextField.rx.text.orEmpty.asDriver()
let passEvent = passTextField.rx.text.orEmpty.asDriver()
        
let buttonEnableEvent = Driver.combineLatest(idEvent, passEvent)
    .map { !$0.0.isEmpty && !$0.1.isEmpty }
        
buttonEnableEvent
    .drive(button.rx.isEnabled)
    .disposed(by: disposeBag)
```
**idEvent** 및 **passEvent** 는 각각 **idTextField** 및 **passTextField**의 **observable** 입니다. (orEmpty로 nil값이 전달되지 않도록 하였습니다.) 텍스트가 입력되면 가장 최근에 입력된 값이 저장되어 있겠네요.

```swift
driver.combineLatest(idEvent, passEvent).map { !$0.0.isEmpty && !$0.1.isEmpty }
```
**combineLatest**를 통해 가장 최근에 입력된 값에 대한 결과 observable을 **buttonEnableEvent** 로 얻어 옵니다.

위에서 말씀드렸다시피 Driver인 **idEvent** 및 **passEvent**는 subscribe시 최근 1개의 결과를 방출합니다. 현재까지는 subscribe (drive)를 하지 않았으므로 결과는 나오는 즉시 release, 즉 사라지게 됩니다.

```swift
buttonEnableEvent.drive(button.rx.isEnabled)
```
**drive(Observable)** 를 통해(드디어 subscribe) **button.rx.isEnabled** observer에 가장 최근의 값 (Bool)을 보내겠네요. 아래와 같이도 사용할 수 있습니다.

```swift
buttonEnableEvent.drive(onNext: { [weak self] (result) in
    self?.button.isEnabled = result
})
```

# 결론
위의 정의 때 말씀드린바와 같이 driver는 UI를 구동시키기 위해 사용되는 observables로 이해하고 사용하면 될 것 같습니다.
