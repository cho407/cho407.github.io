---
title: "CaptionMate"
excerpt: "MacOS기반 AI를 활용한 자동 전사 및 자막 생성 앱 프로젝트"
header:
  overlay_image: /assets/images/CMBanner.png
  overlay_filter: rgba(0, 0, 0, 0.2)
  actions:
    - label: "Project Github"
      url: "https://github.com/cho407/CaptionMate"
  teaser: /assets/images/CMpreview.png
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
CaptionMate는 AI를 활용해 음성을 텍스트로 전사해 자막 파일 형식(SRT/FCPXML/WebVTT/JSON)으로 만들어, 자막 작업 시간을 획기적으로 줄여주는 앱 입니다.

## 주요기능

### Whisper AI 모델 관리

- Grain 에디터가 작성한 큐레이션 피드도 제공됩니다.
- 인기 피드글은 좋아요 순으로, 실시간 피드글은 최신순으로 정렬됩니다.
- 실시간 피드글에서는 구독자 필터 기능을 이용하여 자신이 구독한 사람의 피드글만 확인할 수 있습니다.
- 피드에서는 찍은 사진에 대한 카메라, 필름, 렌즈 정보를 함께 공유할 수 있습니다.
- 댓글, 대댓글 기능을 통해 다른 사용자와 소통할 수 있으며, 피드에 대한 좋아요와 저장 기능도 있습니다.
- 자신이 작성한 피드의 제목과 내용만 수정, 삭제할 수 있습니다.
- 다른 유저의 피드에 대한 신고 기능도 있습니다.
- 피드에 대한 공유 기능이 있습니다.
- 피드 작성 기능에서는 사진 5장까지 업로드 가능하며, 피드에 보여줄 장비(카메라, 렌즈, 필름), 제목과 내용은 필수 입력사항입니다. 또한 지도에서 필름사진의 포토스팟 위치를 정해야 합니다. 나만의 장소를 입력해야 업로드가 가능하며, 검색 기능을 이용하여 원하는 지역 근처로 지도를 이동할 수 있습니다.

  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/950f0cea-dbca-4e36-8e85-e878a54bc3f0" alt="피드 1" width="150" height="300"> <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/07f7c80f-954b-4b50-baa4-6dcf3abf2654" alt="피드 2,3" width="150" height="300">  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/3ef66dea-259a-4e18-867b-940c462c34bf" alt="피드4,5" width="150" height="300">


### 음성 전사(Transcription)

- 매칭, 마켓, 클래스, 정보 카테고리에서 각 분야별 모집글을 확인할 수 있습니다.
- 댓글, 대댓글 기능을 통해 다른 사용자와 소통할 수 있습니다.
- 모든 게시글은 최신순으로 정렬됩니다.
- 모집완료, 판매완료 등의 게시글은 스크롤 아래에 배치됩니다.
- 커뮤니티 작성 기능은 사진 5장까지 업로드 가능하며, 제목과 내용은 필수 입력 사항입니다.
- 작성한 게시글의 제목과 내용만 수정 가능하며, 모집/판매 상태를 변경하거나 삭제할 수 있습니다.
- 커뮤니티 게시글은 저장 기능이 있어 마이 페이지에서 확인 가능합니다.
- 다른 유저 게시글에 대한 신고 기능도 있습니다.

  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/141cce08-801b-497b-865b-e5c494f1b3c1" alt="커뮤1,2" width="150" height="300"> <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/737395e2-0b5a-41e1-b980-42306fc5da65" alt="커뮤5,6" width="150" height="300">  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/5e76c676-846f-4bff-b8c9-15a6a7cee84a" alt="커뮤8" width="150" height="300">


### 자막 파일 내보내기

- 현재 위치에서 가까운 포토스팟, 현상소, 수리점을 지도상에 표시해줍니다.
- 피드에 글을 등록하면 해당 위치가 지도에 표시되어 다른 사용자들도 확인할 수 있습니다.
- 포토스팟 마커를 클릭하면 가까운 포토스팟 피드도 확인할 수 있습니다.
- 검색 기능을 이용하여 원하는 지역 근처의 포토스팟, 현상소, 수리점을 찾아볼 수 있습니다.
- 제보 기능을 이용하여 등록되지 않은 현상소, 수리점 위치 정보를 제보할 수 있습니다.

  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/7c997670-3a20-4622-8470-6cf7c911ffd4" alt="지도1" width="150" height="300"> <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/56d9b787-e75d-4e98-a07e-9f6bf47757a8" alt="지도2" width="150" height="300">  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/63816f74-8606-4f40-9466-1177ab06026c" alt="지도3" width="150" height="300"> 
 
 


### 다국어 및 다크모드 지원

- Grain에서 다른 사용자에게 공개되는 내 정보를 확인할 수 있습니다.
- 구독자 및 현재 구독 중인 사용자 목록을 확인할 수 있습니다.
- 보유한 카메라 및 렌즈 등의 장비 정보를 확인할 수 있습니다.
- 내가 업로드한 피드들을 모아 볼 수 있는 기능이 제공됩니다.

  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/0499812f-a921-4caa-81ba-b87fce33888c" alt="마이1" width="150" height="300"> <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/765efa1c-1df7-4366-8df6-873016a1d65e" alt="마이3" width="150" height="300"> <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/0adafc63-2469-42c6-82f8-8b0f8a71e16f" alt="마이4" width="150" height="300">


## 개발 환경
- Xcode ver. 14.2
- Figma
- Github
- Swift ver. 5.7.2
- SwiftUI

## 라이브러리 및 프레임워크
- AppleNotification
- AppleLogin
- GoogleLogin
- KingFisher
- NaverMap
- FirebaseAuth
- FireStore
- FirebaseStorage
- FirebaseMessage

## 관련 링크
- [앱스토어](https://apps.apple.com/kr/app/%EC%BA%A1%EC%85%98-%EB%A9%94%EC%9D%B4%ED%8A%B8-captionmate-%EC%9E%90%EB%8F%99-%EC%9E%90%EB%A7%89-%EC%83%9D%EC%84%B1/id6753956825?mt=12)
- [Github](https://github.com/cho407/CaptionMate)