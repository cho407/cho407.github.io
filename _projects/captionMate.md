---
title: "CaptionMate"
excerpt: "MacOS기반 AI를 활용한 자동 전사 및 자막 생성 앱 프로젝트"
header:
  overlay_image: /assets/images/CMBanner.png
  overlay_filter: rgba(0, 0, 0, 0.2)
  actions:
    - label: "Project Github"
      url: "https://github.com/cho407/CaptionMate"
  teaser: /assets/images/CMPreview.png
sidebar:
  - title: "프로젝트 명"
    text: "Caption Mate"
  - title: "역할"
    text: "총괄"
  - title: "팀 구성원"
    text: "조형구"
  - title: "기간"
    text: "2025년 2월 23일 ~"
---

## 프로젝트 개요
뉴미디어 콘텐츠와 SNS 숏폼 시장의 급격한 성장으로 유튜브, 인스타그램, 틱톡 등에서 활동하는 크리에이터의 수가 폭발적으로 증가하고 있습니다. 이에 따라 전문 영상 편집자는 물론 일반 사용자들까지 영상 편집과 자막 제작에 대한 수요가 빠르게 늘어나고 있습니다.

**CaptionMate**는 이러한 흐름 속에서 **“사소한 불편함을 똑똑하게 해결하는 것”**을 목표로 만들어졌습니다. 일일이 작업하지 않아도 누구나 전문적인 자막 결과물을 손쉽게 얻을 수 있도록 설계되었습니다.

현재 **CaptionMate**는 프로토타입과 MVP 단계를 거쳐, 앱스토어에서 1.0.0 버전으로 서비스되고 있습니다.
앱은 **SwiftUI** 기반의 **MVVM 아키텍처**로 개발되었으며, MacOS 환경에 최적화된 온디바이스 AI 프로세싱을 중심으로 구축되었습니다. 개발 과정에서는 **사용자 경험(UX) 최적화, AI 모델 기반의 고성능 음성 인식(ASR), 온디바이스 프로세싱**을 핵심 과제로 삼았습니다.

사용자는 CaptionMate를 통해 **고성능 AI 기반 STT(Speech-to-Text)** 기능을 손쉽게 활용하고, **표준 자막 파일(SRT/FCPXML/WebVTT/JSON)**을 자동으로 생성할 수 있습니다.

## 주요 담당 업무
- 앱 컨셉 구체화, 사용자 시나리오 구상 및 프로토타입, MVP 
- 데이터 구조 설계
- 모델 다운로드 및 관리
- UI/UX 디자인
- 전사 결과물 자막형식 내보내기
- 음성파일 미리듣기 및 파형 View
- Localization
- 앱스토어 배포 및 리젝 대응

## 페르소나
매 영상마다 자막을 직접 치느라 업로드가 늦어지는 컨텐츠 크리에이터.

## ADS
CaptionMate는 AI를 활용해 음성을 텍스트로 전사해 자막 파일 형식(SRT/FCPXML/WebVTT/JSON)으로 만들어, 자막 작업 시간을 획기적으로 줄여주는 앱입니다.

## 주요기능

### Whisper AI 모델 관리

- Tiny, Small, Base, Large 등 다양한 성능의 Whisper AI 모델 지원
- 모델 다운로드 및 삭제 지원
- CPU, GPU, Neural Engine 세 가지 compute units 지원

  <img src="https://github.com/user-attachments/assets/c2f23ed3-6203-4fb0-9936-d3eb2549f19f" alt="모델 관리" width="800"><img src="/assets/images/unit.png" alt="연산 유닛" width="400">


### 음성 전사(Transcription) 및 자막 생성

- 상세 설정 지원
- 고성능 STT 지원
- 영어로 자동 번역 지원
- 다국어 지원
- RTF, Speed Factor, tok/s, First token 값 제공
- 자동 언어 감지 지원
- SRT, WebVTT, JSON, Final Cut Pro XML 지원
- FCPXML파일로 바로 Final cut Pro 자막 클립 생성

  <img src="https://github.com/user-attachments/assets/9ee00eb1-f480-4976-b99b-1e56108a3a55" alt="고급 설정" width="800"><img src="https://github.com/user-attachments/assets/c16ea6f8-ceee-4f97-894e-ec3b36a3fa2a" alt="전사 결과" width="800"><img src="https://github.com/user-attachments/assets/0ba88f70-f56b-4938-859c-274aaaaf0803" alt="자막 내보내기" width="400">


### 음성파일 미리듣기

- 음성 파일 미리듣기 플레이어 제공
- 배속, 음량조절, 파형 제공
- 플레이어 단축키 제공

  <img src="/assets/images/player.png" alt="플레이어" width="800">


### 다국어 및 다크모드 지원

- 다크모드 지원
- 영어, 한국어 언어설정 지원

  <img src="https://github.com/user-attachments/assets/5ff7a473-e12c-45b3-8046-754742df4efe" alt="다크 모드" width="800"><img src="https://github.com/user-attachments/assets/95a0abb3-6b33-4c11-a637-ec966df0cadd" alt="다국어 지원" width="400">

## 사용자 플로우 (User Flow)
<p align="center">
<img width="800" alt="CaptionMate User Flow" src="https://github.com/user-attachments/assets/46d4c513-70de-4836-a48a-1c6b4977152a">
</p>

## 엔티티 관계도 (ERD)
<p align="center">
<img width="800" alt="CaptionMate ERD" src="https://github.com/user-attachments/assets/58322566-82af-4e99-9aee-ab749a5eb981">
</p>

## 개발 환경
- Xcode ver. 16.0
- Figma
- Github
- Swift ver. 6.0.3
- SwiftUI
- mint
- Github Actions

## 라이브러리 및 프레임워크
- WhisperKit
- CoreML
- AVFoundation
- UniformTypeIdentifiers
- Whisper

## 관련 링크
- [앱스토어](https://apps.apple.com/kr/app/%EC%BA%A1%EC%85%98-%EB%A9%94%EC%9D%B4%ED%8A%B8-captionmate-%EC%9E%90%EB%8F%99-%EC%9E%90%EB%A7%89-%EC%83%9D%EC%84%B1/id6753956825?mt=12)
- [Github](https://github.com/cho407/CaptionMate)