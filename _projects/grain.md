---
title: "Grain"
excerpt: "iOS기반 필름카메라 사용자들을 위한 SNS 앱 프로젝트"
header:
  overlay_image: /assets/images/grainBanner.png
  overlay_filter: rgba(0, 0, 0, 0.2)
  actions:
    - label: "Project Github"
      url: "https://github.com/APPSCHOOL1-REPO/finalproject-grain"
  teaser: /assets/images/grainMain.png
sidebar:
  - title: "프로젝트 명"
    text: "Grain"
  - title: "역할"
    text: "iOS 개발, UI/UX 디자인"
  - title: "팀 구성원"
    text: "조형구, 박희경, 지정훈, 홍수만, 윤소희, 한승수"
  - title: "기간"
    text: "2023년 1월 16일 ~"
---

## 프로젝트 개요
레트로 열풍과 SNS 사용량의 급격한 증가는 사진 시장의 소비 심리에 큰 변화를 가져왔습니다. 이러한 트렌드는 아날로그의 따뜻한 감성과 디지털 플랫폼의 편리함이 결합된 새로운 방식의 소통을 요구하게 되었습니다. 이에 따라 "Grain" 은 **필름카메라의 아날로그적 특성을 디지털 SNS 플랫폼에 재현하는 것을 목표**로 설정하였습니다.

"Grain" 은 프로토타입부터 MVP 단계를 거쳐 현재 [앱스토어](https://apps.apple.com/kr/app/grain-그레인-필름-카메라-감성-sns/id6446666081)에서 1.0.2 버전으로 서비스되고 있습니다. 이 앱은 **SwiftUI**를 기반으로 **MVVM 아키텍처**를 활용하여 **iOS 환경**에서 구현되었습니다. 그 과정에서 **사용자 경험 최적화, 고성능 이미지 처리, 네트워킹 최적화**를 기술적 핵심 과제로 삼고 개발되었습니다.

"Grain"을 통해 사용자는 자신의 필름 사진 작품을 공유하고, 다른 사용자들의 작품을 감상하며, 필름카메라에 관한 다양한 정보와 팁을 교환하고 포토스팟, 인화소, 수리점 등의 장소정보 또한 제공 받을 수 있습니다.

## 주요 담당 업무
- 앱 컨셉 구체화, 사용자 시나리오 구상 및 프로토타입, MVP 
- 데이터 구조 설계
- 게시물 검색 기능
- 사용자 차단 및 신고 기능
- 피드 화면 UI/UX 디자인
- 사용자 피드백 관련 애니메이션 및 햅틱 피드백 기능(좋아요, 저장, 댓글 등)
- 게시물 외부 앱으로 공유하는 기능
- 이미지 렌더링 및 업로드 기능
- Localization
- 앱스토어 배포 및 리젝 대응


## 페르소나
필름카메라를 취미로 갖거나, 직업으로 가진 사람들

## ADS
그레인은 나만의 필름감성을 공유하고, 필름사진과 관련된 정보들을 공유하고, 필름 유저들을 위한 커뮤니티를 제공하는 앱이다.

## 주요기능

### 피드 탭

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

   <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/4c4dfc28-dc0e-4e37-9e5a-52efeacf6ae9" alt="피드6" width="150" height="300"> <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/27cd1367-cadc-4047-8d1b-ee6dbb043d8e" alt="피드7" width="150" height="300"> <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/b4ab150a-3cfb-4b31-90e1-6bd8932d8d26" alt="피드8" width="150" height="300"> 
  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/e08025e3-3522-4dd9-b80d-60feeb67f2d6" alt="피드9" width="150" height="300">



### 커뮤니티 탭

- 매칭, 마켓, 클래스, 정보 카테고리에서 각 분야별 모집글을 확인할 수 있습니다.
- 댓글, 대댓글 기능을 통해 다른 사용자와 소통할 수 있습니다.
- 모든 게시글은 최신순으로 정렬됩니다.
- 모집완료, 판매완료 등의 게시글은 스크롤 아래에 배치됩니다.
- 커뮤니티 작성 기능은 사진 5장까지 업로드 가능하며, 제목과 내용은 필수 입력 사항입니다.
- 작성한 게시글의 제목과 내용만 수정 가능하며, 모집/판매 상태를 변경하거나 삭제할 수 있습니다.
- 커뮤니티 게시글은 저장 기능이 있어 마이 페이지에서 확인 가능합니다.
- 다른 유저 게시글에 대한 신고 기능도 있습니다.

  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/141cce08-801b-497b-865b-e5c494f1b3c1" alt="커뮤1,2" width="150" height="300"> <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/737395e2-0b5a-41e1-b980-42306fc5da65" alt="커뮤5,6" width="150" height="300">  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/5e76c676-846f-4bff-b8c9-15a6a7cee84a" alt="커뮤8" width="150" height="300">

  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/e1779d18-97eb-4c76-a559-7565eef01e84" alt="커뮤9" width="150" height="300">


### 지도 탭

- 현재 위치에서 가까운 포토스팟, 현상소, 수리점을 지도상에 표시해줍니다.
- 피드에 글을 등록하면 해당 위치가 지도에 표시되어 다른 사용자들도 확인할 수 있습니다.
- 포토스팟 마커를 클릭하면 가까운 포토스팟 피드도 확인할 수 있습니다.
- 검색 기능을 이용하여 원하는 지역 근처의 포토스팟, 현상소, 수리점을 찾아볼 수 있습니다.
- 제보 기능을 이용하여 등록되지 않은 현상소, 수리점 위치 정보를 제보할 수 있습니다.

  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/7c997670-3a20-4622-8470-6cf7c911ffd4" alt="지도1" width="150" height="300"> <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/56d9b787-e75d-4e98-a07e-9f6bf47757a8" alt="지도2" width="150" height="300">  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/63816f74-8606-4f40-9466-1177ab06026c" alt="지도3" width="150" height="300"> 
 
  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/43db18c6-5bf4-4807-b2c2-709258f1c65a" alt="지도4" width="150" height="300"> <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/428525ef-aef0-4de1-9737-3c701fe2d3ab" alt="지도5" width="150" height="300"> <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/ec50f5ee-9cf5-43d2-be5f-2e17692cafb6" alt="피드8" width="150" height="300"> 


### 마이 페이지 탭

- Grain에서 다른 사용자에게 공개되는 내 정보를 확인할 수 있습니다.
- 구독자 및 현재 구독 중인 사용자 목록을 확인할 수 있습니다.
- 보유한 카메라 및 렌즈 등의 장비 정보를 확인할 수 있습니다.
- 내가 업로드한 피드들을 모아 볼 수 있는 기능이 제공됩니다.

  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/0499812f-a921-4caa-81ba-b87fce33888c" alt="마이1" width="150" height="300"> <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/765efa1c-1df7-4366-8df6-873016a1d65e" alt="마이3" width="150" height="300"> <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/0adafc63-2469-42c6-82f8-8b0f8a71e16f" alt="마이4" width="150" height="300">

  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/c70b075f-225d-4ba4-b67c-7dff95626b92" alt="마이2" width="150" height="300">  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/bd8246b4-9a29-4261-93df-c6b159591d2d" alt="차단1" width="150" height="300">  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/3036ea35-492f-489f-8cd4-dfc3cd74801e" alt="차단2" width="150" height="300"> 

### 검색

- 다른 사용자들이 업로드한 피드 및 커뮤니티 글을 검색하여 찾아볼 수 있습니다.
- 사용자 검색 기능을 이용하여 Grain에 가입한 다른 사용자들을 검색하고, 해당 사용자들을 구독할 수 있습니다.

  <img src="https://github.com/APPSCHOOL1-REPO/finalproject-grain/assets/73868968/4aba7664-12d5-4a0f-8822-b024c41ea4d7" alt="검색1" width="150" height="300">


### 알림

- 다른 사용자가 내 피드를 좋아요를 누르면 알림을 받을 수 있습니다.
- 다른 사용자가 내 피드에 댓글을 달면 알림을 받을 수 있습니다.
- 다른 사용자가 내 댓글에 대댓글을 달면 알림을 받을 수 있습니다.
- 다른 사용자가 나를 구독하면 알림을 받을 수 있습니다.

### 설정

- 사용자 프로필을 편집할 수 있습니다.
- 보유한 장비 정보를 변경할 수 있습니다.
- 내가 작성한 커뮤니티 글을 확인할 수 있습니다.
- 저장된 피드 및 커뮤니티 글을 확인할 수 있습니다.
- 고객센터 또는 피드백 기능을 이용하여 문의사항을 제출할 수 있습니다.
- 로그아웃 기능이 제공됩니다.
- 계정 삭제 기능을 이용하여 사용자 계정을 삭제할 수 있습니다.


## 개발 환경
- Xcode ver. 14.2
- Postman 
- Figma
- Notion
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
- [앱스토어](https://apps.apple.com/kr/app/grain-그레인-필름-카메라-감성-sns/id6446666081)
- [Github](https://github.com/APPSCHOOL1-REPO/finalproject-grain)