---
layout: post
comments: true
title:  "[Alg] Binary search"
date:   2020-01-16
categories: Algorithm, Binary search
tags: [Algorithm, Binary search]
---

개발을 10년 넘게 해 왔지만 항상 알고리즘을 공부할 때마다 새로운 이유는 무엇일까요?? 그냥 제대로 정리를 해 놓지 않아서 그런 것 같다는 생각이 듭니다. 이번에는 확실히 정리한다는 각오로 포스팅을 시작합니다.

# Binary Search

[Binary Search](https://en.wikipedia.org/wiki/Binary_search_algorithm) 는 일단 아래와 같은 특징을 가집니다.

1. 기본적으로 sorting된 array에서만 값을 찾을 수 있습니다. 
2. Worst case일 때 O(logN), Best case일 때 O(N)의 성능을 보입니다.
3. 표본이 작을 때를 제외하고는 Linear Search 보다 대체적으로 빠릅니다.
4. Hash tables보다는 느리지만, 광범위하게 사용할 수 있습니다.

위 링크에 가보면 Binary Search의 광범위한 variation에 대해서 볼 수 있습니다. Codility에서 제가 해맸던 부분이 마침 Binary Search이기에 이 부분을 좀 더 자세히 보는 것으로... 대체하도록 하겠습니다.

# Binary Search

```
You are given integers K, M and a non-empty array A consisting of N integers. Every element of the array is not greater than M.

You should divide this array into K blocks of consecutive elements. The size of the block is any integer between 0 and N. Every element of the array should belong to some block.

The sum of the block from X to Y equals A[X] + A[X + 1] + ... + A[Y]. The sum of empty block equals 0.

The large sum is the maximal sum of any block.

For example, you are given integers K = 3, M = 5 and array A such that:

  A[0] = 2
  A[1] = 1
  A[2] = 5
  A[3] = 1
  A[4] = 2
  A[5] = 2
  A[6] = 2
The array can be divided, for example, into the following blocks:

[2, 1, 5, 1, 2, 2, 2], [], [] with a large sum of 15;
[2], [1, 5, 1, 2], [2, 2] with a large sum of 9;
[2, 1, 5], [], [1, 2, 2, 2] with a large sum of 8;
[2, 1], [5, 1], [2, 2, 2] with a large sum of 6.
The goal is to minimize the large sum. In the above example, 6 is the minimal large sum.

Write a function:

public func solution(_ K : Int, _ M : Int, _ A : inout [Int]) -> Int

that, given integers K, M and a non-empty array A consisting of N integers, returns the minimal large sum.

For example, given K = 3, M = 5 and array A such that:

  A[0] = 2
  A[1] = 1
  A[2] = 5
  A[3] = 1
  A[4] = 2
  A[5] = 2
  A[6] = 2
the function should return 6, as explained above.

Write an efficient algorithm for the following assumptions:

N and K are integers within the range [1..100,000];
M is an integer within the range [0..10,000];
each element of array A is an integer within the range [0..M].

Copyright 2009–2020 by Codility Limited. All Rights Reserved. Unauthorized copying, publication or disclosure prohibited.
```

위의 문제는 Binary Search section에 있던 문제입니다. 그래서 누구나 '아 이 문제는 Binary Search 로 풀어야 하는구나'라고 생각합니다. 하지만 그냥 저 문제가 딱 하고 나온다면, 전 과연 저 문제를 Binary Search로 풀어야 겠다고 생각할 수 있을까요?...

사실 위의 문제를 가지고 거의 1시간 동안 씨름 하다가 결국 풀이를 찾아 봤습니다. 풀이를 봐도 이해가 안되니 아마 절대 못풀었을듯...

```swift
import Foundation
import Glibc

// you can write to stdout for debugging purposes, e.g.
// print("this is a debug message")

public func solution(_ K : Int, _ M : Int, _ A : inout [Int]) -> Int {
    // write your code in Swift 4.2.1 (Linux)
    
    func solution() -> Int {
        
        var minV = 0
        var maxV = 0
        
        for (_,e) in A.enumerated() {
            minV = max(minV, e)
            maxV += e
        }
        
        var result = 0
        
        while (minV <= maxV) {
            let mid = (maxV + minV) / 2

            if divide(mid, k:K - 1, a:A) == true {
                maxV = mid - 1
                result = mid
            } else {
                minV = mid + 1
            }
        }
        
        return result
    }  
    
    func divide(_ mid: Int, k: Int, a: [Int]) -> Bool {
    
        var sum = 0
        var mK = k
        for (i,e) in a.enumerated() {
            if sum + e > mid {
                sum = e
                mK -= 1
            } else {
                sum += e
            }
            
            if mK < 0 {
                return false
            } 
        }
        
        return true
    }
    
    return solution()
}

함수가 2개 보입니다.

```swift
func solution() -> Int
```
Solution 는 Binary Search 의 기본적인 과정이 진행 됩니다. 그것은

1. min, max값의 중간 값 mid를 구하고,
2. 이 mid 값과 divide함수를 통해 max, min값이 변경되어야 하는지를 구하고
3. 현재 가장 가능성 있는 값 mid를 result에 삽입하는

과정을 거칩니다.


```swift
func divide(_ mid: Int, k: Int, a: [Int]) -> Bool
```

divide에서 하는 과정이 아주 중요합니다. 제일 이해하기 어려웠던 부분이기도 하고 제가 아는 binary search가 맞나 싶을 정도로 활용을 하니 정신 똑바로 차리고 보겠습니다.

### Divide()

```swift
 func divide(_ mid: Int, k: Int, a: [Int]) -> Bool {
    
        var sum = 0
        var mK = k
        for (i,e) in a.enumerated() {
            if sum + e > mid {
                sum = e
                mK -= 1
            } else {
                sum += e
            }
            
            if mK == 0 {
                return false
            } 
        }
        
        return true
    }
```

얘만 좀 자세히 보죠. 뭐하는 녀석인지 부터 살펴보겠습니다. 우선 mid값 (초기 mid 값은 (5 + 15) / 2의 결과 10 입니다.) 과 K값 그리고 array를 받습니다. K는 ** 몇 개의 그룹으로 나눌 수 있는지 **를 나타내는 Int 형 변수 입니다.

우선 첫번 째로

```swift
if sum + e > mid
```
는 ** array의 값을 하나씩 더하면서 이 값이 mid를 넘는지 ** 체크합니다. mid를 넘으면 mid 를 넘게한 그 e로 다시 sum을 셋팅합니다. 그렇다면 그룹을 만들지는 않았지만 일단 mid보다 작은 그룹이 하나 생성되었습니다. 그래서 총 그룹의 갯수인 K도 1을 줄여줍니다. 

계속 이렇게 만들다가 모든 array를 다 돌았는데도 K가 0보다 작아지지 않았다면 그 의미는 그 합이 mid보다 작은 그룹들로 이 배열을 만들 수 있다는 의미가 됩니다.

```swift
if divide(mid, k:K - 1, a:A) == true {
    maxV = mid - 1
    result = mid
} else {
    minV = mid + 1
}
```

그렇다면 mid보다 작은 large sum이 있다는 뜻이겠네요? 그렇다면 위의 로직에서 일단 maxV 값은 이제 mid보다 1 작은 값으로 셋팅합니다. (다시 min, max를 설정한다고 보면 됩니다. 다시 Binary search를 돌기 위한) 그리고 이 mid는 이제 가장 유력한 large sum이 됩니다. 

반대의 경우도 보겠습니다. 


```swift
 func divide(_ mid: Int, k: Int, a: [Int]) -> Bool {
    
        var sum = 0
        var mK = k
        for (i,e) in a.enumerated() {
            if sum + e > mid {
                sum = e
                mK -= 1
            } else {
                sum += e
            }
            
            if mK == 0 {
                return false
            } 
        }
        
        return true
    }
```

다시 divide 코드 인데요. 이제는 모든 mid가 좀 작아졌습니다. 작아진 mid이니만큼 위의 함수에서 그룹을 만들 때 그룹들의 sum이 모두 mid를 넘는다면 return false를 하게 될 가능성이 높아 집니다. 

```swift
if divide(mid, k:K - 1, a:A) == true {
    maxV = mid - 1
    result = mid
} else {
    minV = mid + 1
}
```

divide 함수가 실패했다는 의미는 무엇일까요? 그것은 저 mid값 보다 작은 sum을 가지는 group을 만들 수 없다는 것 입니다. 즉 minV 값이 좀 더 커져야 함을 의미합니다.

이렇게 돌다보면 min이 max보다 큰 경우가 발생합니다. max는 large sum의 minimum 값을 구하려다 보니 그 scope가 점점 작아졌고, min은 그래도 group의 sum을 만족하는 값으로 커지다 보니 결국 둘의 값이 역전하는 것인데요. 결국 최근의 mid 값 이외에 만족하는 값이 없다는 것을 의미하게 됩니다.





