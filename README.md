# n8n-automation

# n8n AI 일정/리마인더 자동화 워크플로우 튜토리얼

이 튜토리얼은 n8n으로 만든 “AI 기반 일정 어시스턴트” 워크플로우 사용법을 안내합니다.  
이 워크플로우는 구글 캘린더, 구글 시트, 지메일, RSS 뉴스, OpenAI를 활용해 팀 일정 관리와 리마인더 메일 발송, 기술 뉴스 요약까지 자동 처리합니다.

---

## 주요 기능 요약

- 채팅으로 일정 조회, 추가, 변경 자동 처리  
- 구글 시트에 저장된 팀원 이메일 자동 연동  
- 일정 등록/변경 시 리마인더 메일 발송(동의 시)  
- 기술 뉴스 요약 및 이메일 전송  
- OpenAI 기반 자연어 처리로 대화형 인터페이스 지원

---

## 전체 워크플로우 구조

![](./docs/n8n-workflow-overview.png)  
(*워크플로우 구성도 예시 이미지를 docs/에 추가해보세요*)

---

## 노드별 상세 설명

### 1. 트리거 노드

- **When chat message received**  
  → 채팅 입력을 받을 때마다 워크플로우 전체가 시작됨

- **Schedule Trigger**  
  → 일정 주기(예: 1시간)에 한 번 자동 실행 (옵션)

- **Webhook**  
  → 외부 서비스에서 HTTP POST로 명령을 받을 때 워크플로우 시작

---

### 2. 입력 가공

- **Edit Fields / Edit Fields1**  
  → 입력값(chatInput 등)을 원하는 형태로 세팅

---

### 3. AI Agent (핵심 엔진)

- **AI Agent**  
  → OpenAI LLM과 Langchain을 활용해 자연어 명령을 이해  
  → 아래 도구(툴) 노드들을 상황에 맞게 호출

---

### 4. 도구(툴) 노드들

- **Get Contacts**  
  → 구글시트에서 팀원 이메일 목록 자동 조회

- **Get Schedule**  
  → 구글캘린더에서 오늘 또는 특정 날짜의 일정 조회

- **Create Event**  
  → 새 일정 추가

- **Delete Event**  
  → 기존 일정 삭제(일정 변경 시 필수)

- **Send Email**  
  → 일정 리마인더 메일 전송 (사용자 동의 시)

- **RSS Read**  
  → 기술 뉴스 피드를 읽어와 요약

- **OpenAI Chat Model**  
  → AI Agent가 자연어 명령을 처리하는 데 사용

---

## 동작 원리 예시

1. 사용자가 “내일 일정 알려줘”라고 채팅 입력
2. AI Agent가 입력 인식 → Get Contacts/ Get Schedule 노드 호출
3. 내일 일정이 없다면, 일정 추가 안내 후 Create Event 호출
4. 일정 생성/수정 시, “리마인더 이메일 보낼까요?” 질문  
 → 동의하면 Send Email 노드가 메일 발송

---

## 직접 사용해보기

### 1. 환경 변수/연동 설정

- Google Calendar, Sheets, Gmail, OpenAI 등 인증 정보 필수 입력
- 예시 시트 주소:  
  https://docs.google.com/spreadsheets/d/1-FbSUvOmuztxiHSZphf-VcR68qRwBSwbR5MtCcSh3bQ/edit

### 2. 워크플로우 import

1. n8n 에서 ‘새 워크플로우’ → Import → 위 JSON 붙여넣기
2. 각 노드의 연동 계정/토큰 확인 및 연결

### 3. 사용법

- Webhook URL 또는 채팅 인터페이스/슬랙 연동 등으로 명령을 보냅니다.
- 예시 문장:
    - “내일 일정 알려줘”
    - “8월 29일 오전 10시에 회의 추가해줘”
    - “일정 변경해줘”
    - “팀원 전체에게 리마인더 보내줘”
    - “오늘의 기술 뉴스 요약해줘”

---

## 주요 커스텀(프롬프트) 예시

- AI Agent의 systemMessage는 아래와 같이 세팅되어 있습니다.
```
너는 똑똑하고 유능한 어시스턴트야. 오늘 날짜는 {{ $now.format('YYYY-MM-DD') }}야.
...
(※ 전체 프롬프트는 워크플로우 JSON 참조)
```
