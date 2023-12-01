---
title: "[SwiftUI] Dynamic Type Sizes에 맞춘 폰트 사이즈 대응"
description: "[SwiftUI] Dynamic Type Sizes에 맞춘 폰트 사이즈 대응"
categories: App
tags: [개발, HIG, 앱, iOS, swiftUI]
sidebar: 
  nav: "docs"
header:
  teaser: https://i.imgur.com/JuezYFp.png
---
## 서론
HIG에 따르면 디바이스 자체의 글자 크기를 바꾸면 앱의 폰트 사이즈도 동적으로 바뀌게 해야한다. 그래서 우리는 `.body`와 같은 시스템 폰트 사이즈를 활용한다. 하지만 커스텀 폰트 사이즈를 다이나믹하게 바뀌도록 하고싶을때는 어떤 방법을 사용해야할까?

## Font 구조 탐구
`.body`를 살펴보면 아래와 같이 `Font` extension에 포함되어 있다.

![](https://i.imgur.com/JuezYFp.png)

## 코드 구성과 문제점
따라서 우리의 custom font style 도 font extension 에 추가하면 된다는 말이다. 그래서 코드를 구성해보면 다음과 같다.

```swift
import SwiftUI

extension Font {
    static var customBody: Font {
        let defaultBodySize = UIFont.preferredFont(forTextStyle: .body).pointSize
        let preferredSize = UIApplication.shared.preferredContentSizeCategory

        switch preferredSize {
        case .extraSmall, .small, .medium:
            return .system(size: defaultBodySize)
        case .large:
            return .system(size: defaultBodySize)
        case .extraLarge:
            return .system(size: defaultBodySize + 1)
        case .extraExtraLarge:
            return .system(size: defaultBodySize + 2)
        case .extraExtraExtraLarge:
            return .system(size: defaultBodySize + 3)
        default:
            return .body
        }
    }
}
```

.body의 pointSize를 받아오고 그것에 맞춰 상대적인 사이즈를 입력해서 적용하는 방식이다. 이렇게 하면 아래와 같이 뷰에서 이질감 없이 custom font style을 적용시킬 수 있다.

```swift
        Text("Example Text")
            .font(.customBody)
```

하지만 이 방식에는 한가지 문제점이 있다. 앱을 실행한 상태에서 디바이스의 설정을 변경하면 앱에서는 바로 적용이 되지 않고 앱을 종료했다가 다시 켜야 적용이 된다는 점이다. 그 이유는 디바이스의 설정된 컨텐츠 사이즈 값을 컴파일 하는 시점에서 받아오기 때문인데 그 과정에서 사용하는 `preferredContentSizeCategory`가 앱 전체에 대한 컨텐츠 사이즈를 받아오기 때문이다. 별거 아닌 문제라고 생각할수도 있지만 찝찝하니까 이것을 해결해보자.

## 해결책과 완성된 코드
검색한 결과 `@Environment(\.sizeCategory) var sizeCategory`를 이용하면 현재 view의 컨텐츠 사이즈를 실시간으로 받아올 수 있었다. 조금 더 swiftUI 스러운(?) 방법이었다. 따라서 sizeCategory를 전달받을 수 있게 함수 형태로 수정하여 다음과 같이 코드를 작성하였다. 

```swift
extension Font {
    static func customBody(for sizeCategory: ContentSizeCategory) -> Font {
        let defaultBodySize = UIFont.preferredFont(forTextStyle: .body).pointSize

        switch sizeCategory {
        case .extraSmall, .small, .medium:
            return .system(size: defaultBodySize)
        case .large:
            return .system(size: defaultBodySize)
        case .extraLarge:
            return .system(size: defaultBodySize + 1)
        case .extraExtraLarge:
            return .system(size: defaultBodySize + 2)
        case .extraExtraExtraLarge:
            return .system(size: defaultBodySize + 3)
        default:
            return .body
        }
    }
}
struct ContentView: View {
    @Environment(\.sizeCategory) var sizeCategory

    var body: some View {
        VStack{
            Text("Default body size")
                .font(.body)
            Text("Custom body size")
                .font(.customBody(for: sizeCategory))
        }
    }
}
```

이런식으로 적용하면 HIG 기준에 부합하며 폰트 사이즈에 대한 디자인적 요구사항을 충족시킬 수 있다.
