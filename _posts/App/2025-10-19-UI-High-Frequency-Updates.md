---
title: "SwiftUI에서 고빈도 UI 업데이트가 메뉴바를 깜빡이게 만든 이유와 해결 방법"
description: "SwiftUI에서 고빈도 UI 업데이트가 메뉴바를 깜빡이게 만든 이유와 해결 방법"
categories: App
tags: [개발, 앱, SwiftUI, Performance, Architecture]
sidebar: 
  nav: "docs"
header:
  teaser: https://developer.apple.com/assets/elements/icons/swiftui/swiftui-256x256_2x.png
---

CaptionMate 앱 개발 중 이상한 버그를 발견했다. 오디오를 재생하는 동안 상단 메뉴바의 항목들을 클릭하면 메뉴가 계속 깜빡이면서 선택할 수 없는 현상이 발생했다. 이상한 점은 `Button`은 정상적으로 작동하는데, `Menu`만 선택이 불가능하다는 것이었다.

이 글에서는 문제의 원인을 파악하고 해결하는 과정, 그리고 그 과정에서 고민했던 아키텍처 설계에 대해 공유하고자 한다.

## 문제 상황

### 증상

- 오디오 재생 중 메뉴바의 `Language`, `Theme` 등의 `Menu` 항목 선택 불가
- 메뉴를 클릭하면 드롭다운이 열리지만 즉시 닫힘 (깜빡임)
- 오디오 재생이 끝나면 정상 작동
- 같은 조건에서 `Button` 항목은 정상 작동

### 코드 구조 (문제 발생 당시)

```swift
@MainActor
class ContentViewModel: ObservableObject {
    @Published var currentPlayerTime: Double = 0.0  // 30fps로 업데이트
    @Published var audioPlayer: AVAudioPlayer?
    @Published var modelManagementState = ModelManagementState()
    @Published var audioState = AudioState()
    // ... 수많은 다른 @Published 속성들
    
    private func setupPlaybackTimeUpdater() {
        playbackTimerCancellable = Timer.publish(every: 0.033, on: .main, in: .common)
            .autoconnect()
            .sink { [weak self] _ in
                guard let self = self, let player = self.audioPlayer else { return }
                
                if self.audioState.isPlaying {
                    self.currentPlayerTime = player.currentTime  // 30fps 업데이트!
                }
            }
    }
}
```

```swift
@main
struct CaptionMateApp: App {
    @StateObject var contentViewModel: ContentViewModel = .init()
    
    var body: some Scene {
        WindowGroup {
            ContentView(viewModel: contentViewModel)
        }
        .commands {
            CommandMenu("Settings") {
                Menu("Language") {  // 이 부분이 깜빡임!
                    Button("English") {
                        contentViewModel.changeAppLanguage(to: "en")
                    }
                    Button("한국어") {
                        contentViewModel.changeAppLanguage(to: "ko")
                    }
                }
            }
        }
    }
}
```

## 원인 분석

### 1. ObservableObject의 동작 방식

SwiftUI에서 `ObservableObject`의 `@Published` 속성이 변경되면:

```swift
// 내부적으로 이렇게 동작
@Published var currentPlayerTime: Double = 0.0 {
    willSet {
        objectWillChange.send()  // 모든 @Published가 이것을 호출!
    }
}
```

`objectWillChange`가 발생하면:
- 해당 `ObservableObject`를 관찰하는 **모든 View**가 재렌더링됨
- **메뉴바를 포함한** 모든 UI 컴포넌트가 영향을 받음

### 2. 30fps 업데이트의 영향

- 초당 30번 `currentPlayerTime` 업데이트
- 초당 30번 `ContentViewModel.objectWillChange` 발생
- 초당 30번 전체 View 계층 구조 재평가
- **메뉴바도 초당 30번 재렌더링**

### 3. Menu vs Button의 차이

#### Button의 경우
```swift
Button("English") {
    contentViewModel.changeAppLanguage(to: "en")
}
```
- 상태를 유지하지 않음 (stateless)
- 클릭 → 즉시 액션 실행 → 완료
- 재렌더링되어도 영향 없음

#### Menu의 경우
```swift
Menu("Language") {
    Button("English") { /* ... */ }
    Button("한국어") { /* ... */ }
}
```
- 드롭다운이 열린 **상태를 유지**해야 함 (stateful)
- 사용자가 선택하는 동안 열려있어야 함
- **재렌더링이 발생하면 내부 상태가 리셋되어 닫힘**

### 4. 핵심 문제

```
currentPlayerTime 변경 (30fps)
    ↓
ContentViewModel.objectWillChange 발생
    ↓
모든 View 재렌더링 (메뉴바 포함)
    ↓
Menu의 "열림" 상태 리셋
    ↓
드롭다운 닫힘 (깜빡임)
```

## 해결 과정

### 시도 1: 업데이트 빈도 줄이기 ❌

```swift
// 30fps → 10fps로 변경
Timer.publish(every: 0.1, on: .main, in: .common)
```

**결과**: 깜빡임이 느려졌을 뿐, 문제는 여전히 존재했다. 오히려 UI가 버벅거렸다.

### 시도 2: RunLoop Mode 변경 + Throttle ❌

```swift
Timer.publish(every: 0.033, on: .main, in: .tracking)  // .common → .tracking
    .autoconnect()
    .throttle(for: .milliseconds(50), scheduler: RunLoop.main, latest: true)
```

**아이디어**: `.tracking` 모드는 사용자 인터랙션 중에는 타이머를 일시정지시킨다.

**결과**: 깜빡임은 줄었지만, 사용자가 메뉴를 조작하는 동안 오디오 재생 시간이 멈춰보이는 부작용이 발생했다.

### 시도 3: 상태 분리 (최종 해결) ✅

**핵심 아이디어**: 자주 변경되는 상태를 별도의 `ObservableObject`로 분리하여, 전체 ViewModel의 재렌더링을 방지한다.

#### 1단계: AudioPlaybackState 분리

```swift
// 새로운 ObservableObject 생성
@MainActor
class AudioPlaybackState: ObservableObject {
    @Published var currentPlayerTime: Double = 0.0
}

// ContentViewModel에서 분리
@MainActor
class ContentViewModel: ObservableObject {
    // @Published var currentPlayerTime: Double = 0.0  ← 제거
    let audioPlaybackState = AudioPlaybackState()  // let으로 선언!
    
    @Published var audioPlayer: AVAudioPlayer?
    // ... 다른 속성들
    
    private func setupPlaybackTimeUpdater() {
        playbackTimerCancellable = Timer.publish(every: 0.033, on: .main, in: .common)
            .autoconnect()
            .sink { [weak self] _ in
                guard let self = self, let player = self.audioPlayer else { return }
                
                if self.audioState.isPlaying {
                    // ContentViewModel이 아닌 AudioPlaybackState 업데이트
                    self.audioPlaybackState.currentPlayerTime = player.currentTime
                }
            }
    }
}
```

**중요**: `audioPlaybackState`를 `let`으로 선언한 이유
- `@Published let audioPlaybackState`가 아니다!
- `audioPlaybackState` 자체는 변경되지 않으므로 `ContentViewModel.objectWillChange` 발생하지 않음
- `audioPlaybackState.currentPlayerTime`이 변경되면 `AudioPlaybackState.objectWillChange`만 발생

#### 2단계: View에서 명시적으로 관찰

```swift
struct AudioControlView: View {
    @ObservedObject var contentViewModel: ContentViewModel
    @ObservedObject var playbackState: AudioPlaybackState  // 추가
    
    init(contentViewModel: ContentViewModel) {
        self.contentViewModel = contentViewModel
        self.playbackState = contentViewModel.audioPlaybackState  // 초기화
    }
    
    var body: some View {
        VStack {
            // playbackState를 직접 사용
            Text("**\(formatTime(playbackState.currentPlayerTime))** / \(formatTime(contentViewModel.audioState.totalDuration))")
            
            Slider(
                value: Binding(
                    get: { playbackState.currentPlayerTime },
                    set: { contentViewModel.seekToPosition($0) }
                ),
                in: 0...contentViewModel.audioState.totalDuration
            )
        }
    }
}
```

#### 3단계: 동일한 방식으로 ModelManagementState 분리

```swift
// struct에서 class로 변경
@MainActor
class ModelManagementState: ObservableObject {
    @Published var modelStorage: String = "huggingface/models/argmaxinc/whisperkit-coreml"
    @Published var modelState: ModelState = .unloaded
    @Published var loadingProgressValue: Float = 0.0
    @Published var downloadProgress: [String: Float] = [:]
    // ... 다른 속성들
}

// ContentViewModel에서
@MainActor
class ContentViewModel: ObservableObject {
    // @Published var modelManagementState = ModelManagementState()  ← 제거
    let modelManagementState = ModelManagementState()  // let으로 변경
}
```

```swift
// ModelSelectorView에서 명시적 관찰
struct ModelSelectorView: View {
    @ObservedObject var viewModel: ContentViewModel
    @ObservedObject var modelState: ModelManagementState  // 추가
    
    init(viewModel: ContentViewModel) {
        self.viewModel = viewModel
        self.modelState = viewModel.modelManagementState
    }
    
    var body: some View {
        VStack {
            if modelState.modelState == .loading {
                ProgressView(value: modelState.loadingProgressValue)
            }
            // ...
        }
    }
}
```

## 해결 원리

### 변경 전: 폭포수 효과 (Cascading Updates)

```
AudioPlaybackState.currentPlayerTime 변경 (30fps)
    ↓
ContentViewModel.objectWillChange 발생
    ↓
ContentView 재렌더링
    ↓
모든 하위 View 재렌더링 (메뉴바 포함)
    ↓
AudioControlView 재렌더링
    ↓
WaveFormView 재렌더링
    ↓
... (모든 View들)
```

### 변경 후: 선택적 업데이트 (Selective Updates)

```
AudioPlaybackState.currentPlayerTime 변경 (30fps)
    ↓
AudioPlaybackState.objectWillChange 발생
    ↓
AudioPlaybackState를 관찰하는 View만 재렌더링
    ↓
AudioControlView 재렌더링
    ↓
WaveFormView 재렌더링
    ↓
메뉴바는 재렌더링 안 됨! ✅
```

### 데이터 흐름 다이어그램

```
ContentViewModel (let으로 선언된 참조들)
├── audioPlaybackState: AudioPlaybackState
│   └── @Published currentPlayerTime (30fps)
│       └── 관찰: AudioControlView, WaveFormView만
│
├── modelManagementState: ModelManagementState
│   ├── @Published loadingProgressValue
│   └── @Published downloadProgress
│       └── 관찰: ModelSelectorView, ModelManagerView만
│
└── @Published audioPlayer, transcriptionState, ...
    └── 관찰: 전체 View 계층
```

## 아키텍처 고민

### 문제: God Object Anti-pattern

이 문제를 해결하면서 더 근본적인 문제를 발견했다. `ContentViewModel`이 **너무 많은 책임**을 지고 있었다.

```swift
class ContentViewModel: ObservableObject {
    // 오디오 관련 (5개 이상)
    @Published var audioPlayer: AVAudioPlayer?
    @Published var audioState = AudioState()
    let audioPlaybackState = AudioPlaybackState()
    
    // 모델 관련 (10개 이상)
    @Published var whisperKit: WhisperKit?
    let modelManagementState = ModelManagementState()
    @Published var currentLoadedModel: String = ""
    
    // 전사 관련 (5개 이상)
    @Published var transcriptionState = TranscriptionState()
    @Published var transcriptionResult: TranscriptionResult?
    
    // UI 관련
    @Published var uiState = UIState()
    
    // 설정 관련 (15개 이상)
    @AppStorage("selectedModel") var selectedModel: String
    @AppStorage("selectedTask") var selectedTask: String
    // ...
    
    // 메서드도 100개 이상...
    func loadModel() { }
    func playAudio() { }
    func transcribeFile() { }
    func exportTranscription() { }
    // ...
}
```

### 해결 방향 1: Service Layer 패턴

비즈니스 로직을 Service로 분리하고, ViewModel은 UI 로직만 담당하도록 한다.

```swift
// 1. Service 정의
@MainActor
class AudioService: ObservableObject {
    @Published var currentTime: Double = 0.0
    @Published var isPlaying: Bool = false
    @Published var duration: Double = 0.0
    
    private var player: AVAudioPlayer?
    
    func load(url: URL) async throws { }
    func play() { }
    func pause() { }
    func seek(to time: Double) { }
}

@MainActor
class ModelService: ObservableObject {
    @Published var loadingProgress: Float = 0.0
    @Published var state: ModelState = .unloaded
    @Published var currentModel: String = ""
    
    private var whisperKit: WhisperKit?
    
    func loadModel(_ name: String) async throws { }
    func unloadModel() async { }
}

@MainActor
class TranscriptionService: ObservableObject {
    @Published var result: TranscriptionResult?
    @Published var isTranscribing: Bool = false
    
    private let modelService: ModelService
    private let audioService: AudioService
    
    init(modelService: ModelService, audioService: AudioService) {
        self.modelService = modelService
        self.audioService = audioService
    }
    
    func transcribe() async throws {
        guard modelService.state == .loaded else { return }
        // 전사 로직
    }
}
```

```swift
// 2. App에서 Service 주입
@main
struct CaptionMateApp: App {
    @StateObject private var audioService = AudioService()
    @StateObject private var modelService = ModelService()
    @StateObject private var transcriptionService: TranscriptionService
    
    init() {
        let audio = AudioService()
        let model = ModelService()
        let transcription = TranscriptionService(
            modelService: model,
            audioService: audio
        )
        
        _audioService = StateObject(wrappedValue: audio)
        _modelService = StateObject(wrappedValue: model)
        _transcriptionService = StateObject(wrappedValue: transcription)
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(audioService)
                .environmentObject(modelService)
                .environmentObject(transcriptionService)
        }
    }
}
```

```swift
// 3. View에서 필요한 Service만 주입받음
struct AudioControlView: View {
    @EnvironmentObject var audioService: AudioService
    
    var body: some View {
        VStack {
            Text("Time: \(formatTime(audioService.currentTime))")
            
            Button(action: { audioService.play() }) {
                Image(systemName: audioService.isPlaying ? "pause.fill" : "play.fill")
            }
        }
    }
}
```

### 해결 방향 2: Coordinator 패턴

여러 Service 간의 협업이 필요한 경우 Coordinator가 중재한다.

```swift
class AppCoordinator: ObservableObject {
    let audioService: AudioService
    let modelService: ModelService
    let transcriptionService: TranscriptionService
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        self.audioService = AudioService()
        self.modelService = ModelService()
        self.transcriptionService = TranscriptionService(
            modelService: modelService,
            audioService: audioService
        )
        
        setupBindings()
    }
    
    private func setupBindings() {
        // 모델 로딩 완료시 UI 업데이트
        modelService.$state
            .sink { [weak self] state in
                if state == .loaded {
                    self?.notifyModelReady()
                }
            }
            .store(in: &cancellables)
    }
    
    func startTranscription() async {
        do {
            try await transcriptionService.transcribe()
        } catch {
            handleError(error)
        }
    }
}
```

### 해결 방향 3: Combine을 통한 간접 통신

Service들이 직접 참조하지 않고 이벤트 버스를 통해 통신한다.

```swift
class AppEventBus {
    static let shared = AppEventBus()
    
    let modelLoaded = PassthroughSubject<String, Never>()
    let audioLoaded = PassthroughSubject<URL, Never>()
    let transcriptionComplete = PassthroughSubject<TranscriptionResult, Never>()
}

// Service A
class ModelService: ObservableObject {
    func loadModel() async {
        // 모델 로딩...
        AppEventBus.shared.modelLoaded.send(modelName)
    }
}

// Service B
class TranscriptionService: ObservableObject {
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        AppEventBus.shared.modelLoaded
            .sink { [weak self] modelName in
                self?.enableTranscription()
            }
            .store(in: &cancellables)
    }
}
```

### 각 패턴의 장단점 비교

| 패턴 | 장점 | 단점 | 적합한 경우 |
|------|------|------|------------|
| **Service Layer** | - 명확한 책임 분리<br>- 재사용성 높음<br>- 테스트 용이 | - 초기 설계 복잡<br>- 보일러플레이트 증가 | 중대형 프로젝트 |
| **Coordinator** | - 복잡한 흐름 제어 가능<br>- 네비게이션 관리 우수 | - Coordinator 자체가 복잡해질 수 있음 | 화면 전환이 많은 앱 |
| **Event Bus** | - 느슨한 결합<br>- 유연한 통신 | - 디버깅 어려움<br>- 데이터 흐름 추적 힘듦 | 이벤트 기반 아키텍처 |

## 현재 프로젝트의 선택

현재 프로젝트에서는 **점진적 개선** 접근을 선택했다:

1. **단기**: 고빈도 업데이트 상태만 분리 (AudioPlaybackState, ModelManagementState)
2. **중기**: 주요 기능별 Service 분리 고려
3. **장기**: 전체 아키텍처 재설계 (필요시)

### 왜 점진적 접근을 선택했나?

1. **즉각적인 문제 해결**: 메뉴바 깜빡임은 바로 해결됨
2. **최소한의 변경**: 기존 코드를 크게 수정하지 않음
3. **학습 곡선**: 팀원들이 새로운 패턴을 학습할 시간 확보
4. **리스크 관리**: 대규모 리팩토링은 새로운 버그를 만들 수 있음

## 교훈 및 베스트 프랙티스

### 1. ObservableObject 설계 원칙

```swift
// ❌ 나쁜 예: 모든 것을 하나의 ViewModel에
class ViewModel: ObservableObject {
    @Published var frequentlyChanging: Double = 0.0  // 30fps
    @Published var rarelyChanging: String = ""        // 가끔
    @Published var userSettings: Settings = .init()   // 거의 안 바뀜
}

// ✅ 좋은 예: 변경 빈도에 따라 분리
class FrequentState: ObservableObject {
    @Published var value: Double = 0.0  // 30fps
}

class MainViewModel: ObservableObject {
    let frequentState = FrequentState()  // let!
    @Published var rarelyChanging: String = ""
    @Published var userSettings: Settings = .init()
}
```

### 2. 관찰 최소화 원칙

```swift
// ❌ 나쁜 예: 전체 ViewModel 관찰
struct SomeView: View {
    @ObservedObject var viewModel: MainViewModel
    
    var body: some View {
        Text("\(viewModel.frequentState.value)")
        // frequentState가 변경되면
        // MainViewModel.objectWillChange가 발생하지 않아도
        // 명시적으로 관찰해야 함
    }
}

// ✅ 좋은 예: 필요한 State만 관찰
struct SomeView: View {
    @ObservedObject var viewModel: MainViewModel
    @ObservedObject var frequentState: FrequentState
    
    init(viewModel: MainViewModel) {
        self.viewModel = viewModel
        self.frequentState = viewModel.frequentState
    }
    
    var body: some View {
        Text("\(frequentState.value)")
        // frequentState 변경 시 이 View만 재렌더링
    }
}
```

### 3. let vs @Published 선택 기준

```swift
class ParentViewModel: ObservableObject {
    // ✅ 인스턴스 자체가 교체되지 않으면 let 사용
    let childState = ChildState()
    
    // ❌ 이렇게 하면 childState의 변경이 ParentViewModel도 변경시킴
    // @Published var childState = ChildState()
    
    // ✅ 인스턴스가 교체되어야 하면 @Published 사용
    @Published var currentUser: User?  // nil → User 교체
}
```

### 4. 성능 모니터링

```swift
class AudioPlaybackState: ObservableObject {
    @Published var currentPlayerTime: Double = 0.0 {
        willSet {
            #if DEBUG
            print("AudioPlaybackState.objectWillChange called")
            #endif
        }
    }
}

// Instruments를 사용한 프로파일링
// - Time Profiler: CPU 사용량 확인
// - SwiftUI: View body 호출 횟수 확인
```

### 5. 메뉴바 관련 주의사항 (macOS)

- `CommandMenu` 내의 `Menu`는 상태를 유지하는 UI 컴포넌트
- 부모 View의 재렌더링이 발생하면 Menu의 내부 상태가 리셋됨
- 가능한 한 메뉴바와 관련된 ViewModel은 변경을 최소화
- 모달이나 시트가 열릴 때도 비슷한 문제 발생 가능

## 성능 개선 결과

### 변경 전
- 오디오 재생 중 전체 View 계층 재렌더링: **30fps (초당 30회)**
- 메뉴바 재렌더링: **30fps**
- CPU 사용량: **15-20%** (M1 Pro 기준)
- 메뉴 선택 불가 ❌

### 변경 후
- AudioControlView, WaveFormView만 재렌더링: **30fps**
- 메뉴바 재렌더링: **0fps (변경 없음)**
- CPU 사용량: **8-12%** (M1 Pro 기준)
- 메뉴 선택 정상 작동 ✅
- **CPU 사용량 약 40% 감소**

## 결론

SwiftUI의 `ObservableObject`는 강력하지만, 잘못 사용하면 불필요한 재렌더링을 유발할 수 있다. 특히 고빈도로 업데이트되는 상태(타이머, 애니메이션, 스트림 데이터 등)를 다룰 때는:

1. **상태를 변경 빈도에 따라 분리**하여 재렌더링 범위를 최소화
2. **let으로 선언**하여 인스턴스 교체로 인한 불필요한 objectWillChange 방지
3. **View에서 명시적으로 관찰**하여 필요한 컴포넌트만 업데이트
4. **아키텍처 패턴**(Service Layer, Coordinator 등)을 적용하여 책임 분리

이러한 원칙들은 단순히 메뉴바 깜빡임을 해결하는 것을 넘어, 더 나은 앱 아키텍처를 만드는 데 도움이 된다. 작은 버그 하나가 전체 시스템을 재설계하는 계기가 될 수 있다는 것을 배웠다.

## 참고 자료

- [Apple Documentation - Combine](https://developer.apple.com/documentation/combine)
- [SwiftUI Performance Tips - Apple WWDC](https://developer.apple.com/videos/play/wwdc2021/10022/)
- [Managing Model Data in Your App - Apple Documentation](https://developer.apple.com/documentation/swiftui/managing-model-data-in-your-app)

---

**프로젝트**: [CaptionMate](https://github.com/cho407/CaptionMate)  
