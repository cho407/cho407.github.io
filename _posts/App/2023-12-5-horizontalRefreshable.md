---
title: "[SwiftUI] Custom Horizontal Rereshable 구현"
description: "[SwiftUI] Custom Horizontal Rereshable 구현"
categories: App
tags: [개발, HIG, 앱, iOS, swiftUI]
sidebar: 
  nav: "docs"
header:
  teaser: https://i.imgur.com/6RPaFu5.png
---
스택오버플로우를 구경하던 중 가로로 refresh action을 구현하고싶어서 고민중인 사람의 질문을 발견했다. 아쉽게도 apple에서 제공하는 메소드 중에는 가로로 refresh 하는 기능은 없기때문에 간단한 dragGesture와 animation을 이용해서 다음과 같이 구현해보았다.

```swift
extension View {
    func horizontalRefreshable(threshold: CGFloat = 100, isRefreshing: Binding<Bool>, action: @escaping () -> Void) -> some View {
        self.modifier(PullToRefreshModifier(threshold: threshold, isRefreshing: isRefreshing, action: action))
    }
}

struct PullToRefreshModifier: ViewModifier {
    let threshold: CGFloat
    @Binding var isRefreshing: Bool
    let action: () -> Void
    @State private var dragOffset = CGSize.zero

    func body(content: Content) -> some View {
        ZStack {
            if isRefreshing {
                ProgressView()
                    .progressViewStyle(CircularProgressViewStyle())
                    .position(x: threshold / 2, y: UIScreen.main.bounds.height / 2)
            }

            content
                .offset(x: isRefreshing ? threshold : dragOffset.width, y: 0)
        }
        .gesture(
            DragGesture()
                .onChanged { gesture in
                    if gesture.translation.width > 0 && abs(gesture.translation.height) < 50 {
                        dragOffset = gesture.translation
                    }
                }
                .onEnded { _ in
                    if dragOffset.width > threshold {
                        isRefreshing = true
                        DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
                            isRefreshing = false
                            dragOffset = .zero
                            action()
                        }
                    } else {
                        dragOffset = .zero
                    }
                }
        )
        .animation(.easeInOut, value: dragOffset)
        .animation(.easeInOut, value: isRefreshing)
    }
}

struct ContentView: View {
    @State private var isRefreshing = false

    var body: some View {
        VStack {
            Text("Your View")
        }
        .frame(maxWidth: .infinity, maxHeight: .infinity)
        .background(Color.red)
        .horizontalRefreshable(isRefreshing: $isRefreshing) {
            print("Refresh")
        }
    }
}
```

ViewModifier로 구현하여 기존 `.refreshable`과 비슷한 현태로 사용할 수 있게 구현하였다.
![](https://i.imgur.com/wtaQ2z8.png)

![](https://i.imgur.com/6RPaFu5.png)

그 결과 위와 같이 작동하는데 나름 그럴싸한(?) 성능구현이 되었지만 모든 상황에서 문제 없이 작동 하는지 그리고 조금더 디테일한 사용에 있어서의 interaction은 손봐야 할 필요가 있어보인다.

