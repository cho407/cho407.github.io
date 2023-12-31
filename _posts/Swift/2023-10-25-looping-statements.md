---
title: "Swift 반복문 for-in, while, repeat-while의 비교"
description: "Swift의 반복문을 공식문서와 실제 사용에 따른 성능적 비교와 사용 상황 정리"
categories: Swift
tags: [개발, Swift, SwiftUI, 문법, 반복문]
sidebar: 
  nav: "docs"
header:
  teaser: https://i.imgur.com/CEveOns.png
---

코딩을 하면서 가장 많이 가장 유용하게 쓰이는 구문 중 하나가 **Loop Statements(반복문)** 일 것이다. 상황에 따라 구분되어 사용되기도 하지만 대부분의 경우 자신이 익숙한 구문을 이용하는 경우도 적지 않다. 그래서 조금 더 자세히 비교해보고자 한다.

Swift는 세가지 loop statements를 가지고 있다:  `for`-`in` statement,  `while` statement, 그리고  `repeat`-`while` statement. 하나씩 천천히 살펴보도록 하자.

### Break Statement 와 Continue Statement
Loop statements의 제어 흐름은 `break`와 `continue` 구문에 의해 변경될 수 있다. Break statement의 경우에는 loop statements, if statement, switch statement 을 종료시키는 역할을 하는데 다음과 같은 형태로 사용된다.

```swift
break 
break <#label name#>
```

조금 더 이해를 돕자면 다음과 같다.
```swift
for i in 1...5 {
  
  if i == 3 {
    break
  }
 
print(i)
}

// output: 1 2
```

Continue statement의 경우에는 현재의 loop statement를 종료하고 다음 loop를 실행시키는 구문이다. 다음과 같은 Break statement와 동일한 형태를 가진다.

```swift
continue
continue <#label name#>
```

실제 용례를 살펴보면 다음과 같다.
```swift
for i in 1...5 {
  
  if i == 3 {
    continue
  }
 
print(i)
}

// output: 1 2 4 5
```

### For-In Statement
`for`-`in` 문은 [Sequence](https://developer.apple.com/documentation/swift/sequence) protocol을 준수하는 collection(또는 any type) 각 항목에 대해 코드 블록을 한 번씩 실행하도록 한다.

>**Sequence**
요소에 순차적이고(sequential) 반복된(iterated) 접근을 하는 타입

`for`-`in` 구문은 다음의 형식을 가진다:

```swift
for <#item#> in <#collection#> {
   <#statements#>
}
```

`for`-`in` 문의 구동 방식을 살펴보면 먼저 iterator type([IteratorProtocol](https://developer.apple.com/documentation/swift/iteratorprotocol)protocol을 준수하는 type)의 값을 얻기 위해 collection expression(콜렉션 표현식)에서 makeIterator() method를 호출한다. 그 후  iterator에서 next() method를 호출하여 loop를 프로그램이 실행한다. 만약 vlaue가 `nil`값이 아니라면 아이템 패턴에 할당되어 구문을 실행하며 loop를 시작하고 `nil`값인 경우 할당과 실행을 하지 않고 `for`-`in`끝내게 된다.

>**IteratorProtocol**
>Sequence에 한번에 하나씩 접근하게 하는 타입으로 Sequence protocol과 밀접하게 연관되어 있는데  Sequence 는 콜렉션의 요소에 접근하기 위해 iterator를 생성한다.

일반적으로 `for`-`in`문은 컬렉션(배열, 딕셔너리, 집합 등)의 각 요소를 반복하여 접근할 때나 특정 횟수만큼 반복 실행할 때 사용 되는데 다음과 같은 장단점이 있다.

**장점**
- 코드가 간결하고 읽기 쉽다.
- 컬렉션의 요솔르 순차적으로 반복하며 접근하기 때문에 반복의 범위나 종료 조건을 명시적으로 지정할 필요가 없다.

**단점**
- 일반적인 반복 구조에서의 조건 제어나 중간에 반복을 조절하는 데 제한적일 수 있다.

### While Statement
`while` 문은 조건이 참으로 남아있는 한 코드가 반복적으로 실행되도록 한다. 

`for`-`in` 구문은 다음의 형식을 가진다:
```swift
while <#condition#> {
   <#statements#>
}
```

`while`문의 구동 방식은 우선 조건이 평가되는데 true 이면 다음 단계로 넘어가고 false 이면 `while`문을 멈춘다. true 의 조건이 충족된다면 구문을 실행시키고 다시 이전 단계로 돌아오며 조건이 false 가 나올 때 까지 반복되는 방식이다. condition에 대한 판별의 return값은 당연히 Bool type이어야 한다. 하지만 Optional Binding의 값을 가질 수 있는데 다음의 예시와 함께 살펴보자.

```swift
var number: Int? = 10

while let actualNumber = number {
    print(actualNumber)
    number = nil
}
```
위 코드에서 `while let actualNumber = number` 부분은 `optional binding`을 사용하여 `number` 옵셔널에서 값(`10`)을 `actualNumber` 변수에 바인딩한다. 그러면 10이라는 `Int`값과 함께 `nil`인지 아닌지에대한 Bool 값도 동시에 받기 때문에 `nil`이라면 while 문이 실행 되며 실행된 while문 안에 `nil`을 할당하였기 때문에 두번째 loop에서는 condition에서 false를 return 하기 때문에 while문을 멈추게 된다.

`while`문은 특정 조건이 참인 동안 반복적으로 코드를 실행할 때 사용 되는데 다음과 같은 장단점이 있다.

**장점**
- 조건에 따라 동적으로 반복 횟수를 조절할 수 있다.
- 초기 상태에서 조건을 검사하기 때문에 조건이 처음부터 거짓인 경우 루프 내의 코드는 실행되지 않아 코스트를 낮출 수 있다.
**단점**
- 초기 조건 설정과 갱신을 직접 관리해야 하기 때문에 코드가 복잡해 질 수 있다.

### Repeat-While Statement
앞선 `while` 문이 조건을 먼저 검증 했다면, `repeat`-`while` 문은 조건 검증 전에 구문을 한번 실행시킨다.

`repeat`-`while` 구문은 다음의 형식을 가진다:
```swift
repeat {
   <#statements#>
} while <#condition#>
```

구문을 먼저 실행하고 그후에 조건을 검증하여 true이면 다시 구문을 실행하고 false이면 종료되는 방식이다. `while` 문과 비슷하지만 조건이 처음부터 false이면 바로 종료되는 `while`과는 다르게 무조건 1번 이상 실행된다는 점에서 차이가 있다.

코드 블록을 최소 한 번 실행하고, 이후에 특정 조건이 참인 동안 반복적으로 실행할 때 사용하는데 다음과 같은 장단점이 있다.

**장점**
- 반복문의 코드 블록이 최소한 한 번은 실행된다. 이러한 특성 때문에 특정 상황에서는 `while`보다 유용할 수 있다.
- 조건에 따라 동적으로 반복 횟수를 조절할 수 있다.
**단점**
- 초기 조건 설정과 조건 갱신을 직접 관리해야 한다.
- 조건 검사가 루프의 끝에서 이루어지므로, 조건이 처음부터 거짓인 경우에도 루프 내의 코드가 한 번 실행된다.

결론적으로, 각 반복문의 선택은 상황과 요구 사항에 따라 달라진다. 코드의 가독성과 의도를 명확히 표현할 수 있는 구문을 선택하는 것이 중요하다. 

### 성능비교
그렇다면 성능적인 차이는 존재할까?

```swift
 func measureTimeOfExecution(_ operation: () -> Void) -> Double {
        let startTime = CFAbsoluteTimeGetCurrent()
        operation()
        let endTime = CFAbsoluteTimeGetCurrent()
        return endTime - startTime
    }
```

기존에는 간단한 더하기 연산을 loop로 돌려 앱의 성능을 xcode instrument로 측정하려 했으나 단순한 loop 한개의 메모리 와 cpu 사용양은 정말 미미 했기 때문에 측정이 안되어 위 코드의 방식으로 단순히 loop 별 연산에 걸린 시간을 측정하는 방식으로 테스트 해보았다. 물론 정확하지 않고 실제 활용에는 여러가지 요소들이 고려되어야 하기 때문에 재미로 보기 바란다.

<p align="center"> 
<img src="https://i.imgur.com/CEveOns.png">
</p>

`for-in`의 동작 시간이 `while`, 및 `repeat-while`보다 조금 길게 나왔지만 사실 loop 자체의 기본 동작에는 큰 성능 차이가 없다. 하지만 실제 성능은 주로 루프 내부에서 수행되는 연산이나 조건의 복잡도에 따라 결정된다.

이론적으로 각 루프의 특징을 생각해보면:

1. **for-in**:
    
    - 이 루프는 `Sequence`나 `Collection` 타입의 객체를 순회하기에 최적화되어 있다.
    - 내부적으로는 이터레이터를 사용하여 각 항목을 가져오는 방식으로 작동한다.
    - `Sequence`의 길이나 복잡도에 따라 성능 차이가 있을 수 있다.
2. **while**:
    
    - `while` 루프는 주어진 조건이 참인 동안 계속 실행된다.
    - 조건의 복잡도에 따라 성능에 영향을 줄 수 있다.
    - 루프의 시작에서 매번 조건을 평가해야 한다.
3. **repeat-while**:
    
    - 이 루프는 최소한 한 번은 실행되며, 주어진 조건이 참인 동안 계속 실행된다.
    - 루프의 끝에서 조건을 평가하기 때문에, 루프의 내용이 최소한 한 번은 실행될 것이라는 것을 보장한다.

실제 성능 차이는 일반적인 사용에서는 눈에 띄지 않을 정도로 작다. 하지만 특정 상황에서는 loop의 선택이 성능에 영향을 줄 수 있습니다. 예를 들어, 매우 큰 데이터 셋을 순회해야 하는 경우 `for-in`이 다른 두 루프보다 더 효율적일 수 있다. 반면, 복잡한 조건을 평가해야 하는 경우에는 `while` 루프의 조건 평가 부담 때문에 성능에 영향을 줄 수 있다.

### 번외 사례
번외로 SwiftUI 에서 사용 되는 사례로 생각해보자.


1. **ForEach vs Range-based Loop**
    
    - `ForEach`가 성능적으로 떨어진다는 이야기가 종종 있지만 `ForEach`와 범위 기반의 `for-in` 루프는 실질적으로 거의 동일한 성능을 제공한다. `ForEach`는 내부적으로 `for-in`을 사용하여 컬렉션을 반복한다.
    - 성능의 병목점은 주로 반복문의 내부에서 수행되는 연산 및 생성되는 뷰의 복잡도에 있다.
2. **List vs VStack/HStack**
    
    - `List`와 `VStack`/`HStack` 내에서의 `ForEach` 반복문은 성능 측면에서 차이가 있을 수 있다.
        - `List`는 재사용 가능한 셀을 사용하여 성능을 최적화하므로 많은 수의 아이템을 표시할 때 유리하다.
        - 반면에 `VStack` 또는 `HStack`은 모든 아이템에 대해 뷰를 생성하므로 큰 리스트에서는 성능 저하가 발생할 수 있다.
3. **Conditional Loop with `if let` and `Optional`**
    
    - `if let` 구문의 성능 오버헤드는 무시할 만큼 작습니다. 조건부 렌더링은 복잡한 뷰 구조에서 뷰의 생성 및 렌더링 비용을 절약할 수 있기 때문에 오히려 성능 이점을 가져올 수 있다.

### 마무리
루프를 선택하는 것은 때로는 코드의 성능과 가독성, 그리고 개발의 편리성 사이에서 균형을 잡아야 하는 문제일 때가 많다. Swift에서 제공하는 `for-in`, `while`, 그리고 `repeat-while` 루프는 각각 그들만의 특징과 장점을 가지고 있다. 하지만 이 중 어느 것이 "최선"이라는 것은 없다. 대신, 개발자가 현재 작업하고 있는 문제의 도메인, 그리고 그 문제를 해결하기 위해 어떤 데이터와 연산을 다루는지에 따라 가장 적절한 루프를 선택해야 한다.

코드의 목적과 컨텍스트를 고려하여 가장 적절한 도구와 방법을 선택하는 것이 중요하다. 끝으로, 성능과 가독성, 그리고 코드의 유지보수성 사이에서 항상 균형을 찾아가며, 프로젝트와 팀의 필요에 맞는 최선의 선택을 해야 할 것이다.


