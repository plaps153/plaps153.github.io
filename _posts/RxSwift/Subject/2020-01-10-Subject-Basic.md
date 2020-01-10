---
layout: post
comments: true
title:  "[RxSwift][Subject] Subject Basic"
date:   2020-01-10
categories: RxSwift, Subject
tags: [RxSwift, Subject, Development]
---

ReactiveX에서 Subject는 bridge 혹은 proxy라 불리웁니다. 왜 그렇게 불리는지 궁금했는데 Subject를 제대로 알고나니 그 의미가 좀 더 명확해 졌습니다.

Bridge(다리), proxy(대리인)는 모두 둘 사이를 잇는 느낌이 강합니다. Subject는 때로는 observer로 동작하며 다른 observable을 구독(subscribe)하고, 때로는 observable로서 동작을 함으로서 또 다른 observer에게 subscribe 당하는 역할을 하게 됩니다. 둘 사이를 잇는 느낌이 나죠?

# Hot & Cold Observable
Subject에 대해 살펴보기 전에 먼저 observable의 종류에 대해 살펴보겠습니다.

### Cold observable
Cold observable은 subscribe 될 때만 items을 emit하는 observable을 가리킵니다. 즉 subscribe시 부터 item을 emit(방출)함으로서 Observer가 모든 item을 구독하는 것을 보장합니다.

### Hot observable
Hot Observable은 subscribe에 상관없이, 즉 관찰하는 대상이 있건 없건 만들어지자 마자 items을 emit하는 observable을 가리킵니다. Emit 중간 아무때나 subscribe를 시작할 수 있습니다. 물론 모든 item을 구독할 수 없을 가능성이 큽니다.

###  Hot & Cold observable and Subject와의 관계

Subscribe를 설명하다 말고 갑자기 뜬금없이 Hot & Cold observable을 설명한 이유는 뭘까요? 그것은 바로 Subject는 cold observable을 hot observable로 바꿀 수 있기 때문입니다.

그 이유를 [ReactiveX] 에서는 다음과 같이 말하고 있습니다.

> Because a Subject subscribes to an Observable,

일단 Subject가 observable, 즉 cold observable을 subscribe하기에 hot으로 변한다는 이야기 같습니다. 예제가 좀 필요할 것 같습니다. 저와 같은 의문을 품은 분이 있군요 [참고]

```swift
let theSubject = PublishSubject<String>()
let theObservable = Observable.just("Hello?")
```

Publish subject에 대해서는 아직 다루지 않았지만 일단 subject의 한 종류라고만 알아둡시다. 두 subject(theSubject)와 observable(Cold, theObservable)을 어떤식으로 관계지을 수 있을까요? Subject가 어떤식으로 observable을 subscribe할 수 있을까요? 두 가지 방법이 가능합니다.

1. Bind
2. Subscribe (말 그대로)

먼저 Subscribe 부터 보겠습니다.

```swift
theObservable.subscribe(onNext: {theSubject.next($0)}).disposed(by: disposedBag) // next event만을 받음
```
혹은
```swift
theObservable.subscribe(theSubject).disposed(by: disposedBag) // 모든 결과를 받음
```

어떤가요? 깔끔하죠? 이제 theObservable은 subscribe되고 그 결과는 subject로 들어가게 됩니다. 본의아니게 subject가 observable을 subscribe하게 되었네요.

그렇다면 bind를 통해서는 어떻게 subscribe하는지 보겠습니다.

```swift
theObservable.bind(to: theSubject).disposed(by: disposedBag)
```

TheObservable을 theSubject와 묶습니다. 자연스럽게 theObservable의 결과가 subject로 흘러갑니다. 

> bind와 subscribe는 error를 처리하느냐 처리하지 않느냐의 차이일 뿐 입니다. 아래 소스에서 확인하시죠
>```swift
>    /**
>    Subscribes an element handler to an observable sequence.
>    In case error occurs in debug mode, `fatalError` will be raised.
>    In case error occurs in release mode, `error` will be logged.
>    - parameter onNext: Action to invoke for each element in the observable sequence.
>    - returns: Subscription object used to unsubscribe from the observable sequence.
>    */
>    public func bind(onNext: @escaping (Element) -> Void) -> Disposable {
>        return self.subscribe(onNext: onNext, onError: { error in
>            rxFatalErrorInDebug("Binding error: \(error)")
>        })
>    }
>```
> Bind로는 error처리를 할 수 없습니다. 따라서 error처리가 필요할 때는 반드시 subscribe를 사용해야 합니다. [참고1]

이정도면 Subject의 기본적인 개념에 대해서는 충분히 다뤘다 생각합니다. 다음 포스트에서 4가지 subject에 대해 다루겠습니다.

[참고1]: https://stackoverflow.com/questions/55294293/rxswift-bindonnext-vs-subscribeonnext
[참고]: https://stackoverflow.com/questions/51985484/in-rxswift-how-can-i-set-up-a-subject-to-observe-another-observable
[ReactiveX]: http://reactivex.io/documentation/subject.html
