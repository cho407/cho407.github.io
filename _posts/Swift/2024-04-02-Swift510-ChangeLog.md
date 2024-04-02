---
title: "Swift 5.10 ver ChangeLoge 번역"
description: "Swift 5.10 ver ChangeLoge 번역"
categories: Swift
tags: [개발, Swift, ChangLog]
sidebar: 
  nav: "docs"
header:
  teaser: https://developer.apple.com/swift/images/swift-og.png
---
Swift 5.10 의 ChangeLog 를 한글로 번역한 것입니다. 다소 오역이 있을 수 있으니 발견하면 댓글로 남겨주세요.

## Swift 5.10

* Swift 5.10은 엄격한 동시성 검사를 완전히 수행하면서 알려진 모든 정적 데이터 경쟁 상태의 안전성 문제를 해결합니다.
`-strict-concurrency=complete`를 사용하여 코드를 작성할 때, Swift 5.10은 `nonisolated(unsafe)` 또는 `@unchecked Sendable`과 같은 명시적인 안전하지 않은 옵트아웃을 사용하지 않는 한 컴파일 시간에 모든 데이터 경쟁 가능성을 진단합니다.
예를 들어, Swift 5.9에서 다음 코드는 `@MainActor`에 의해 분리된 초기화자가 액터 외부에서 평가되어 런타임에 충돌하지만, `-strict-concurrency=complete` 하에서는 진단되지 않았습니다:

  ```swift
  @MainActor
  class MyModel {
    init() {
      MainActor.assertIsolated()
    }

    static let shared = MyModel()
  }

  func useShared() async {
    let model = MyModel.shared
  }

  await useShared()
  ```

  위 코드는 `@MainActor`에 의해 분리된 정적 변수가 첫 번째 접근 시 `@MainActor`에 의해 분리된 초기값을 평가하는데, `nonisolated` 컨텍스트에서 동기적으로 접근되기 때문에 데이터 경쟁을 허용합니다. Swift 5.10에서는 `-strict-concurrency=complete`로 코드를 컴파일하면 접근이 비동기적으로 이루어져야 한다는 경고를 생성합니다:

  ```
  warning: expression is 'async' but is not marked with 'await'
    let model = MyModel.shared
                ^~~~~~~~~~~~~~
                await
  ```

  Swift 5.10은 `Sendable` 및 액터 분리 검사에서 여러 다른 버그를 수정하여 엄격한 동시성 검사의 보장을 강화했습니다.
Swift 5.10의 완전한 동시성 모델은 보수적입니다. Swift 6 이전에 엄격한 동시성 검사의 사용성을 개선하기 위한 여러 Swift Evolution 제안이 활발히 개발 중입니다.

* [SE-0412][]:
전역 및 정적 변수는 프로그램의 어떤 컨텍스트에서도 접근할 수 있는 메모리를 제공하기 때문에 데이터 경쟁에 취약합니다. Swift 5.10의 엄격한 동시성 검사는 전역 및 정적 변수가 데이터 경쟁을 방지하기 위해 다음 중 하나가 되도록 요구합니다:

  1. 전역 액터에 분리되거나,
  2. 불변이며 `Sendable` 타입입니다.

  예를 들어:

  ```swift
  var mutableGlobal = 1
  // warning: var 'mutableGlobal' is not concurrency-safe because it is non-isolated global shared mutable state
  // (unless it is top-level code which implicitly isolates to @MainActor)

  @MainActor func mutateGlobalFromMain() {
    mutableGlobal += 1
  }

  nonisolated func mutateGlobalFromNonisolated() async {
    mutableGlobal += 10
  }

  struct S {
    static let immutableSendable = 10
    // okay; 'immutableSendable' is safe to access concurrently because it's immutable and 'Int' is 'Sendable'
  }
  ```

  수동 동기화가 제공될 때 데이터 분리 위반을 억제하기 위해 전역 또는 정적 변수에 `nonisolated(unsafe)` 수정자를 사용할 수 있습니다:

  ```swift
  // This global is only set in one part of the program
  nonisolated(unsafe) var global: String!
  ```

  `nonisolated(unsafe)`는 저장된 속성 및 지역 변수를 포함한 모든 형태의 저장소에 사용될 수 있으며, 많은 사용 사례에서 `@unchecked Sendable` 래퍼 타입의 필요성을 없애는 `Sendable` 검사에 대한 더 세밀한 옵트아웃을 제공합니다:

  ```swift
  import Dispatch

  // 'MutableData' is not 'Sendable'
  class MutableData { ... } 

  final class MyModel: Sendable {
    private let queue = DispatchQueue(...)
    // 'protectedState' is manually isolated by 'queue'
    nonisolated(unsafe) private var protectedState: MutableData
  }
  ```

  데이터 분리를 달성하기 위한 동기화 메커니즘의 올바른 구현 없이는 독점성 집행 또는 Thread Sanitizer와 같은 도구에서 동적 런타임 분석이 여전히 실패를 식별할 수 있습니다.

* [SE-0411][]:
Swift 5.10은 이전에 액터 외부에서 동기적으로 평가될 수 있었던 분리된 기본 저장된 속성 값에 대한 데이터 경쟁 안전성 구멍을 해결합니다. 예를 들어, 다음 코드는 Swift 5.9에서 `-strict-concurrency=complete` 하에서 경고 없이 컴파일되지만, `MainActor.assertIsolated()` 호출 시 런타임에 충돌합니다:

  ```swift
  @MainActor func requiresMainActor() -> Int {
    MainActor.assertIsolated()
    return 0
  }

  @MainActor struct S {
    var x = requiresMainActor()
    var y: Int
  }

  nonisolated func call() async {
    let s = await S(y: 10)
  }

  await call()
  ```

  이는 `requiresMainActor()`가 S의 멤버별 초기화자에 기본 인수로 사용되지만, 기본 인수는 항상 호출자에서 평가되기 때문입니다. 이 경우, 호출자는 일반 실행자에서 실행되므로 기본 인수 평가가 충돌합니다.
Swift 5.10에서 `-strict-concurrency=complete` 하에서 기본 인수 값은 포함 함수 또는 저장된 속성과 동일한 분리를 안전하게 공유할 수 있습니다. 위 코드는 여전히 유효하지만, 분리된 기본 인수는 호출자의 분리 도메인에서 평가되도록 보장됩니다.

### 원본
## Swift 5.10

* Swift 5.10 closes all known static data-race safety holes in complete strict
concurrency checking.

  When writing code against `-strict-concurrency=complete`, Swift 5.10 will
  diagnose all potential for data races at compile time unless an explicit
  unsafe opt out, such as `nonisolated(unsafe)` or `@unchecked Sendable`, is
  used.

  For example, in Swift 5.9, the following code crashes at runtime due to a
  `@MainActor`-isolated initializer being evaluated outside the actor, but it
  was not diagnosed under `-strict-concurrency=complete`:

  ```swift
  @MainActor
  class MyModel {
    init() {
      MainActor.assertIsolated()
    }

    static let shared = MyModel()
  }

  func useShared() async {
    let model = MyModel.shared
  }

  await useShared()
  ```

  The above code admits data races because a `@MainActor`-isolated static
  variable, which evaluates a `@MainActor`-isolated initial value upon first
  access, is accessed synchronously from a `nonisolated` context. In Swift
  5.10, compiling the code with `-strict-concurrency=complete` produces a
  warning that the access must be done asynchronously:

  ```
  warning: expression is 'async' but is not marked with 'await'
    let model = MyModel.shared
                ^~~~~~~~~~~~~~
                await
  ```

  Swift 5.10 fixed numerous other bugs in `Sendable` and actor isolation
  checking to strengthen the guarantees of complete concurrency checking.

  Note that the complete concurrency model in Swift 5.10 is conservative.
  Several Swift Evolution proposals are in active development to improve the
  usability of strict concurrency checking ahead of Swift 6.

* [SE-0412][]:

  Global and static variables are prone to data races because they provide memory that can be accessed from any program context.  Strict concurrency checking in Swift 5.10 prevents data races on global and static variables by requiring them to be either:

    1. isolated to a global actor, or
    2. immutable and of `Sendable` type.

  For example:

  ```swift
  var mutableGlobal = 1
  // warning: var 'mutableGlobal' is not concurrency-safe because it is non-isolated global shared mutable state
  // (unless it is top-level code which implicitly isolates to @MainActor)

  @MainActor func mutateGlobalFromMain() {
    mutableGlobal += 1
  }

  nonisolated func mutateGlobalFromNonisolated() async {
    mutableGlobal += 10
  }

  struct S {
    static let immutableSendable = 10
    // okay; 'immutableSendable' is safe to access concurrently because it's immutable and 'Int' is 'Sendable'
  }
  ```

  A new `nonisolated(unsafe)` modifier can be used to annotate a global or static variable to suppress data isolation violations when manual synchronization is provided:

  ```swift
  // This global is only set in one part of the program
  nonisolated(unsafe) var global: String!
  ```

  `nonisolated(unsafe)` can be used on any form of storage, including stored properties and local variables, as a more granular opt out for `Sendable` checking, eliminating the need for `@unchecked Sendable` wrapper types in many use cases:

  ```swift
  import Dispatch

  // 'MutableData' is not 'Sendable'
  class MutableData { ... } 

  final class MyModel: Sendable {
    private let queue = DispatchQueue(...)
    // 'protectedState' is manually isolated by 'queue'
    nonisolated(unsafe) private var protectedState: MutableData
  }
  ```

  Note that without correct implementation of a synchronization mechanism to achieve data isolation, dynamic run-time analysis from exclusivity enforcement or tools such as the Thread Sanitizer could still identify failures.

* [SE-0411][]:

  Swift 5.10 closes a data-race safety hole that previously permitted isolated
  default stored property values to be synchronously evaluated from outside the
  actor. For example, the following code compiles warning-free under
  `-strict-concurrency=complete` in Swift 5.9, but it will crash at runtime at
  the call to `MainActor.assertIsolated()`:

  ```swift
  @MainActor func requiresMainActor() -> Int {
    MainActor.assertIsolated()
    return 0
  }

  @MainActor struct S {
    var x = requiresMainActor()
    var y: Int
  }

  nonisolated func call() async {
    let s = await S(y: 10)
  }

  await call()
  ```

  This happens because `requiresMainActor()` is used as a default argument to
  the member-wise initializer of `S`, but default arguments are always
  evaluated in the caller. In this case, the caller runs on the generic
  executor, so the default argument evaluation crashes.

  Under `-strict-concurrency=complete` in Swift 5.10, default argument values
  can safely share the same isolation as the enclosing function or stored
  property. The above code is still valid, but the isolated default argument is
  guaranteed to be evaluated in the callee's isolation domain.

  [참고 링크(swift.org)](https://www.swift.org/blog/swift-5.10-released/)
