---
layout: post
comments: true
title:  "[Swift] Generic Basic"
description: "Swift generic basic"
date:   2020-01-10
category: Swift, Generic
tags: [Swift, Generic, Development]
---

Generic 정리는 [Swift 공식문서]를 참고하였습니다.


## Generics
Generic 은 Swift의 가장 강력한 기능 중 하나 입니다. 또한 대부분의 swift 라이브러리는 generic을 이용하여 구현되었습니다. 심지어 Array와 Dictionary도 모두 generic colleciton 입니다. Array는 Int만 저장하게 할 수도 있고 String만 저장하게 할 수도 있습니다. 또한 어떤 타입이든지 저장하게 할 수 도 있습니다. 

## Generic이 해결한 문제

만약 두 숫자를 바꾸는 함수를 구현한다고 합니다.

```swift
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

해당 함수를 이용한 예제는 다음과 같습니다.
```swift
var someInt = 3
var anotherInt = 107
swapTwoInts(&someInt, &anotherInt)
print("someInt is now \(someInt), and anotherInt is now \(anotherInt)")
// Prints "someInt is now 107, and anotherInt is now 3"
```

그런데 여기서 잠깐, Int 뿐만 아니라 다른 type (*String*, *Double*)도 변경하고 싶다면, 어떻게 해야 할까요? 비슷한 함수를 아래와 같이 작성해야 할까요?

```swift
func swapTwoStrings(_ a: inout String, _ b: inout String) {
    let temporaryA = a
    a = b
    b = temporaryA
}

func swapTwoDoubles(_ a: inout Double, _ b: inout Double) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

비슷한 함수를 여러번 작성하는 것은 낭비 입니다. 이를 해결하기 위해 generic이 사용됩니다.

# Generic functions

두 객체를 바꾸는 함수를 generic으로 구현해 보겠습니다.

```swift
func swapTwoValues<T>(_ a:inout T, b:inout T) {
    let tempA = a
    a = b
    b = tempA
}
```

Generic에서는 실제 type (Int 등) 대신에 placeholder type name (여기에서는 T)를 사용합니다. T가 특별한 것은 이것이 의미하는 바가 **a와 b는 같은 타입을 갖는다**는 것이라는 겁니다.

또한 placeholder type name T는 braket <>안에 들어가게 되는데 이는 앞으로 이 T가 이 swapTwoValues의 type으로서 사용될 것이라는 것을 의미하게 됩니다. (Type은 1개 이상 사용될 수 있습니다.)

자 이제는 단 하나의 함수로 아래와 같은 실행이 가능해 졌네요.

```swift
var someInt = 3
var anotherInt = 107
swapTwoValues(&someInt, &anotherInt)
// someInt is now 107, and anotherInt is now 3

var someString = "hello"
var anotherString = "world"
swapTwoValues(&someString, &anotherString)
// someString is now "world", and anotherString is now "hello"
```

# Type Parameters

위에서 말한 placeholder type T가 바로 Type parameter입니다.
물론 위와 같이 braket<> 사이에 적습니다.

Type parameter를 braket사이에 적음으로서, 해당 타입을 
1. **parameter**에 쓸 수도 있고
2. **return type**에도 쓸 수 있습니다. 
3. 또한 함수의 **type annotation**에도 쓸 수 있습니다.
    ```swift
    var myStr: T
    ```

각각의 경우 type은 ***실제로 사용될 때*** 실제 type으로 변환되게 됩니다.

또한 위에서 말한대로 type은 한 개 이상 사용할 수 있습니다. (T , U , V , Key , Value , and so on)

>단 type parameter는 camel case (맨 앞 대문자)로 사용하길 권하고 있는데 value와 구분을 짓기 위함입니다.

# Generic type 확장

Generic을 extension할 때에는 따로 extension 정의 옆에 (braket을 이용하여)  type을 적을 필요가 없습니다. Type parameter는 original type 정의 (Stack<Element>)에서 가져오기 때문이죠. 아래 Stack이라는 struct를 확장한다고 가정해 보겠습니다.

```swift
struct Stack<Element> {
    var items = [Element]()
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
}
```

위의 Stack struct를 확장해보면
```swift
extension Stack {
    var topItem: Element {
        return items.isEmpty ? nil : items[items.count - 1]
    }
}
```

로 확장할 수 있습니다. topItem의 type anotation 으로 original의 Element를 가져다 쓸 수 있는 것을 확인할 수 있습니다.

# Type Constraints
위의 swapTwoValue(_:_:)과 Stack의 type은 어떤 타입도 받을 수 있습니다. 하지만 때때로 위의 type에 제한 (constraints)을 걸어야 할 때가 있습니다. 예를 들어 보겠습니다.

### Type constraints syntax
Dictionary를 생각해 보겠습니다. Dictionary의 key는 반드시 [hashable] 해야 합니다. 만약 custom dictionary를 구성할 때 아래와 같이 한다고 가정해 보겠습니다.

```swift
func MyCustomDictionary<T, U>(key: T, value: U) {
    ...
}
```

일반적으로 dictionary에 Key로 사용되는 type **T**는 반드시 hashable해야 합니다. 위와같이 쓰면 hashable하지 않은 type도 key로 들어오게 됩니다. 그렇다면 이제 T에 제약을 걸어줄 때가 된 것 같습니다.

아래와 같이 하면 어떨까요?
```swift
func MyCustomDictionary<T: Hashable, U: SomeProtocol>(key: T, value: U) {
    ...
}
```

type **U**의 someProtocol은 제가 임의로 제약을 준 것입니다. 중요한 것은 type **T**의 제약 **Hashable** 입니다. 이로서 type T에는 hashable한 type만 허락되게 됩니다.

### Type constraints in Action
실제 type constraints가 함수 내에서 어떻게 사용되는지를 살펴보겠습니다. 어렵지 않습니다. :)

```swift
func findIndex(ofString valueToFind: String, in array: [String]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

위의 일반적인 (nongeneric)한 함수를 아래와 같이 generic한 함수로 변경해 보겠습니다.

```swift
func findIndex<T>(ofString valueToFind: T, in array: [T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind { // 1
            return index
        }
    }
    return nil
}
```

위의 소스는 컴파일이 되지 않습니다. 바로 위의 1번 라인에서 발생하는 이슈 때문인데요. A와 B를 비교하기 위해서는 (=) 반드시 type T가 Equatable protocol를 따라야 합니다. 하지만 위의 소스만으로는 parameter로 들어오는 모든 T가 Equatable protocol을 준수하는 type만 들어온다고 보장할 수 없습니다.

그렇다면 어떻게 바꿔야 할지 감이 오시죠? 아래와 같이 바꿔봤습니다. Type parameter에 Equatable 제약을 줬습니다. 이제 type T는 반드시 Equatable protocol을 따르는 type들만 들어오겠네요.

```swift
func findIndex<T: Equatable>(ofString valueToFind: T, in array: [T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

아래 예제로 위의 함수 동작을 확인할 수 있습니다.

```swift
let doubleIndex = findIndex(of: 9.3, in: [3.14159, 0.1, 0.25])
// doubleIndex is an optional Int with no value, because 9.3 isn't in the array
let stringIndex = findIndex(of: "Andrea", in: ["Mike", "Malcolm", "Andrea"])
// stringIndex is an optional Int containing a value of 2
```

# Associate Type
간단히 말해 protocol에서 사용되는 generic을 associate type이라고 생각하시면 편할 것 같습니다. 예를 보겠습니다.

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
```
위에서 Item이 바로 associate type으로 선언된 placeholder name 입니다. 이 type은 해당 protocol이 사용되어질 때 비로소 그 type이 정해집니다. 

위의 protocol을 실제로 어떻게 사용하는지 보시죠. 우선 nongeneric struct에 사용하는 예 입니다.

```swift
struct IntStack: Container {
    // original IntStack implementation
    var items = [Int]()
    mutating func push(_ item: Int) {
        items.append(item)
    }
    mutating func pop() -> Int {
        return items.removeLast()
    }
    // conformance to the Container protocol
    typealias Item = Int
    mutating func append(_ item: Int) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Int {
        return items[i]
    }
}
```

우선 위 Container 프로토콜을 모두 준수하고 있습니다. 두 개의 함수 및 하나의 변수를 모두 구현하였습니다. 또한 눈여겨 볼 부분이 있는데요.

```swift
typealias Item = Int
```

typealias는 다들 아실 것이라 생각됩니다. 특정 타입 (rhs)을 내가 원하는 type (lhs)로 매칭시킬 때 사용할 수 있습니다. 이제 Item 는 Int으로 표현되죠. 
공식 문서에는 아래와 같이 설명되어 있습니다.
>The definition of typealias Item = Int turns the abstract type of Item into a concrete type of Int for this implementation of the Container protocol.

즉 추상 type이었던 Item이 실제 타입으로 바뀌는 부분입니다.

__물론 swift의 type 추론으로 위의 *typealias Item = Int*는 생략이 가능합니다.__

아래 generic struct에 사용된 예제도 보겠습니다.

```swift
struct Stack<Element>: Container {
    // original Stack<Element> implementation
    var items = [Element]()
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
    // conformance to the Container protocol
    mutating func append(_ item: Element) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Element {
        return items[i]
    }
}
```

물론 위에도 아래 소스가 생략되어 있습니다. type 추론을 통해서 생략이 되었습니다.

```swift
typealias Item = Element
```

### Associate type에 Constraints(제약) 걸기
아래와 같이 Associate type에 constraints를 걸 수도 있습니다.

```swift
import UIKit

protocol Container {
    associatedtype Item: Equatable
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}

struct Stack<Element: Equatable>: Container {
    var arr:[Element] = [Element]()

    mutating func append(_ item: Element) {
        arr.append(item)
    }

    var count: Int {
        return arr.count
    }

    subscript(i: Int) -> Element {
        return arr[i]
    }
}

func example() {
    var stack = Stack<Int>()
    stack.append(3)
    print(stack.count)
}

example()
```
이 Container protocol을 따르기 위해서는 container의 Item 으로 사용되는 Element 도 반드시 Equatable을 준수해야 합니다. ()

# Associated type을 지정함으로서 기존 type을 확장하기

아래와 같이 기존 타입을 확장할 수 있습니다.

```swift
extension Array: Container {}
```

Array 의 Element type은 Container의 Item으로 사용되게 됩니다. 또한 위와 같이 하면 Array는 이미 Container의 필수구현항목을 모두 구현하고 있기에 문제 없이 Container protocol이 적용 됩니다. 그렇게 되면 Array 는 Container로서 사용될 수 있게 됩니다. Parameter로 받을 때 아래와 같이 받을 수 있게 됩니다.

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}

extension Array: Container {}

var array:Array<Int> = Array<Int>()

func takeContainer<T: Container>(_ container:T) {
    var a:T = container
}
takeContainer(array)
```
위와 같이 Container를 받는 parameter에 Array가 들어갈 수 있게 됩니다.

[hashable]: https://developer.apple.com/documentation/swift/hashable
[Swift 공식문서]: https://docs.swift.org/swift-book/LanguageGuide/Generics.html
