---
layout: post
comments: true
title:  "[Metal] Metal Basic #1"
date:   2020-02-25
categories: Metal
tags: [Metal]
---

<p align="center">
  <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn%3AANd9GcT9AHbNXfr601XLX2QjGfGKFPzTqDordgE7z9nx2BvovKfoOpHK" />
</p>

5년 전 쯤 OpenGL을 이용하여 video player를 만든 경험이 있습니다. 물론 AVPlayer를 통해서도 아주 손쉽게 video player를 구현할 수 있습니다만... OpenGL을 썼던 이유는 360 Camera preview video player를 만들기 위함이었죠. 이제 OpenGL도 [deprecated](https://developer.apple.com/documentation/opengles/) 된 마당에 새로운?(이미 [2014.6월에 발표된...ㅎ](https://ko.wikipedia.org/wiki/%EB%A9%94%ED%83%88_(API))) Metal API를 공부하면서 기존의 앱을 업그레이드 해보려 합니다.

# 1. 들어가면서

이번 Basic에는 아래 3개의 case에 대해서 다뤄보도록 하겠습니다. 

1. What is the Metal
2. What is the MetalKit

일단 주제만 나열했는데도 정신이 혼미해지고 당이 딸리기 시작하네요. 그만큼 제게는 무거운 주제인 것 같습니다. 하지만... 시작이 반이라 하였으니 시작해 보도록 하죠. (이미 반 했습니다! ㅎㅎ)


## What is the Metal

2014년, Apple 수석 부사장인 [Craig Federighi] 는 WWDC 2014에서 Metal을 처음 소개합니다. 이 분이 나와서 발표하는 날은 긴장을 좀 해야죠? iOS 8부터 지원되었으며 A7 코어를 사용하는 iPhone 및 iPad 제품에 지원이 되다가 이후 tvOS, macOS 등으로 그 여역을 넓혀 갔습니다. ([History 참고])

좀 지루한 이야기를 계속 해보겠습니다.

### OpenGL vs Metal
[OpenGL]은 [cross-language], [cross-platform] 입니다. 즉 동일한 문법의 언어로 다양한 platform에서 graphic관련 작업을 할 수 있게 도와주는 API 입니다. 이와는 반대로 Metal API는 Swift, Objective-C 에 의해 호출될 수 있는 객체지향 API 입니다. 물론 [Apple processor] 에서만 동작하는 API지요.

또한 Metal은 다음과 같은 특징을 가집니다.

1. Low-overhead interface
- Metal은 성능에 있어 병목현상을 제거하도록 설계되었습니다. 예를들어 [state validation]과 같은 경우 입니다.
- Metal framework는 buffer와 texture 객체
### Draw call

Metal은 low-level,low-overhead HW-accelerated 3D 그래픽 엔진 입니다. Metal과 GPU간의 layer는 OpenGL의 그것과는 다르게 상대적으로 아주 얇습니다. 그 의미는 OpenGL에 비해 Metal이 overhead가 적다는 이야기 이겠지요. 아래 그림을 한번 보겠습니다.

<p align="center">
  <img src="https://architosh.com/wp-content/uploads/2015/06/metal_1.jpg" width="325" height="255" />
  <img src="https://architosh.com/wp-content/uploads/2015/06/metal-2.jpg" width="325" height="255" />
</p>

상대적으로 두꺼운 layer를 가진 OpenGL abstraction layer는 다양한 플랫폼에서 OpenGL API를 사용할 수 있도록 하는 잇점이 있습니다. 하지만 이로인해 발생하는 가장 큰 문제점은 바로 상당한 양의 overhead가 발생한다는 것입니다.

이 overhead로 인해 발생하는 가장 큰 문제점은 바로 [draw call] throughput이 떨어진다는 것입니다. Draw call은 CPU가 GPU에게 어떤 object를 한 frame 기간동안 render하라고 명령하는 것이라 볼 수 있습니다. 이미 CPU는 고성능의 GPU를 따라가기 벅차고 주어진 시간동안 high level graphics 를 모두 수행하는 것도 벅차게 됩니다. 이러한 overhead가 발생하는 가장 큰 이유는 draw call이 진행될 때 shader compile과 state validation이 CPU에서 일어나기 때문입니다. 이 때문에 이 기간동안 수행될 수 있는 physics processing과 기타 object를 더 그릴 수 있는 시간이 줄어들게 됩니다.

Metal의 draw call time은 OpenGL과 비교할 때 최대 약 10배로 많다고 합니다. 그만큼 GPU의 idle time도 줄일 수 있고, CPU도 다른 목적으로 더 사용할 수 있게 됩니다.





## MetalKit

[state validation]: https://www.khronos.org/registry/OpenGL-Refpages/gl2.1/xhtml/glValidateProgram.xml
[draw call]: 
[OpenGL]: https://en.wikipedia.org/wiki/OpenGL
[Craig Federighi]: https://www.apple.com/kr/leadership/craig-federighi/
[History]: https://en.wikipedia.org/wiki/Metal_(API)
[cross-language]: https://en.wikipedia.org/wiki/Language-independent_specification
[cross-platform]: https://en.wikipedia.org/wiki/Cross-platform
[Apple processor]: https://en.wikipedia.org/wiki/Apple-designed_processors
