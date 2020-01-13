---
layout: post
comments: true
title:  "[RxSwift][Subject] Subject indepth"
date:   2020-01-13
categories: RxSwift, Subject
tags: [RxSwift, Subject, Development]
---

RxSwift에서 제공하는 [Subject]의 종류는 다음과 같습니다.

1. PublishSubject
2. ReplaySubject
3. BehaviorSubject
4. AsyncSubject

각각 하나씩 살펴보도록 하겠습니다.

# 1. PublishSubject
![publishSubject](publishSubject.png)

각 Observer가 subscribe한 시점부터의 item을 subscribe할 수 있습니다. Subscribe 이전의 값은 얻을 수 없습니다.
만약 subscribe 이후 error가 발생했다면 당연히 error를 배출합니다.

```swift
Observable<Int>.interval(RxTimeInterval.seconds(1), scheduler: MainScheduler.instance)
.take(10)
    .bind(to: self.publishSubject)
    .disposed(by: self.disposedBag)

self.publishSubject
    .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
    .debug()
    .subscribe()
    .disposed(by: self.disposedBag)
```

3초 후의 item에 대해 subscribe를 하므로 아래와 같이 결과가 출력 됩니다.

```swift
2020-01-13 11:22:45.029: ViewController.swift:69 (viewDidLoad()) -> subscribed
2020-01-13 11:22:49.030: ViewController.swift:69 (viewDidLoad()) -> Event next(3)
2020-01-13 11:22:50.030: ViewController.swift:69 (viewDidLoad()) -> Event next(4)
2020-01-13 11:22:51.030: ViewController.swift:69 (viewDidLoad()) -> Event next(5)
2020-01-13 11:22:52.030: ViewController.swift:69 (viewDidLoad()) -> Event next(6)
2020-01-13 11:22:53.030: ViewController.swift:69 (viewDidLoad()) -> Event next(7)
2020-01-13 11:22:54.030: ViewController.swift:69 (viewDidLoad()) -> Event next(8)
2020-01-13 11:22:55.030: ViewController.swift:69 (viewDidLoad()) -> Event next(9)
2020-01-13 11:22:55.030: ViewController.swift:69 (viewDidLoad()) -> Event completed
2020-01-13 11:22:55.030: ViewController.swift:69 (viewDidLoad()) -> isDisposed
```

# 2. ReplaySubject
![](ReplaySubject.png)

ReplaySubject는 subscribe시점과 관계 없이 buffer에 저장되어 있는 최근 n개 + 이후 배출되는 item에 대한 값을 얻을 수 있습니다.

아래는 replaySubject를 하는 부분입니다. 아래와 같이 buffer의 크기를 지정할 수 있습니다.

```swift
var replaySubject: ReplaySubject<Int> = ReplaySubject<Int>.create(bufferSize: 3)
```

ReplaySubject를 이용한 예제 입니다.

```swift
Observable<Int>.interval(RxTimeInterval.seconds(1), scheduler: MainScheduler.instance)
.take(10)
    .bind(to: self.replaySubject)
    .disposed(by: self.disposedBag)

self.replaySubject
    .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
    .debug()
    .subscribe()
    .disposed(by: self.disposedBag)
```

3초 후의 item에 대해 subscribe를 하였지만 buffer가 3개 이므로 이전 3개에 대한 값을 저장하고 있다가 같이 배출해 줍니다. 아래와 같이 결과가 출력 됩니다.

```swift
2020-01-13 11:22:45.029: ViewController.swift:69 (viewDidLoad()) -> subscribed
2020-01-13 11:22:49.030: ViewController.swift:69 (viewDidLoad()) -> Event next(0)
2020-01-13 11:22:49.030: ViewController.swift:69 (viewDidLoad()) -> Event next(1)
2020-01-13 11:22:49.030: ViewController.swift:69 (viewDidLoad()) -> Event next(2)
2020-01-13 11:22:49.030: ViewController.swift:69 (viewDidLoad()) -> Event next(3)
2020-01-13 11:22:50.030: ViewController.swift:69 (viewDidLoad()) -> Event next(4)
2020-01-13 11:22:51.030: ViewController.swift:69 (viewDidLoad()) -> Event next(5)
2020-01-13 11:22:52.030: ViewController.swift:69 (viewDidLoad()) -> Event next(6)
2020-01-13 11:22:53.030: ViewController.swift:69 (viewDidLoad()) -> Event next(7)
2020-01-13 11:22:54.030: ViewController.swift:69 (viewDidLoad()) -> Event next(8)
2020-01-13 11:22:55.030: ViewController.swift:69 (viewDidLoad()) -> Event next(9)
2020-01-13 11:22:55.030: ViewController.swift:69 (viewDidLoad()) -> Event completed
2020-01-13 11:22:55.030: ViewController.swift:69 (viewDidLoad()) -> isDisposed
```

# 3. BehaviorSubject
![](BehaviorSubject.png)

BehaviorSubject를 subscribe하면 가장 최근에 발행한 item을 전달합니다. 만약 아무 item도 전달하지 않았다면 초기값을 전달합니다. 

따라서 아래와 같이 생성시 초기값을 지정해 주는 것이 필요합니다.

```swift
var behaviorSubject: BehaviorSubject<Int> = BehaviorSubject<Int>(value: -1)
```

BehaviorSubject 이용한 예제 입니다.

```swift
Observable<Int>.interval(RxTimeInterval.seconds(1), scheduler: MainScheduler.instance)
.take(10)
    .bind(to: self.behaviorSubject)
    .disposed(by: self.disposedBag)

self.behaviorSubject
    .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
    .debug()
    .subscribe()
    .disposed(by: self.disposedBag)
```

3초 후의 item에 대해 subscribe를 하였지만 이전 결과 역시 전달되므로 Event next(2)를 같이 배출해 줍니다. 아래와 같이 결과가 출력 됩니다.

```swift
2020-01-13 11:46:10.980: ViewController.swift:69 (viewDidLoad()) -> subscribed
2020-01-13 11:46:13.984: ViewController.swift:69 (viewDidLoad()) -> Event next(2)
2020-01-13 11:46:14.981: ViewController.swift:69 (viewDidLoad()) -> Event next(3)
2020-01-13 11:46:15.981: ViewController.swift:69 (viewDidLoad()) -> Event next(4)
2020-01-13 11:46:16.980: ViewController.swift:69 (viewDidLoad()) -> Event next(5)
2020-01-13 11:46:17.981: ViewController.swift:69 (viewDidLoad()) -> Event next(6)
2020-01-13 11:46:18.981: ViewController.swift:69 (viewDidLoad()) -> Event next(7)
2020-01-13 11:46:19.981: ViewController.swift:69 (viewDidLoad()) -> Event next(8)
2020-01-13 11:46:20.981: ViewController.swift:69 (viewDidLoad()) -> Event next(9)
2020-01-13 11:46:20.981: ViewController.swift:69 (viewDidLoad()) -> Event completed
2020-01-13 11:46:20.982: ViewController.swift:69 (viewDidLoad()) -> isDisposed

```

# 4. AsyncSubject
![](AsyncSubject.png)

AsyncSubject는 subscribe 후에 아무 값도 배출하고 있지 않다가 해당 subject가 완료되면 마지막 값을 배출하게 됩니다.

```swift
var behaviorSubject: BehaviorSubject<Int> = BehaviorSubject<Int>(value: -1)
```

AsyncSubject 이용한 예제 입니다.

```swift
Observable<Int>.interval(RxTimeInterval.seconds(1), scheduler: MainScheduler.instance)
.take(10)
    .bind(to: self.asyncSubject)
    .disposed(by: self.disposedBag)

self.asyncSubject
    .debug()
    .subscribe()
    .disposed(by: self.disposedBag)
```

AsyncSubject는 마지막 결과만 리턴하는 이유로 굳이 3초 후에 subscribe를 하지 않았습니다. 결과는 아래와 같습니다.

```swift
2020-01-13 11:52:26.988: ViewController.swift:68 (viewDidLoad()) -> subscribed
2020-01-13 11:52:36.990: ViewController.swift:68 (viewDidLoad()) -> Event next(9)
2020-01-13 11:52:36.990: ViewController.swift:68 (viewDidLoad()) -> Event completed
2020-01-13 11:52:36.990: ViewController.swift:68 (viewDidLoad()) -> isDisposed

```

위의 결과를 보면 약 10초 후 complete이 될 때 마지막 결과 (9)를 배출하고 종료하는 것을 볼 수 있습니다.


[Subject]: 2020-01-10-Subject-Basic.md
