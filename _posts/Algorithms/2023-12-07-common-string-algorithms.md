---
title: "[Swift 알고리즘] 공통 글자수 출력 문제"
description: "[Swift 알고리즘] 공통 글자수 출력 문제"
categories: Algorithms
tags: [Swift, Algorithms]
sidebar: 
  nav: "docs"
header:
  teaser: /assets/images/algorithms.jpg
---
>Given two strings, find the number of common characters between them.

위 문제를 풀어보도록 하자. 공통된 글자의 수를 구하는 문제인데 간단한 문제이다. 풀이 자체는 간단하지만 풀이에 접근하는 여러가지 방식과 그 내부에 내포하는 여러가지 지식들을 탐구해보고자 글을 작성한다.

```swift
func solution(s1: String, s2: String) -> Int {
    var s1Arr: [String] = s1.map(String.init)
    var s2Arr: [String] = s2.map(String.init)
    var commonCount: Int = 0
    
    for i in 0..<s1Arr.count {
        if s2Arr.contains(s1Arr[i]) {
            s2Arr.remove(at: s2Arr.firstIndex(of: s1Arr[i])!)
            commonCount += 1
        }
    }
    return commonCount
}
```

최초에는 위와 같이 코드를 작성했다. 사실 큰 문제는 없다. 강제 언래핑을 사용하긴 했지만 if문을 사용했기 때문에 nil 값을 반환할 경우를 배제 시켰기 때문에 이론대로라면 에러가 뜨지 않을것이다. 하지만 강제 언래핑은 지양해야 하기 때문에 옵셔널 바인딩을 사용해보도록 하자.

```swift
func solution(s1: String, s2: String) -> Int {
    var s2Arr = Array(s2)
    var commonCount = 0

    for char in s1 {
        if let index = s2Arr.firstIndex(of: char) {
            s2Arr.remove(at: index)
            commonCount += 1
        }
    }
    return commonCount
}
```

기존 코드에서 `if let`을 사용해서 옵셔널 바인딩을 하여 해결하였는데 또한가지 차이점이 있다.  `.map(String.init)`을 사용해서 String 배열로 변환해서 사용했던 기존의 방식과 다르게 Array(s2)로 바로 타입 캐스팅을 하여 사용 했다. 그리고 s1의 인덱스를 사용하는 것이 아니라 바로 `for-in`문을 사용하여 character를 사용하였다.

`s2.map(String.init)`과 `Array(s2)`는 비슷한 형태로 결과를 출력하지만 엄밀히 보면 다르다. `s2.map(String.init)`는 `[String]` 타입이고 `Array(s2)`는 `[Character]` 타입이다. String은 string.element 즉, character의 컬렉션이기 때문에 앞의 방법은 문자열의 요소들을 탐색하고 다시 문자열의 형태로 초기화해서 배열에 담은 것이고, 뒤의 방법은 character의 컬렉션으로 이루어진 string을 그대로 쪼개서 배열 안에 담은 것이다. 따라서 비슷해보이지만 타입에 유의해서 사용해야한다.

그렇다면 또 다른 방법은없을까? 

```swift
func solution(s1: String, s2: String) -> Int {
    var count1: [Character: Int] = [:]
    var count2: [Character: Int] = [:]

    // s1의 각 문자에 대한 빈도수 계산
    for char in s1 {
        count1[char, default: 0] += 1
    }

    // s2의 각 문자에 대한 빈도수 계산
    for char in s2 {
        count2[char, default: 0] += 1
    }

    // 공통 문자의 빈도수 합산
    var commonCount: Int = 0
    for (char, count) in count1 {
        commonCount += min(count, count2[char, default: 0])
    }

    return commonCount
}
```

위 방법은 딕셔너리를 사용하여 각각의 문자에 대한 빈도수를 계산하고 공통 문자의 빈도수를 합산하는 방식이다. 기존의 방식은 `O(n * m)`의 시간 복잡도를 가지는 반면 이 방식은 `O(n + m)`의 시간 복잡도를 가지기 때문에 시간 복잡도가 선형적으로 증가하여 훨씬 더 효율적이다.

단순히 미적으로 보기에는 기존의 코드들이 더 짧기 때문에 좋은 코드 처럼 보이지만 상황에 따라서 기능적으로는 이 코드가 더 효율적일 수 있다. 이처럼 단순히 문제 풀이를 위해 방법들을 외우는 것도 좋지만 어떤 원리에서 함수가 작동 하는지 고민하고 그것을 위해서 어떤 방법들이 있는지 여러가지 방법들을 고민하고 비교하는 과정을 거치는 것 또 한 큰 의미가 있는 것 같다. 면접이나 코딩 테스트 때문에 어쩔수 없이 시험공부 하듯이 하게 되는 측면이 있지만 실력 향상을 위해서는 복잡한 문제보다는 단순한 문제부터 다방면으로 생각해보는 연습을 하는것도 추천한다.
