---
title: "CoreML/MPSGraph 캐시 충돌과 디스크 용량 관리 문제 해결기"
description: "CaptionMate 개발 중 CoreML 런타임 캐시 충돌과 임시 파일 디스크 폭증 문제를 해결한 과정과 자동 복구 메커니즘 구현"
categories: App
tags: [개발, 앱, CoreML, MPSGraph, 캐시관리, 성능최적화]
sidebar: 
  nav: "docs"
header:
  teaser: https://developer.apple.com/assets/elements/icons/coreml/coreml-256x256_2x.png
---

CaptionMate 앱 개발 중 심각한 문제 두 가지가 발생했다. 첫 번째는 CoreML/MPSGraph 런타임 캐시 충돌로 인한 모델 로딩 실패와 크래시, 두 번째는 모델 추론 과정에서 생성되는 임시 파일들이 맥북의 저장 공간을 과도하게 차지하는 문제였다. 

이 글에서는 이러한 문제들을 체계적으로 분석하고 해결한 과정, 그리고 구현한 자동 복구 메커니즘에 대해 공유하고자 한다.

## 문제 상황

### 문제 A: 모델 전환 시 이전 모델 캐시 충돌로 인한 MPSGraph 크래시

#### 증상
- **모델을 바꾸는 과정에서** 이전 모델의 캐시가 남아있어 충돌 발생
- `MPSGraph` 관련 에러로 **앱 크래시** 발생
- `MPSGraphExecutable` 초기화 실패로 모델 로딩 불가
- 앱 재시작 후에도 동일한 문제 반복

#### 에러 메시지 예시
```
⚠️ Model load failed: MPSGraph error occurred during model initialization
⚠️ Model load failed: MPSGraphExecutable creation failed
⚠️ Model load failed: MPSGraph cache conflict detected
```

#### 문제 발생 시나리오
```
1. whisper-small 모델 로딩 → 정상 작동
2. whisper-large 모델로 전환 시도
3. 이전 모델(whisper-small)의 MPSGraph 캐시가 남아있음
4. 새로운 모델(whisper-large) 로딩 시 캐시 충돌
5. MPSGraph 에러 발생 → 앱 크래시
```

### 문제 B: 임시/캐시 파일 디스크 폭증

#### 증상
- **앱 작동에는 문제없지만** 디스크 용량을 과도하게 차지
- 모델 추론 과정에서 생성되는 임시 파일들이 **정리되지 않고 쌓임**
- 오디오 파일 처리 시 생성되는 임시 파일들 누적
- 다운로드된 모델 파일들의 중복 캐시 누적

#### 디스크 사용량 분석
```
~/Library/Caches/com.example.app/
├── com.apple.e5rt.e5bundlecache/     # CoreML 런타임 캐시 (누적)
├── com.apple.CoreML/                  # CoreML 모델 캐시 (누적)
└── huggingface/models/                # WhisperKit 모델 파일 (누적)

/tmp/
├── coreml_*                          # CoreML 임시 파일 (정리 안됨)
├── whisper_*                         # WhisperKit 임시 파일 (정리 안됨)
├── *.wav, *.mp3, *.m4a              # 오디오 처리 임시 파일 (정리 안됨)
└── download_*                        # 다운로드 임시 파일 (정리 안됨)
```

#### 문제 발생 시나리오
```
1. 오디오 파일 처리 → 임시 파일 생성
2. 모델 추론 실행 → 추가 임시 파일 생성
3. 앱 종료 시 임시 파일 정리 안됨
4. 반복 사용으로 임시 파일 누적
5. 디스크 용량 부족 (수 GB 단위)
```

## 원인 분석

### 문제 A: 모델 전환 시 캐시 충돌 원인

#### 1. MPSGraph 캐시 메커니즘
CoreML은 모델별로 고유한 MPSGraph 캐시를 생성한다:

```swift
// 모델별 MPSGraph 캐시 생성 과정
1. whisper-small 로딩 → MPSGraph A 생성 및 캐시
2. whisper-large 로딩 → MPSGraph B 생성 시도
3. 이전 MPSGraph A 캐시가 남아있어 충돌 발생
4. MPSGraph B 초기화 실패 → 크래시
```

#### 2. 모델 전환 시 캐시 충돌 발생 이유
- **이전 모델의 MPSGraph 캐시가 완전히 해제되지 않음**
- 새로운 모델 로딩 시 이전 캐시와 충돌
- GPU 메모리와 시스템 메모리에서 동시 충돌
- WhisperKit의 모델 해제 메커니즘이 불완전

#### 3. 캐시 파일 손상
- 비정상 종료 시 MPSGraph 캐시 파일이 손상된 상태로 남음
- 손상된 캐시로 인한 새로운 모델 로딩 실패
- 손상된 캐시가 시스템 레벨에서 충돌 유발

### 문제 B: 임시 파일 누적 원인

#### 1. CoreML 추론 과정에서의 임시 파일 생성
```swift
// WhisperKit 추론 과정에서 생성되는 임시 파일들
1. 오디오 파일 로드 → 임시 WAV 파일 생성
2. Mel Spectrogram 계산 → 중간 결과 임시 저장
3. 모델 추론 → GPU 메모리 → 임시 파일로 스왑
4. 결과 후처리 → 추가 임시 파일 생성
5. 앱 종료 시 → 임시 파일 정리 안됨 (누적)
```

#### 2. 캐시 정리 메커니즘 부재
- **앱 종료 시 임시 파일 자동 정리 안 됨**
- **모델 전환 시 이전 모델 캐시 정리 안 됨**
- **디스크 공간 부족 시 자동 정리 메커니즘 없음**
- 반복 사용으로 임시 파일이 수 GB 단위로 누적

## 해결 과정

### 시도 1: 수동 캐시 정리 ❌

```swift
// 앱 시작 시 캐시 정리
func clearCacheOnStartup() {
    let fileManager = FileManager.default
    let cacheDir = fileManager.urls(for: .cachesDirectory, in: .userDomainMask).first!
    
    do {
        let contents = try fileManager.contentsOfDirectory(at: cacheDir, includingPropertiesForKeys: nil)
        for item in contents {
            try fileManager.removeItem(at: item)
        }
    } catch {
        print("Failed to clear cache: \(error)")
    }
}
```

**문제점**: 
- 앱 시작 시에만 정리되어 런타임 중 문제 해결 안 됨
- 사용자 데이터까지 삭제할 위험
- 모델 로딩 중 문제 발생 시 대응 불가

### 시도 2: 에러 발생 시 캐시 정리 ❌

```swift
// 모델 로딩 실패 시 캐시 정리
do {
    try await whisperKit.loadModels()
} catch {
    clearCache()
    // 재시도...
}
```

**문제점**:
- 에러 발생 후에야 대응 (사후 처리)
- 사용자 경험 저하 (로딩 시간 증가)
- 근본적 해결책이 아님

### 시도 3: 자동 복구 메커니즘 (최종 해결) ✅

**핵심 아이디어**: 사전 예방 + 자동 복구 + 지속적 모니터링

#### 1단계: 디스크 공간 사전 확인

```swift
// 디스크 공간 확인 및 캐시 정리
func checkDiskSpace() -> (available: Int64, required: Int64, isEnough: Bool) {
    let fileManager = FileManager.default
    guard let cachesDirectory = fileManager.urls(for: .cachesDirectory, in: .userDomainMask).first else {
        return (available: 0, required: 0, isEnough: false)
    }
    
    do {
        let resourceValues = try cachesDirectory.resourceValues(forKeys: [.volumeAvailableCapacityKey])
        guard let availableSpace = resourceValues.volumeAvailableCapacity else {
            return (available: 0, required: 0, isEnough: false)
        }
        
        let availableSpaceInt64 = Int64(availableSpace)
        let requiredSpace: Int64 = 3_000_000_000 // 3GB
        
        return (
            available: availableSpaceInt64,
            required: requiredSpace,
            isEnough: availableSpaceInt64 > requiredSpace
        )
    } catch {
        print("Failed to check disk space: \(error.localizedDescription)")
        return (available: 0, required: 0, isEnough: false)
    }
}

// 모델 로딩 전 디스크 공간 확인
let diskSpace = checkDiskSpace()
if !diskSpace.isEnough {
    print("⚠️ Insufficient disk space: Available \(diskSpace.available / 1_000_000) MB, Required \(diskSpace.required / 1_000_000) MB")
    clearCoreMLRuntimeCache()
}
```

#### 2단계: CoreML 캐시 디렉토리 정리

```swift
func clearCoreMLRuntimeCache() {
    let fileManager = FileManager.default
    
    // 1. 앱의 캐시 디렉토리 찾기
    guard let cachesDirectory = fileManager.urls(for: .cachesDirectory, in: .userDomainMask).first else {
        print("Cache directory not found.")
        return
    }
    
    // 2. 앱의 번들 ID 가져오기
    let bundleID = Bundle.main.bundleIdentifier ?? "com.example.app"
    
    // 3. CoreML 캐시 디렉토리 찾기
    let possibleCacheDirs = [
        cachesDirectory.appendingPathComponent(bundleID)
            .appendingPathComponent("com.apple.e5rt.e5bundlecache"),
        cachesDirectory.appendingPathComponent(bundleID)
            .appendingPathComponent("com.apple.CoreML"),
        cachesDirectory.appendingPathComponent("com.apple.CoreML"),
        cachesDirectory.appendingPathComponent("CoreML"),
    ]
    
    // 4. 모든 가능한 캐시 디렉토리 정리
    var clearedAny = false
    for cacheDir in possibleCacheDirs {
        if fileManager.fileExists(atPath: cacheDir.path) {
            do {
                let contents = try fileManager.contentsOfDirectory(at: cacheDir, includingPropertiesForKeys: nil)
                for item in contents {
                    do {
                        try fileManager.removeItem(at: item)
                        print("Cache item deleted: \(item.lastPathComponent)")
                        clearedAny = true
                    } catch {
                        print("Failed to delete cache item: \(item.lastPathComponent) - \(error.localizedDescription)")
                    }
                }
                print("CoreML cache directory cleaned: \(cacheDir.path)")
            } catch {
                print("Failed to access CoreML cache directory: \(error.localizedDescription)")
            }
        }
    }
}
```

#### 3단계: 임시 파일 필터링 및 정리

```swift
// 임시 디렉토리 정리
let tempDirectory = URL(fileURLWithPath: NSTemporaryDirectory())
do {
    let tempContents = try fileManager.contentsOfDirectory(
        at: tempDirectory,
        includingPropertiesForKeys: [.fileSizeKey, .creationDateKey]
    )
    
    // 앱 관련 임시 파일 필터링
    let appTempFiles = tempContents.filter { file in
        let fileName = file.lastPathComponent.lowercased()
        return fileName.contains("coreml") ||
            fileName.contains("whisper") ||
            fileName.contains(".bundle") ||
            fileName.contains("model") ||
            fileName.contains("mps") ||
            fileName.contains("mlmodel") ||
            fileName.contains(".wav") ||
            fileName.contains(".mp3") ||
            fileName.contains(".m4a") ||
            fileName.contains(".aac") ||
            fileName.contains(".flac") ||
            fileName.hasPrefix("tmp") ||
            fileName.contains("download")
    }
    
    var totalClearedSize: Int64 = 0
    for tempFile in appTempFiles {
        do {
            let resourceValues = try tempFile.resourceValues(forKeys: [.fileSizeKey])
            let fileSize = resourceValues.fileSize ?? 0
            
            try fileManager.removeItem(at: tempFile)
            totalClearedSize += Int64(fileSize)
            print("Temporary file deleted: \(tempFile.lastPathComponent) (\(ByteCountFormatter.string(fromByteCount: Int64(fileSize), countStyle: .file)))")
            clearedAny = true
        } catch {
            print("Failed to delete temporary file: \(tempFile.lastPathComponent) - \(error.localizedDescription)")
        }
    }
    
    if totalClearedSize > 0 {
        print("Total temporary files cleaned: \(ByteCountFormatter.string(fromByteCount: totalClearedSize, countStyle: .file))")
    }
} catch {
    print("Failed to search temporary directory: \(error.localizedDescription)")
}
```

#### 4단계: MPSGraph 에러 자동 복구

```swift
// 모델 로드 시도
do {
    try await whisperKit.loadModels()
} catch {
    print("⚠️ Model load failed: \(error.localizedDescription)")
    
    // MPSGraph 관련 에러인 경우 캐시 정리 후 한 번만 재시도
    let errorString = error.localizedDescription
    if errorString.contains("MPSGraph") || 
       errorString.contains("MPSGraphExecutable") || 
       errorString.contains("No space left on device") {
        
        print("MPSGraph error or disk space shortage detected, retrying after cache cleanup...")
        clearCoreMLRuntimeCache()
        
        // 한 번 더 시도
        do {
            try await Task.sleep(nanoseconds: 500_000_000) // 500ms 대기
            try await whisperKit.loadModels()
        } catch {
            await MainActor.run {
                modelManagementState.modelState = .unloaded
                modelManagementState.hasModelLoadError = true
                modelManagementState.modelLoadError = "Model load failed (after retry): \(error.localizedDescription)"
            }
            return
        }
    } else {
        // 다른 에러는 즉시 실패 처리
        await MainActor.run {
            modelManagementState.modelState = .unloaded
            modelManagementState.hasModelLoadError = true
            modelManagementState.modelLoadError = error.localizedDescription
        }
    }
}
```

## 해결 원리

### 변경 전: 수동 관리 (Manual Management)

```
사용자 앱 사용
    ↓
모델 로딩 시도
    ↓
MPSGraph 에러 발생
    ↓
앱 크래시 또는 수동 재시작
    ↓
임시 파일 누적
    ↓
디스크 공간 부족
```

### 변경 후: 자동 복구 시스템 (Auto-Recovery System)

```
모델 로딩 시도
    ↓
디스크 공간 사전 확인
    ↓
부족 시 자동 캐시 정리
    ↓
모델 로딩 시도
    ↓
MPSGraph 에러 감지
    ↓
자동 캐시 정리 + 재시도
    ↓
성공 또는 명확한 에러 메시지
```

## 해결 로직 및 메커니즘

### 문제: 캐시 관리의 복잡성

CoreML 캐시 관리는 단순한 파일 삭제가 아니라 여러 요소를 고려해야 한다:

1. **캐시 위치의 다양성**: Apple이 버전별로 캐시 위치를 변경
2. **안전성**: 시스템 파일과 앱 파일 구분
3. **효율성**: 필요한 파일만 정확히 타겟팅
4. **사용자 경험**: 정리 과정이 사용자에게 미치는 영향 최소화

### 해결 메커니즘: 이중 캐시 관리 시스템

#### 문제 A 해결: 모델 전환 시 MPSGraph 캐시 정리 메커니즘
```swift
// 모델 로딩 전 이전 모델 캐시 정리
func loadModel(_ model: String, redownload: Bool = false) {
    // 1. 기존 WhisperKit 인스턴스 해제
    if let kit = whisperKit {
        await kit.unloadModels()
        whisperKit = nil
    }
    
    // 2. MPSGraph 에러 감지 및 자동 복구
    do {
        try await whisperKit.loadModels()
    } catch {
        if errorString.contains("MPSGraph") {
            clearCoreMLRuntimeCache()  // MPSGraph 캐시 정리
            try await whisperKit.loadModels()  // 재시도
        }
    }
}
```

#### 문제 B 해결: 임시 파일 정리 메커니즘
```swift
// 앱별 임시 파일 필터링 및 정리
func clearCoreMLRuntimeCache() {
    // 1. CoreML 캐시 디렉토리 정리
    let possibleCacheDirs = [
        cachesDirectory.appendingPathComponent("com.apple.e5rt.e5bundlecache"),
        cachesDirectory.appendingPathComponent("com.apple.CoreML"),
        // ...
    ]
    
    // 2. 임시 파일 필터링 및 정리
    let appTempFiles = tempContents.filter { file in
        let fileName = file.lastPathComponent.lowercased()
        return fileName.contains("coreml") ||
            fileName.contains("whisper") ||
            fileName.contains("model") ||
            // ... 앱 관련 파일만 필터링
    }
}
```

### 각 메커니즘의 특징 비교

| 메커니즘 | 목적 | 적용 시점 | 대상 |
|----------|------|-----------|------|
| **MPSGraph 캐시 정리** | 모델 전환 시 충돌 방지 | 모델 로딩 전/에러 발생 시 | MPSGraph 런타임 캐시 |
| **임시 파일 정리** | 디스크 용량 절약 | 디스크 부족 시/주기적 | 임시/캐시 파일 |

## 현재 프로젝트의 선택

현재 CaptionMate에서는 **이중 캐시 관리 시스템**을 구현했다:

1. **문제 A 해결**: 모델 전환 시 MPSGraph 캐시 정리 메커니즘
2. **문제 B 해결**: 임시 파일 정리 메커니즘
3. **통합 관리**: 두 메커니즘을 하나의 `clearCoreMLRuntimeCache()` 함수로 통합

### 왜 이 방식을 선택했나?

1. **문제별 맞춤 해결**: 각 문제의 특성에 맞는 정확한 해결책
2. **코드 재사용성**: 하나의 함수로 두 문제 모두 해결
3. **효율성**: 필요한 시점에만 정리하여 성능 영향 최소화
4. **유지보수성**: 명확한 책임 분리로 코드 관리 용이

## 교훈 및 베스트 프랙티스

### 1. CoreML 캐시 관리 원칙

```swift
// ✅ 좋은 예: 다중 캐시 디렉토리 대응
let possibleCacheDirs = [
    cachesDirectory.appendingPathComponent(bundleID)
        .appendingPathComponent("com.apple.e5rt.e5bundlecache"),
    cachesDirectory.appendingPathComponent(bundleID)
        .appendingPathComponent("com.apple.CoreML"),
    cachesDirectory.appendingPathComponent("com.apple.CoreML"),
    cachesDirectory.appendingPathComponent("CoreML"),
]

// ❌ 나쁜 예: 단일 경로만 고려
let cacheDir = cachesDirectory.appendingPathComponent("CoreML")
```

### 2. 안전한 파일 삭제

```swift
// ✅ 좋은 예: 앱별 파일 필터링
let appTempFiles = tempContents.filter { file in
    let fileName = file.lastPathComponent.lowercased()
    return fileName.contains("coreml") ||
        fileName.contains("whisper") ||
        fileName.contains("model") ||
        // ... 앱 관련 파일만 필터링
}

// ❌ 나쁜 예: 전체 임시 디렉토리 삭제
try fileManager.removeItem(at: tempDirectory)
```

### 3. 에러 복구 메커니즘

```swift
// ✅ 좋은 예: 특정 에러만 재시도
if errorString.contains("MPSGraph") || 
   errorString.contains("MPSGraphExecutable") {
    clearCoreMLRuntimeCache()
    try await whisperKit.loadModels()
}

// ❌ 나쁜 예: 모든 에러에 재시도
catch {
    clearCache()
    retry()
}
```

### 4. 디스크 공간 모니터링

```swift
// ✅ 좋은 예: 모델별 공간 요구사항 계산
let estimatedSize: Int64
if model.contains("large") {
    estimatedSize = 3_600_000_000 // 3GB + 20%
} else if model.contains("medium") {
    estimatedSize = 1_800_000_000 // 1.5GB + 20%
} else {
    estimatedSize = 600_000_000 // 500MB + 20%
}

// ❌ 나쁜 예: 고정된 공간 요구사항
let requiredSpace: Int64 = 1_000_000_000 // 1GB
```

### 5. 로깅 및 모니터링

```swift
// ✅ 좋은 예: 상세한 로깅
print("Cache item deleted: \(item.lastPathComponent)")
print("Total temporary files cleaned: \(ByteCountFormatter.string(fromByteCount: totalClearedSize, countStyle: .file))")
print("CoreML cache directory cleaned: \(cacheDir.path)")

// ❌ 나쁜 예: 로깅 없음
try fileManager.removeItem(at: item)
```

## 성능 개선 결과

### 변경 전
- MPSGraph 에러 발생 시 앱 크래시 또는 수동 재시작 필요
- 임시 파일 누적으로 디스크 공간 부족
- 모델 로딩 실패율: 높음
- 사용자 경험: 불안정

### 변경 후
- MPSGraph 에러 자동 감지 및 복구
- 디스크 공간 사전 확인으로 예방적 관리
- 임시 파일 자동 정리로 디스크 공간 관리

## 결론

CoreML/MPSGraph 캐시 관리는 단순한 파일 삭제가 아니라 **시스템의 안정성과 사용자 경험을 보장하는 핵심 인프라**다. 

이번 문제 해결 과정을 통해 배운 것들:

1. **예방적 관리의 중요성**: 문제 발생 전에 미리 대응하는 것이 가장 효과적
2. **자동 복구 메커니즘**: 사용자 개입 없이 시스템이 스스로 문제를 해결
3. **안전한 파일 관리**: 시스템 파일과 앱 파일을 구분하여 안전하게 관리
4. **지속적 모니터링**: 일회성 해결이 아닌 지속적인 관리 시스템 구축

이러한 원칙들은 CoreML뿐만 아니라 다른 머신러닝 프레임워크나 대용량 데이터를 다루는 앱에서도 적용할 수 있는 범용적인 해결책이다. 작은 캐시 문제 하나가 전체 앱의 안정성을 좌우할 수 있다는 것을 배웠다.

## 참고 자료

- [Apple Documentation - CoreML](https://developer.apple.com/documentation/coreml)
- [Managing Model Data in Your App - Apple Documentation](https://developer.apple.com/documentation/coreml/managing_model_data_in_your_app)
- [MPSGraph Documentation - Apple](https://developer.apple.com/documentation/metalperformanceshadersgraph)

---

**프로젝트**: [CaptionMate](https://github.com/cho407/CaptionMate)
