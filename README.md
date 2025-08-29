# n8n-automation


# n8n 워크플로우에 AI 에이전트 붙여서 나만의 비서봇 만들기

> 코딩 없이 **n8n**과 **AI Agent**를 활용해 나만의 비서봇을 구축하는 튜토리얼입니다.  
> 기본 일정 관리에서 시작해, 뉴스 요약 기능과 외부 웹페이지 명령까지 확장합니다.

---

## 목차
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

 참고 영상: [n8n 설치 가이드](https://youtu.be/DhuaKAW819s?si=TRuZOP8i_0Shcta6)     
   
- 2:35 ~ 5:30 부분 참고해서 docker 설치 및 n8n 다운 가능.
혹은 아래와 같이 터미널에서 설치할 수도 있음

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

###  n8n 특징
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


## 4. 비서봇 실행 (기본 워크플로우)

- 사용 파일: **[secretary.json](./secretary.json)**

### 4-1) 워크플로우 가져오기
- n8n UI 상단 메뉴 **Workflows → Import from File** 선택  
- 저장소의 **`secretary.json`** 파일 업로드 → **Import** 클릭  
- 워크플로우 이름을 확인하고 저장(⌘/Ctrl + S)

### 4-2) 크리덴셜(자격증명) 연결
- 각 노드에서 **Credentials**가 연결되어 있는지 확인
  - **Google Calendar**: `googleCalendarOAuth2Api`
  - **Google Sheets**: `googleSheetsOAuth2Api`
  - **Gmail**: `gmailOAuth2`
  - **OpenAI**: `openAiApi`
- 처음이라면 노드 안 **Credentials → Add credential** 에서 OAuth 인증/키 등록

### 4-3) 타임존 및 환경 설정
- **Workflow Settings** → **Timezone**: `Asia/Seoul`

### 4-4) 채팅으로 테스트 실행
- 트리거: **When chat message received**
- 워크플로우 화면 상단의 **Open chat** 버튼 클릭 → 대화창에서 아래 프롬프트로 테스트

#### 예시 프롬프트 & 기대 동작
- `내일 오후 3시에 홍길동과 미팅 추가해줘. 참석자도 넣어줘.`  
  - 일정 생성 → 리마인더 이메일 여부 확인 후 **Send Email** 실행  
- `오늘 일정 브리핑해줘`  
  - 오늘 일정 조회 및 요약

### 4-5) 실행 로그 확인
- **Executions** 탭에서 노드별 입력/출력 및 AI Tool 호출 확인 가능

### 4-6) (선택) 공개 채팅 링크 만들기
- 트리거 노드 → **Make chat publicly available** 활성화  
- **Hosted Chat** URL 발급 후 공유 가능

---

## 5. 응용 1 – 뉴스 요약 챗봇 확장

- 사용 파일: **[up_secretary.json](./up_secretary.json)**

### 추가 기능
- **RSS Read** → 기술 뉴스 RSS 피드 수집  
- **AI Agent** → 기사 요약  
- **Gmail Tool** → 요약 결과 메일 발송  
- (선택) **Schedule Trigger** → 매일/매주 정기 브리핑  

### 예시 프롬프트
- `오늘의 IT 뉴스 3줄 요약해줘`  
  - RSS → 요약 → 메일 여부 확인

---

## 6. 응용 2 – 외부 웹/스크립트에서 명령 내리기 (Webhook)

### Webhook 설정
- 노드 Path 예: `/webhook/ai`  
- 로컬 실행 시 URL: `http://localhost:5678/webhook/ai`

### HTML 예시
```html
<form action="http://localhost:5678/webhook/ai" method="POST">
  <input type="text" name="chatInput" placeholder="명령 입력" />
  <button type="submit">Send</button>
</form>
```

### cURL 예시
```bash
curl -X POST   -H "Content-Type: application/json"   -d '{"chatInput":"내일 일정 브리핑해줘"}'   http://localhost:5678/webhook/ai
```

---

## 7. 트러블슈팅 체크리스트

- 코드 블록 닫힘 누락 여부 확인  
- Google OAuth 승인 및 API 활성화 확인  
- Workflow 타임존이 `Asia/Seoul`인지 확인  
- Docker 실행 시 `--restart unless-stopped` 옵션 사용 권장  

---

## 8. JSON 파일 바로가기

- **기본 비서봇**: [secretary.json](./secretary.json)  
- **응용 확장 버전**: [up_secretary.json](./up_secretary.json)


