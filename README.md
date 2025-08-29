# n8n-automation


# 🤖 n8n 워크플로우에 AI 에이전트 붙여서 나만의 비서봇 만들기

> 코딩 없이 **n8n**과 **AI Agent**를 활용해 나만의 비서봇을 구축하는 튜토리얼입니다.  
> 기본 일정 관리에서 시작해, 뉴스 요약 기능과 외부 웹페이지 명령까지 확장합니다.

---

## 📌 목차
0. [n8n 설치 및 환경 설정](#0-n8n-설치-및-환경-설정)  
1. [n8n 워크플로우 기본 만들기](#1-n8n-워크플로우-기본-만들기)  
2. [AI Agent 설정하기](#2-ai-agent-설정하기)  
3. [비서봇 만들기 (Google 서비스 연동)](#3-비서봇-만들기-google-서비스-연동)  
4. [응용1 - 뉴스 요약 챗봇](#4-응용1---뉴스-요약-챗봇)  
5. [응용2 - 외부 웹페이지에서 명령 내리기](#5-응용2---외부-웹페이지에서-명령-내리기)  
6. [전체 워크플로우 예시(JSON)](#6-전체-워크플로우-예시json)  
7. [마무리 및 주의사항](#7-마무리-및-주의사항)

---

## 0. n8n 설치 및 환경 설정

👉 참고 영상: [n8n 설치 가이드](https://youtu.be/DhuaKAW819s?si=TRuZOP8i_0Shcta6)

### 설치 명령어
```bash
docker run -it --rm \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```
- `5678` → n8n 기본 포트  
- `~/.n8n` → Credential 및 설정 저장 위치  
- 브라우저에서 `http://localhost:5678` 접속 후 n8n UI 확인

---


## 1. n8n 워크플로우 기본 만들기

- **New Workflow** 클릭  
- **Trigger Node** 추가 (`Manual Trigger` 또는 `Chat message received`)  
- **Set Node**로 데이터 처리  
- **Google Calendar, Gmail, Google Sheets** 등 서비스 노드 연결  

### ✅ n8n 특징
- 오픈소스 & 셀프 호스팅 가능  
- 다양한 노드 지원 (트리거/액션/AI)  
- 보안성 우수  
- **AI Agent 내장** → 다른 자동화 툴과의 차별점  

---

## 2. AI Agent 설정하기

AI 활용 방식은 두 가지입니다:
- **AI Chain**: 정해진 순서대로 실행 중 AI 사용  
- **AI Agent**: AI가 직접 어떤 툴을 사용할지 판단  

### 설정 단계
1. **Trigger Node** → `Chat message received`  
2. **AI Agent Node** 추가  
   - 모델: `OpenAI GPT-4.1 mini` 또는 로컬 모델(LLaMA, Gemma)  
   - Memory: `Simple Memory` (최근 대화 10개 저장)  
3. **System Prompt 작성**
    ```text
    너는 똑똑하고 유능한 어시스턴트야. 오늘 날짜는 {{ $now.format('yyyy-MM-dd') }}야.
    사용자의 일정 관리와 이메일 발송을 도와주는 비서 역할을 한다.
    ```
4. **Timezone 설정** → `Asia/Seoul`

---

## 3. 비서봇 만들기 (Google 서비스 연동)

### Google API 연동 준비
- Google Cloud Console에서 프로젝트 생성  
- OAuth 동의화면 및 OAuth 클라이언트 ID 발급  
- n8n Redirect URI 등록  
- API 활성화: Calendar, Gmail, Sheets  

### 주요 노드
- **Google Calendar**
  - Get Many → 일정 조회  
  - Create Event → 일정 생성  
  - Delete Event → 일정 삭제  
- **Google Sheets**
  - Read → 연락처 시트 조회  
- **Gmail**
  - Send → 일정 리마인더 메일 발송  

---

## 4. 응용1 - 뉴스 요약 챗봇

- **RSS Read Tool** → 기술 뉴스 RSS 불러오기  
- **AI Agent** → 기사 요약  
- **Gmail Tool** → 요약 결과 메일 발송  

### 테스트 문구
