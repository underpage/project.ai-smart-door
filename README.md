# AI Smart Door System

## 개요

**스마트 현관 방문자 감지 및 알림 시스템**

아두이노 웹캠을 활용하여 현관 앞을 24시간 모니터링하는 스마트 보안 시스템입니다.  
YOLO 모델을 통해 사람, 택배 등 다양한 객체를 실시간 감지하고, 텔레그램을 통해 사용자에게 즉시 알립니다.



## 주요 기능

**핵심 기능**
- 웹 대시보드를 통해 현관 모니터링 제공
- YOLO 기반 객체 감지 (사람, 패키지, 배달 음식, 우편물, 미분류)
- 이벤트 발생시 텔레그램을 통해 사용자에게 즉각 알림

**이벤트 관리**
- 이벤트 히스토리 조회
- 이벤트 필터링 및 검색 (날짜, 객체 타입)
- 감지 전후 상황 파악을 위해 영상 클립(연속 프레임) 저장
- 보관 기간이 지나면 데이터 자동 삭제
- 미분류 이미지 자동 수집 및 적재


### 로드맵
- 대시보드를 통한 카메라 원격 제어 (재시작, 설정 변경 등)
- 객체가 움직이는 경우 추적 및 기록 (SORT/DeepSORT 알고리즘)
- 행동 패턴 학습 (자주 오는 방문자 인식, 비정상 행동 감지 등)



## 워크 플로우

사용자 관점과 데이터 처리 관점에서의 시스템 동작 흐름

**주요 흐름**
1. 현관문 앞에 카메라 설치
2. 백엔드 서버에서 ESP32-CAM의 URL에 접근해 영상 스트림 수신(Consume)
3. 수신된 영상을 1초에 n개 프레임으로 추출하고 OpenCV 전처리(화질 개선) 수행
4. 처리된 이미지를 AI Service Platform에 전송
5. AI Service에서 객체 감지 및 분류를 수행하고 감지가 되면 결과를 반환
6. 결과 수신시 감지 전 3초 + 감지 후 7초 분량의 영상 클립(연속 프레임)을 스토리지에 업로드
7. 감지된 객체에 대한 정보를 데이터베이스에 저장
8. 미분류로 분류된 이미지는 별도로 보관해 향후 모델 재학습에 활용
9. 동시에 객체 탐지 상황을 사용자에게 알림


**용어 정리**

용어 | 설명
---|---
이벤트 | AI 모델이 객체를 탐지한 상황
임계값 | AI가 객체라고 판단하는 기준
스트림 | 연속적으로 전송되는 데이터 흐름
프레임 | 영상을 구성하는 이미지 한 장으로 1초 영상은 24~30개의 프레임으로 구성됨
클립 | 특정 기준을 중심으로 잘라낸 영상 조각


**제약 사항**  
아두이노를 현관문 앞에 24시간 운영하기에 물리적 제약이 존재하므로  
개발 단계에서는 사전 녹화된 영상 파일을 사용하여 핵심 기능을 개발하고 검증합니다.  



## 시스템 아키텍처

본 시스템은 **AI Smart Door**과 **AI Service Platform**(외부) 두 개의 독립적인 프로젝트로 구성됩니다.


**전체 구성도**
```
 [H/W & Users]              [AI Smart Door (Project)]                [External Systems]
┌─────────────┐       ┌───────────────────────────────────┐       ┌───────────────────────┐
│  ESP32-CAM  │──────>│  Stream Consumer (Background)     │       │  AI Service Platform  │
│ (Web Server)│ HTTP  │  - Frame Capture                  │  HTTP │                       │
└─────────────┘       │  - API Request Handler            │──────>│  - Inference API      │
                      │                                   │<──────│  - Model Management   │
┌─────────────┐       ├───────────────────────────────────┤  JSON └───────────────────────┘
│  End User   │<──────│  Notification Service             │
│ (Telegram)  │  Msg  │  - Telegram Bot                   │       ┌───────────────────────┐
└─────────────┘       │                                   │ S3    │  MinIO Server         │
                      ├───────────────────────────────────┤ Proto │                       │
┌─────────────┐       │  Dashboard                        │──────>│  - images/            │
│ Admin (Web) │──────>│  - Event Gallery                  │       └───────────────────────┘
└─────────────┘       │  - Event Log                      │
                      └───────────────────────────────────┘
```


### 구성 요소 역할

**Hardware**
- WiFi 연결 관리 및 MJPEG 영상 스트림 송출

**AI Smart Door**
- 영상 수집 및 전처리
- 이벤트 관리 및 알림
- 사용자 인터페이스

**AI Service Platform**
- AI 추론 서빙
- 모델 라이프사이클 관리

**MinIO**
- 모델 레지스트리
- 학습 데이터 저장
- 이벤트 저장



## 프로젝트 구성

**구조**
```bash
ai-smart-door/
├── camera/                 # 1. Hardware
│   ├── esp32-cam/
│   └── README.md
├── server/                 # 2. Backend (FastAPI)
└── dashboard/              # 3. Frontend (React)
```

**저장소**
- [Camera](./camera/README.md): ESP32-CAM 구동을 위한 가이드
- [Server](./server/README.md): 영상 처리, AI 연동, API 구현
- [Dashboard](./dashboard/README.md): 웹 대시보드



### 기술 스택

**AI Smart Door**
- Backend
  - Python 3.11
  - FastAPI
  - OpenCV
  - Telegram Bot API
  - SQLModel
- Frontend
  - Vite
  - React

**외부 의존성**
- Gateway
  - Nginx
- Database
  - Redis
  - MySQL
- MinIO
- AI Service Platform
  - YOLO



## 프로젝트 설정 및 실행


**네트워크**
```
┌────────────────────────────────────────────────────────┐
│                      my-net (외부 네트워크)              │
│                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │    MySQL     │  │    MinIO     │  │  AI Service  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│         ↑                 ↑                  ↑         │
│         └─────────────────┼──────────────────┘         │
│                  ┌────────┴─────────┐                  │
│                  │ smartdoor-server │                  │
│                  └────────┬─────────┘                  │
└───────────────────────────┼────────────────────────────┘
                            │
┌───────────────────────────┼────────────────────────────┐
│              smartdoor-internal (내부 네트워크)          │
│                           │                            │
│  ┌────────────────────────┴─────┐                      │
│  │      smartdoor-server        │                      │
│  └────────┬──────────────────┬──┘                      │
│  ┌────────┴────────┐  ┌──────┴──────────┐              │
│  │ smartdoor-redis │  │ smartdoor-nginx │              │
│  └─────────────────┘  └─────────┬───────┘              │
│                       ┌─────────┴───────────┐          │
│                       │ smartdoor-dashboard │          │
│                       └─────────────────────┘          │
└────────────────────────────────────────────────────────┘
```


**포트 구성**

서비스 | 포트 | 설명
---|---|---
Nginx     | 80 | 진입점
Dashboard | 3000 | 웹 대시보드
Server    | 8000 | REST API 서버
Redis | 6379 | 인메모리
MySQL | 3306 | 데이터베이스
MinIO | 9000 (API), 9001 (Web) | 스토리지
AI Service Platform | 8090 | 추론 API


**실행**
- 프로젝트 실행전 데이터베이스, 스토리지, AI Service가 실행되어야 합니다.
- 각 서비스는 `my-net` 네트워크에 연결되어야 합니다.
- 각 서비스의 컨테이너 이름이 env에 정의되어야 합니다.

```bash
cp .env.example .env

# 전체 서비스 실행
podman-compose up -d

# 개별 서비스 실행
podman-compose up server

# 로그 확인
docker-compose logs -f server

# 네트워크 연결 확인
podman inspect smartdoor-server | grep -A 10 Networks
```


**배포**
```bash
# 프로덕션 이미지 빌드 및 배포
podman-compose -f podman-compose.prod.yml up -d --build
```