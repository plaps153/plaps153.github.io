---
layout: post
comments: true
title:  "[RxSwift][Operator] Publish & Share"
date:   2020-01-11
categories: RxSwift, Operator
tags: [RxSwift, Publish, Share, Multicast, PublistSubject, Development]
---

Publish, Share는 Observable의 items을 공유하기 위해 사용되는 operator입니다. 하나의 observable에서 emit되는 items을 여러 subscribe를 통해 공유하고 싶을 때 사용합니다. 자세히 살펴보겠습니다.

어떤 한 Observable을 subscribe 한다고 가정해 봅시다. 이 Observable은 network API 를 request하고 결과를 기다리는 observable이라고 해보죠.

```swift
func send<T: Codable>(_ request: Request) -> Observable<T> {
     return Observable<T>.create { observer -> Disposable in
        Alamofire.request(request)
            .validate()
            .responseJSON { response in
                switch response.result {
                case .success:
                    do {
                        let model: T = try JSONDecoder().decode(T.self, from: data)
                        observer.onNext(model)
                    } catch {
                        observer.onError(error)
                    }
                case .failure(let error):
                    observer.onError(error)
                }
        }
 
        return Disposables.create()
}

let observable: Observable<T> = send(generateRequest()).map {print("Subscribe start \($0)")}

//  1
observable.subscribe(onNext: {
  print("Frist subscribe: \($0)")
}).dispose(by: disposedBag)

// 2
observable.subscribe(onNext: {
  print("Second subscribe: \($0)")
}).dispose(by: disposedBag)

```

위와 같은 코드가 있다고 가정해 봅시다. 두 subscribe, 1과 2의 결과는 어떻게 나올까요? 다음과 같이 나올 것 입니다.

```swift
Subscribe start #someResonse
First subscribe: #someRespnse
Subscribe start #otherResponse
Second subscribe: #otherResponse
```

각 subscribe마다 observable이 생성되어 각각에 대해 request를 진행한 후 각각 다른 결과를 return하게 됩니다. 물론 어떤 목적에 의해 위와 같이 각각 다른 request를 subscribe하고 싶은 경우도 있습니다. 하지만 request는 한번만 진행하고 그 결과를 공유하고 싶을 때는 어떻게 할까요?

> 물론 위와같은 network request에 대한 response를 공유하는 경우는 거의 없습니다. 위의 예제 보다는 지속적으로 result를 emit하는 observable을 생각하면 훨씬 이해하기 쉬울 것 입니다.

다음 operator를 통해 가능합니다.

1. Publish
2. Share

> 참고로 publish는 multicast + PublishSubject 의 조합입니다.

# Publish

![publish_image](http://reactivex.io/documentation/operators/images/publishConnect.c.png)

Publish에 대해 알아보기 전에 먼저 publish가 어떤식으로 구현되어 있는지부터 살펴봐야 겠습니다.

```swift
extension ObservableType {

    /**
    Returns a connectable observable sequence that shares a single subscription to the underlying sequence.

    This operator is a specialization of `multicast` using a `PublishSubject`.

    - seealso: [publish operator on reactivex.io](http://reactivex.io/documentation/operators/publish.html)

    - returns: A connectable observable sequence that shares a single subscription to the underlying sequence.
    */
    public func publish() -> ConnectableObservable<E> {
        return self.multicast { PublishSubject() }
    }
}
```

위의 소스는 ObservableType을 확장하여 publish 함수를 구현한 부분입니다. 주목할 부분은 일단 comment에 있습니다.

> Returns a connectable observable sequence that shares a single subscription to the underlying sequence.

뜻은 이 후에 일어나는 sequence이 subscribe를 할지라도 하나의 subscription의 결과를 공유하는 connectable observable sequence를 리턴한다고 되어있습니다. 위에서 설명한 것 처럼 몇 개의 subscribe가 붙을지라도 하나의 subscription 결과만을 공유하게 됩니다.

또 하나 주목해야 할 점은 바로 ***Connectable observable*** 인데요. Connectable observable은 복잡하게 생각할 것 없이 connect operator를 걸어줘야 sequence가 시작되는 observable 입니다. (위의 그림에서 보면 connect() 이후에 observable이 결과를 emit하는 것을 볼 수 있습니다.)


```swift
connectableObservable.connect().disposed(by: disposeBag)
```

> 참고로 Connectable observable operator는 아래와 같이 4 종류가 있습니다.
> 
> 1. Connect
> 2. Publish
> 3. RefCount
> 4. Relay

자 그렇다면 위의 예제에서 각 subscribe가 하나의 결과를 공유하게 만드려면 어떻게 해야할까요? 아래와 같습니다.

```swift
let observable: Observable<T> = send(generateRequest()).map {print("Subscribe start \($0)")}

// 1
let connectableObservable: ConnectableObservable<T> = observable.publish()

// 2
connectableObservable.subscribe(onNext: {
  print("Frist subscribe: \($0)")
}).dispose(by: disposedBag)

// 3
connectableObservable.subscribe(onNext: {
  print("Second subscribe: \($0)")
}).dispose(by: disposedBag)

// 4
connectableObservable.connect().disposed(by: disposedBag)
```

1 과 같이 publish operator를 이용해 observable을 connectableObservable로 변경한 후 2,3과 같이 connectableObservable을 subscribe한 후 (subscribe를 한다 할지라도 connectableObservable은 connect하기 전 까지는 item을 emit하지 않습니다.) 4와 같이 connect() operator를 통해 items을 emit하도록 합니다.

결과는 아래와 같이 나오겠네요.

```swift
Subscribe start #someResonse
First subscribe: #someRespnse
Second subscribe: #someRespnse
```

### refCount
Publish를 설명할 때 refCount를 설명하지 않을 수 없습니다.

![refCount](http://reactivex.io/documentation/operators/images/publishRefCount.c.png)

publish와 함께 사용되는 refCount operator는 첫 번째 subscribe가 실행될 때 자동으로 connect operator를 붙여 줍니다. 따라서 manual하게 connect operator를 호출할 필요가 없습니다. 즉 connectable observable을 일반적인 observable의 동작과 유사하게 만듭니다. (아시다시피 일반적인 observable은 subscribe시에 items를 emit합니다.)

# Share

Share는 publish와 비슷하나 조금 다릅니다. 아래 share의 구현 소스를 보시죠.

```swfit
extension ObservableType {

    /**
     Returns an observable sequence that **shares a single subscription to the underlying sequence**, and immediately upon subscription replays  elements in buffer.
     
     This operator is equivalent to:
     * `.whileConnected`
     ```
     // Each connection will have it's own subject instance to store replay events.
     // Connections will be isolated from each another.
     source.multicast(makeSubject: { Replay.create(bufferSize: replay) }).refCount()
     ```
     * `.forever`
     ```
     // One subject will store replay events for all connections to source.
     // Connections won't be isolated from each another.
     source.multicast(Replay.create(bufferSize: replay)).refCount()
     ```
     
     It uses optimized versions of the operators for most common operations.
     - parameter replay: Maximum element count of the replay buffer.
     - parameter scope: Lifetime scope of sharing subject. For more information see `SubjectLifetimeScope` enum.
     - seealso: [shareReplay operator on reactivex.io](http://reactivex.io/documentation/operators/replay.html)
     - returns: An observable sequence that contains the elements of a sequence produced by multicasting the source sequence.
     */
    public func share(replay: Int = 0, scope: SubjectLifetimeScope = .whileConnected)
        -> Observable<Element> {
        switch scope {
        case .forever:
            switch replay {
            case 0: return self.multicast(PublishSubject()).refCount()
            default: return self.multicast(ReplaySubject.create(bufferSize: replay)).refCount()
            }
        case .whileConnected:
            switch replay {
            case 0: return ShareWhileConnected(source: self.asObservable())
            case 1: return ShareReplay1WhileConnected(source: self.asObservable())
            default: return self.multicast(makeSubject: { ReplaySubject.create(bufferSize: replay) }).refCount()
            }
        }
    }
}
```

다른 것은 볼 필요가 없고 이 부분이 중요 합니다. 

```swift
        case .whileConnected:
            switch replay {
            case 0: return ShareWhileConnected(source: self.asObservable())
            case 1: return ShareReplay1WhileConnected(source: self.asObservable())
            default: return self.multicast(makeSubject: { ReplaySubject.create(bufferSize: replay) }).refCount()
            }
        }
```
publish 차이점은 publish는 item을 PublishSubject를 통해 배출했다면, share는 replySubject를 사용합니다. ***즉 subscribe 이전의 data도 관심을갖고 있다는 의미이겠지요.***

좀 더 깊이 있게 들어가 보겠습니다.

```swift
share(replay: 0, scope: .whileConnected)
```

우선 reply와 scope이 의미하는 바를 살펴봐야 겠습니다.
### Replay

![Reply](https://miro.medium.com/max/1029/1*iI-XytOTmOtG23wKegje9g.png)

*** [이미지 참고]: https://medium.com/gett-engineering/rxswift-share-ing-is-caring-341557714a2d ***

몇 개의 item을 subscribe에 전달할 것인지를 나타냅니다. Subject 중 ReplaySubject를 생각하면 이해가 빠를 것 같습니다. ReplaySubject는 subscribe를 하면 buffer에 쌓여있던, subscribe 이전의 item도 한번에 배출하게 되는데요 여기서도 마찬가지로 replay는 버퍼의 갯수를 의미 합니다. 0개는 기존 item를 상관하지 않겠다는 의미이니 publish operator와 동일하게 동작하겠네요. 1개 이상 부터는 subscirbe시 subscribe 이전의 item이 replay 에 명시된 숫자 크기만큼 전달됩니다.


### Scope
Scope은 "item들이 언제 replay될 것인지"를 결정합니다. 종류는 아래 2가지가 있습니다.

1. .whileConnected
2. .forever

> RxSwift core contributor인 [Shai Mishali의 말]에 의하면 99%의 경우 .whileConnected를 사용할 것이라고 말하고 있습니다. 하지만 .forever를 알아두는 것도 나쁘지는 않습니다.

1. .whileConnected : iOS의 ARC와 유사합니다. item들은 referenc-counting manner 방식으로 replay 됩니다. 어떤 의미냐 하면, share operator에 의해 share되는 observable의 subscriber 갯수가 1개 이상일 때는 buffer의 크기대로 item을 저장하고, 즉 replay가 1이라 가정할 때 subscribe를 하면 subscribe 이전에 받았던 1개를 배출하고 그 뒤에 발생하는 item은 차례대로 배출하지만, subscriber가 없어지면 (0개가 되면) 공유하기 위해 사용되는 stream의 cache가 clear 되게 됩니다. 즉 subscriber의 갯수가 shared stream을 유지하는 기준이 되는 거죠. 즉 subscrib가 없다가 다시 subscribe가 시작되면 replay buffer에 있던 item은 모두 clear되었으니, 이제 그 이후에 배출되는 것에 대해서 다시 버퍼를 채우게 됩니다.

2. .forever : 말 그대로 subscriber의 갯수에 상관 없이 그 share stream이 유지되는 겁니다. subscribe 갯수가 1->0으로 되었다가 다시 subscribe가 생겨도 기존에 shared stream의 cache가 clear 되지 않아 기존 item이 배출되게 됩니다. 원하지 않는 데이터가 subscribe될 수 있으니 유의하여 사용해야 할 것 같습니다.

<img src="https://miro.medium.com/max/1510/1*wKKemAmseKM6boREZ9i6fg.png" width="400" height="400" />
<img src="https://miro.medium.com/max/1668/1*F7ud6kKEL3TPsbpjAfioTQ.png" width="400" height="400" />
 
[Shai Mishali]: https://medium.com/gett-engineering/rxswift-share-ing-is-caring-341557714a2d
