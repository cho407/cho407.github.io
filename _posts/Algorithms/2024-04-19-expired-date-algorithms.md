---
title: "[Swift] 개인정보 수집 유효기간 알고리즘(Feat. dateFormatter)"
description: "[Swift] 개인정보 수집 유효기간 알고리즘(Feat. dateFormatter)"
categories: Algorithms
tags: [Swift, Algorithms]
sidebar: 
  nav: "docs"
header:
  teaser: /assets/images/algorithms.jpg
---
### 문제

고객의 약관 동의를 얻어서 수집된 1~`n`번으로 분류되는 개인정보 `n`개가 있습니다. 약관 종류는 여러 가지 있으며 각 약관 마다 개인정보 보관 유효기간이 정해져 있습니다. 당신은 각 개인정보가 어떤 약관으로 수집 됐는지 알고 있습니다. 수집된 개인정보는 유효기간 전까지만 보관 가능하며, 유효기간이 지났다면 반드시 파기해야 합니다.

예를 들어, A라는 약관의 유효기간이 12 달이고, 2021년 1월 5일에 수집된 개인정보가 A약관으로 수집되었다면 해당 개인정보는 2022년 1월 4일까지 보관 가능하며 2022년 1월 5일부터 파기해야 할 개인정보입니다.  
당신은 오늘 날짜로 파기해야 할 개인정보 번호들을 구하려 합니다.

**모든 달은 28일까지 있다고 가정합니다.**

오늘 날짜를 의미하는 문자열 `today`, 약관의 유효기간을 담은 1차원 문자열 배열 `terms`와 수집된 개인정보의 정보를 담은 1차원 문자열 배열 `privacies`가 매개변수로 주어집니다. 이때 파기해야 할 개인정보의 번호를 오름차순으로 1차원 정수 배열에 담아 return 하도록 solution 함수를 완성해 주세요.


![](https://i.imgur.com/zFvhMVz.png)

### 풀이

사실 알고리즘 자체는 단순하다. 입력 자료들을 원하는 형태로 가공하고 약관에 맞게 expire date를 계산해서 날짜가 지난 약관들을 골라내는 방식이다. 하지만 dateFormatter의 활용을 하지 못하면 곤란해지는 문제이기에 한번더 정리하고자 이 글을 작성한다. 우선 전체적인 풀이 부터 정리하자면 다음과 같다. 

```swift
import Foundation

func solution(_ today:String, _ terms:[String], _ privacies:[String]) -> [Int] {
    let dateFormatter = DateFormatter()
    dateFormatter.dateFormat = "yyyy.MM.dd"
    guard let todayDate = dateFormatter.date(from: today) else { return[] }
    
    var termsDic: [String: Int] = [:]
    var result: [Int] = []
    for term in terms {
        let parts = term.split(separator: " ").map(String.init)
        termsDic[parts[0]] = Int(parts[1]) ?? 0
    }
    
    for (index, privacy) in privacies.enumerated() {
        let parts = privacy.split(separator: " ").map(String.init)
        let dateStr = parts[0]
        let termType = parts[1]
        
        if let startDate = dateFormatter.date(from: dateStr), let monthsToAdd = termsDic[termType] {
            var dateComponent = DateComponents()
            dateComponent.month = monthsToAdd
            let endDate = Calendar.current.date(byAdding: dateComponent, to: startDate)!
            
            if endDate <= todayDate {
                result.append(index + 1) 
            }
        }
    }
        
    return result
}
```

### DateFormatter

나의 풀이를 자세히 들여다 보면 이부분이 나온다.
```swift
    let dateFormatter = DateFormatter()
    dateFormatter.dateFormat = "yyyy.MM.dd"
    guard let todayDate = dateFormatter.date(from: today) else { return[] }
```

`DateFormatter()`의 component 를 사용하기 위해 다음과 같이 할당한 후 내가 사용해야 하는 날짜 자료형에 대한 `dateFormat`을 할당한 후에 `guard let`을 이용하여 언래핑 하여 today 매개변수로 받아주는 `STring`타입의 날짜를 `dateFormat`으로 변환시킨 과정이다.

그 이후에 아래 다음과 같은 과정이 나온다.

```swift
        if let startDate = dateFormatter.date(from: dateStr), let monthsToAdd = termsDic[termType] {
            var dateComponent = DateComponents()
            dateComponent.month = monthsToAdd
            let endDate = Calendar.current.date(byAdding: dateComponent, to: startDate)!
            
            if endDate <= todayDate {
                result.append(index + 1) 
            }
        }
```

`.split`을 통해 추출 해낸 `dateStr` 을 dateFormatter로 변환시키고 시작 날짜를 정의한다. 그 후에 더할 개월 수를 Int타입으로 정의한다.
```swift
var dateComponent = DateComponents()
            dateComponent.month = monthsToAdd
            let endDate = Calendar.current.date(byAdding: dateComponent, to: startDate)!
```
그런다음 위와 같은 과정으로 `dateComponent.month`에 `monthsToAdd` 를 할당하고 startDate에서 개월수를 더하여 endDate를 추출하여 비교하는것이다.

간단한 사용에 대한 내용만 정리했는데 조금더 자세한 내용이 필요하다만 아래 공식문서를 참고 바란다.

공식문서: [Dates and Times](https://developer.apple.com/documentation/foundation/dates_and_times)
