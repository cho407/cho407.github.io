---
title: "Swift 알고리즘 문제 메소드 모음"
description: "Swift 알고리즘 문제 메소드 모음"
categories: Algorithms
tags: [Swift, Algorithms]
sidebar: 
  nav: "docs"
header:
  teaser: /assets/images/algorithms.jpg
---
### 배열(Array) 관련 메소드

1. **`append(_:)`**
```swift
var numbers = [1, 2, 3]
numbers.append(4)  // [1, 2, 3, 4]
```
    
2. **`insert(_:at:)`**
```swift
var letters = ["a", "b", "d"]
letters.insert("c", at: 2)  // ["a", "b", "c", "d"]
```

3. **`remove(at:)`**
```swift
var fruits = ["Apple", "Banana", "Cherry"]
fruits.remove(at: 1)  // ["Apple", "Cherry"]
```
    
4. **`sort()` / `sorted()`**
```swift
var numbers = [3, 1, 4, 1]
numbers.sort()  // [1, 1, 3, 4]
let sortedNumbers = numbers.sorted()  // [1, 1, 3, 4]
```
    
5. **`reverse()` / `reversed()`**
```swift
var numbers = [1, 2, 3]
numbers.reverse()  // [3, 2, 1]
let reversedNumbers = numbers.reversed()  // [1, 2, 3]
```
    
6. **`filter(_:)`**
```swift
let numbers = [1, 2, 3, 4, 5]
let evenNumbers = numbers.filter { $0 % 2 == 0 }  // [2, 4]
```
    
7. **`map(_:)`**
```swift
let numbers = [1, 2, 3]
let squared = numbers.map { $0 * $0 }  // [1, 4, 9]
```
    
8. **`reduce(_:_:)`**
```swift
let numbers = [1, 2, 3, 4]
let sum = numbers.reduce(0, +)  // 10
```
    
9. **`firstIndex(of:)` / `lastIndex(of:)`**
```swift
let numbers = [1, 2, 3, 2, 1]
let firstIndex = numbers.firstIndex(of: 2)  // 1
let lastIndex = numbers.lastIndex(of: 2)  // 3
```
    
10. **`contains(_:)`**
```swift
let fruits = ["Apple", "Banana", "Cherry"]
let containsApple = fruits.contains("Apple")  // true
```
11. **`compactMap(_:)`**    
```swift
let possibleNumbers = ["1", "2", "three", "///4///", "5"]
let mapped: [Int?] = possibleNumbers.map { str in Int(str) } // [1, 2, nil, nil, 5]
let compactMapped: [Int] = possibleNumbers.compactMap { str in Int(str) } // [1, 2, 5]
let n: Int = 1234
let numArray: [Int] = String(n).compactMap { Int(String($0)) } // [1, 2, 3, 4]
```

### 문자열(String) 관련 메소드

1. **`split(separator:)`**
```swift
let sentence = "Hello, World"
let words = sentence.split(separator: " ")  // ["Hello,", "World"]
```
    
2. **`replacingOccurrences(of:with:)`**
```swift
let originalString = "Hello, World"
let replacedString = originalString.replacingOccurrences(of: "World", with: "Swift")  // "Hello, Swift"
```
    
3. **`substring(with:)`**
```swift
let str = "Hello, World"
let index = str.index(str.startIndex, offsetBy: 7)
let substring = str[index...]  // "World"
// swift 5
let str = "Hello, Swift"
// 첫 번째 문자부터 특정 위치까지 추출
let start = str.startIndex
let end = str.index(start, offsetBy: 4)
let substring1 = str[start...end]  // "Hello"
// 특정 위치부터 끝까지 추출
let start2 = str.index(str.startIndex, offsetBy: 7)
let substring2 = str[start2...]  // "Swift"
// 특정 범위 내 추출
let range = start2...end
let substring3 = str[range]  // "Swif"
```
    
4. **`trimmingCharacters(in:)`**
```swift
let rawString = "  Hello, Swift  "
let trimmedString = rawString.trimmingCharacters(in: .whitespaces)  // "Hello, Swift"
```
    
5. **`lowercased()` / `uppercased()`**
```swift
let originalString = "Swift"
let lowercased = originalString.lowercased()  // "swift"
let uppercased = originalString.uppercased()  // "SWIFT"
```
    
6. **`hasPrefix(_:)` / `hasSuffix(_:)`**
```swift
let string = "Hello, Swift"
let hasPrefix = string.hasPrefix("Hello")  // true
let hasSuffix = string.hasSuffix("Swift")  // true
```

7. **.reversed()**
```swift
let string = "hello"
let reversed = String(string.reversed()) // olleh
```

8. **String -> String.element 배열**
```swift
let hello: String = "hello"
let helloArr = Array(hello) // ["h", "e", "l", "l", "o"]
let stringArr = hello.map(String.init) // ["h", "e", "l", "l", "o"]
// 출력 형태는 같아보이지만 [hello] 타입과 [String] 타입으로 차이남
```

### 세트(Set) 관련 메소드

1. **`insert(_:)`**
```swift
var numbers: Set = [1, 2, 3]
numbers.insert(4)  // [1, 2, 3, 4]
```
    
2. **`remove(_:)`**
```swift
var numbers: Set = [1, 2, 3, 4]
numbers.remove(3)  // [1, 2, 4]
```
    
3. **`contains(_:)`**
```swift
let numbers: Set = [1, 2, 3]
let containsTwo = numbers.contains(2)  // true
```
    
4. **`union(_:)`**
```swift
let set1: Set = [1, 2, 3]
let set2: Set = [4, 5, 6]
let unionSet = set1.union(set2)  // [1, 2, 3, 4, 5, 6]
```
    
5. **`intersection(_:)`**
```swift
let set1: Set = [1, 2, 3]
let set2: Set = [2, 3, 4]
let intersectionSet = set1.intersection(set2)  // [2, 3]
```
    
6. **`subtracting(_:)`**
```swift
let set1: Set = [1, 2, 3]
let set2: Set = [2, 3, 4]
let subtractingSet = set1.subtracting(set2)  // [1]
```
    
7. **`isSubset(of:)`**
```swift
let set1: Set = [1, 2]
let set2: Set = [1, 2, 3, 4]
let isSubset = set1.isSubset(of: set2)  // true
```

8. **`allSatisfy(_:)`**
```
let numbers = [28, 32, 64, 90]
let passed = numbers.allSatisfy { $0 >= 28 }
```

    

### 딕셔너리(Dictionary) 관련 메소드

1. **`updateValue(_:forKey:)`**
```swift
var capitals = ["KR": "Seoul", "JP": "Tokyo"]
capitals.updateValue("Washington D.C.", forKey: "US")  // ["KR": "Seoul", "JP": "Tokyo", "US": "Washington D.C."]
```
    
2. **`removeValue(forKey:)`**
```swift
var capitals = ["KR": "Seoul", "JP": "Tokyo"]
capitals.removeValue(forKey: "JP")  // ["KR": "Seoul"]
```
    
3. **`keys` / `values`**
```swift
let capitals = ["KR": "Seoul", "JP": "Tokyo"]
let keys = capitals.keys  // ["KR", "JP"]
let values = capitals.values  // ["Seoul", "Tokyo"]
```
    
4. **`filter(_:)`**
```swift
let numbers = [1: "One", 2: "Two", 3: "Three"]
let filtered = numbers.filter { $0.key % 2 == 0 }  // [2: "Two"]
```
    
5. **`mapValues(_:)`**
```swift
let numbers = [1: "One", 2: "Two", 3: "Three"]
let mappedValues = numbers.mapValues { $0.uppercased() }  // [1: "ONE", 2: "TWO", 3: "THREE"]
```
    

### 알고리즘 관련 기타 유용한 함수 및 연산자

1. **`min(_:_:)` / `max(_:_:)`**
```swift
let minValue = min(3, 5)  // 3
let maxValue = max(3, 5)  // 5
```
    
2. **`abs(_:)`**
```swift
let absoluteValue = abs(-5)  // 5
```
    
3. **`pow(_:_:)`**
```swift
let powerValue = pow(2, 3)  // 8.0
```
    
4. **`..<` / `...`**
```swift
let range = 1..<5  // 1, 2, 3, 4
let closedRange = 1...5  // 1, 2, 3, 4, 5
```
    
5. **`zip(_:_:)`**
```swift
let numbers = [1, 2, 3]
let letters = ["A", "B", "C"]
let zipped = zip(numbers, letters)  // [(1, "A"), (2, "B"), (3, "C")]
```
