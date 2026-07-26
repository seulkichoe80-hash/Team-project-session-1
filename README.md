# 📰 [Project B] 뉴스 요약 자동화 워크플로우 (Automated Tech News Summarization Pipeline)

> **RSS 피드 수집부터 Generative AI(OpenAI) 기반 3줄 요약, 노션(Notion) DB 자동 저장까지 연결하는 노코드 무인 업무 자동화 파이프라인 프로젝트**

---

## 📌 1. 프로젝트 개요 (Project Overview)

새로운 프로젝트 시작 시 경쟁사 동향이나 기술 뉴스 수집 과정에서 소요되는 단순 반복 작업과 에너지를 대폭 줄이기 위해 설계된 파이프라인입니다.  
본 프로젝트는 **Make(Integromat)** 자동화 툴을 기반으로 **RSS 피드 데이터 수집 → 조건 필터링 및 중복 방지 → OpenAI GPT 가공(3줄 요약 & 감성 분석 & 썸네일) → Notion DB 저장 → 에러 핸들링 및 알림**까지 전 과정을 사람의 개입 없이 매일 정해진 시각에 무인으로 실행되는 시스템을 구축합니다.

### 🎯 핵심 설계 방향
* **단계별 책임제 운영**: 단순 기능 분담을 넘어 `전처리 → AI 가공 → 저장/에러 처리 → 검증 및 문서화`로 이어지는 파이프라인 흐름 단계별 책임제 운영
* **무중단 안정성 확보**: 스케줄 트리거, GUID/URL 기반 중복 방지, 2회 재시도(Retry/Break) 및 예외 알림 로직 구축
* **비용 최적화**: 주제 키워드 필터링 및 기사 1건당 AI 호출 1회 제한으로 API 비용 폭탄 철저 방지

---

## 👥 2. 팀 구성 및 역할 분담 (R&R)

### 🔄 2.1 전체 파이프라인 흐름 및 역할 구조

```text
[팀원 1: 수집 & 데이터 전처리] ──► [팀원 2: AI 가공 & 프롬프트] ──► [팀원 3: 노션 DB & 에러 핸들링]
                                                                        │
[팀원 4: 총괄 QA & 테스트 케이스 & 문서화] ◄──────────────────────────────┘
```

---

### 👤 2.2 팀원별 세부 역할 및 수행 업무

#### 👤 팀원 1(김승현): 데이터 수집 & 조건 필터링 (Data Ingestion & Preprocessing)
* **역할 범위**: RSS 수집 + 중복 저장 방지 + 주제 필터링
* **실제 수행 업무**:
  * **소스 선정 & 트리거 설정**: 안정적인 기술 뉴스 RSS 피드 주소 등록 및 타임존(Asia/Seoul)/스케줄러 설정
  * **주제 필터링 조건 작성**: 수집된 뉴스 중 미션 요구사항(예: AI 관련 주제)만 통과시키는 Filter/Router 조건 설계
  * **중복 방지 로직 구축**: 매일 정기 실행 시 동일 기사가 중복으로 넘어가지 않도록 하는 방어 로직(GUID/URL 기반 중복 검사) 구성
> 💡 **필요성 & 설계 의도**: RSS만 등록해두면 매일 중복되거나 관련 없는 뉴스까지 전부 AI 호출로 넘어가 무의미한 API 비용이 발생하기 때문입니다.

---

#### 👤 팀원 2(이태우): AI Prompt Engineering & 가공 (AI Processing)
* **역할 범위**: OpenAI 연동 + 3줄 요약 프롬프트 + 보너스 과제(감성 분석/썸네일)
* **실제 수행 업무**:
  * **요약 프롬프트 설계**: 기사 본문을 정확히 3줄로 요약하고 노션 속성에 맞게 출력하도록 System/User Prompt 엔지니어링
  * **JSON Output 파싱**: Make의 `Parse JSON` 모듈을 활용해 요약문과 분류 태그를 매핑 가능한 데이터 형태로 가공
  * **보너스 기능 개발**:
    * **보너스 1**: DALL-E 모듈을 연결하여 뉴스 주제 기반 썸네일 이미지 자동 생성
    * **보너스 2**: 기사의 긍정/부정 감성 분석(Sentiment Analysis) 태그 자동 생성
> 💡 **필요성 & 설계 의도**: 프롬프트 품질 및 JSON 파싱 구조에 따라 AI 요약 결과물의 완성도와 노션 DB 저장 형태가 완전히 달라집니다.

---

#### 👤 팀원 3(공미순): 저장소 아키텍처 & 에러 핸들링 (Storage & Reliability)
* **역할 범위**: Notion DB 구축 + 데이터 매핑 + 재시도 및 실패 알림 로직
* **실제 수행 업무**:
  * **Notion DB 설계**: 요구사항에 맞춘 속성(제목, 요약, 원문 링크, 발행일시, 감성태그, 썸네일) 정의 및 Integration 연결
  * **Make-Notion 데이터 매핑**: AI 가공 결과를 노션 페이지 속성에 올바르게 1:1 매핑
  * **예외 처리 & 재시도 전략 (보너스 과제)**:
    * RSS 비어있음, OpenAI API 장애, 노션 연결 오류 발생 시 최대 2회 재시도(Retry/Break) 및 대체 경로 설계
    * 최종 실패 시 관리자에게 이메일/메신저로 실패 알림 발송 로직 구성
> 💡 **필요성 & 설계 의도**: 사람의 개입 없이 365일 안정적으로 구동되는 '무중단 파이프라인'을 만들기 위한 핵심 축입니다.

---

#### 👤 팀원 4(최슬기): 총괄 QA & 테스팅 & 문서화 (QA & Technical Writer)
* **역할 범위**: 통합 테스트 + 보안 마스킹 + README/기획서 산출물 작성
* **실제 수행 업무**:
  * **End-to-End 통합 테스트**: 1~3번 팀원이 구축한 노드가 끊김 없이 이어서 구동되는지 실행 및 검증
  * **제약사항 & 비용 관리**: 기사 1건당 AI 호출 1회 원칙 준수 여부 및 불필요한 무한 루프 체크
  * **보안 마스킹**: 산출물/스크린샷 내 API Key, Notion Token, 개인 이메일 마스킹 처리
  * **최종 제출물(README) 작성**: 팀 역할 정리, 주제 필터링 기준 선택 이유, 에러 처리 정책 명세서 작성 및 계정 연동 QA 관리
> 💡 **필요성 & 설계 의도**: 팀 프로젝트의 최종 평가 기준인 제출 서류와 시연 스크린샷의 완결성을 담당합니다.

---

## 🔄 3. 자동화 워크플로우 구조 (Workflow Architecture)

```text
[1] 스케줄 트리거 (매일 09:00 AM KST)
         │  (타임존: Asia/Seoul)
         ▼
[2] RSS 피드 수집 (지정된 기술 뉴스 RSS 피드에서 최신 항목 가져오기)
         │
         ▼
[3] 중복 방지 & 주제 필터링 (GUID/URL 기반 중복 저장 방지 + AI 주제 관련 키워드 조건 1건 선택)
         │
         ▼
[4] AI 가공 (OpenAI GPT-4o / DALL-E)
    ├── 3줄 요약 생성 (System/User 프롬프트)
    ├── 긍정/부정/중립 감성 분석 태그 생성 (보너스 2)
    └── DALL-E 커스텀 썸네일 이미지 자동 생성 (보너스 1)
         │
         ▼
[5] 노션(Notion) DB 자동 저장
    (제목, 요약문, 원문 링크, 발행일시, 감성 태그, 썸네일 속성 매핑)
         │
         ▼
[6] 예외 처리 & 에러 핸들링
    ├── RSS 비어있음 / 연동 장애 시 최대 2회 재시도 (Retry/Break)
    └── 최종 실패 시 관리자 이메일/메신저 실패 알림 발송
```

---

## ⚙️ 4. 세부 기능 및 설계 기준

### 1) 데이터 수집 및 주제 필터링
* **스케줄 트리거**: 매일 09:00 AM (Asia/Seoul 타임존) 자동 정기 구동
* **주제 필터링 기준 (키워드 목록)**:
  * 주요 키워드: `AI`, `Artificial Intelligence`, `LLM`, `GPT`, `Machine Learning`, `생성형 AI`, `딥러닝`
  * **선택 이유**: 수많은 IT/기술 뉴스 중 핵심 AI 기술 트렌드 기사만을 선별해 수집 효율을 극대화하고, 불필요한 AI 요약 API 호출 비용 발생을 방지하기 위함입니다.
* **중복 방지 키 적용**: `GUID` 또는 `원문 링크(URL)` 식별자를 활용하여 이미 저장된 기사는 자동으로 스킵합니다.

### 2) AI 요약 및 가공 (OpenAI API)
* **3줄 요약 프롬프트**: 뉴스 본문을 파악한 뒤 개조식(- 형태)의 명확하고 간결한 3줄 요약문 생성
* **기사 1건당 AI 호출 1회 원칙**: 불필요한 호출 최소화
* **JSON Output 파싱**: Make의 `Parse JSON`을 활용하여 요약문과 감성 태그 데이터를 정규화
* **[보너스 1] 썸네일 이미지 자동 생성**: DALL-E 모듈 연동을 통해 뉴스 주제에 맞는 커스텀 썸네일 생성 및 Notion DB 저장
* **[보너스 2] 감성 분석 태그**: 기사의 어조와 내용을 분석하여 `Positive(긍정)`, `Negative(부정)`, `Neutral(중립)` 태그 생성

### 3) 노션(Notion) 데이터베이스 저장
| 속성명 (Property) | 속성 유형 (Type) | 저장 내용 |
| :--- | :--- | :--- |
| **제목 (Title)** | Title | 뉴스 기사 제목 |
| **요약문 (Summary)** | Text / Canvas | AI가 생성한 개조식 3줄 요약문 |
| **원문 링크 (URL)** | URL | 해당 기사 원문 페이지 주소 |
| **발행일시 (Date)** | Date | RSS 피드 기준 기사 작성/발행일 |
| **감성 분석 (Sentiment)** | Select / Tag | 긍정 / 부정 / 중립 태그 |
| **썸네일 (Thumbnail)** | Files & Media / URL | AI 자동 생성 썸네일 이미지 |

---

## 🛡️ 5. 에러 처리 및 안정성 정책 (Error Handling Policy)

1. **AI 호출 제약 및 재시도 상한 설정**
   * **원칙**: 기사 1건당 요약 호출은 1회(기본)로 제한합니다.
   * **재시도 정책**: API 일시적 타임아웃, Rate Limit 발생 시 **최대 2회까지 재시도(Retry/Break)** 후 실패 처리합니다.
   * **선택 이유**: 외부 서비스 장애 발생 시 자동 복구를 시도하여 파이프라인의 무중단 구동을 유지하되, 무한 재시도로 인한 API 과금 폭탄을 방지합니다.

2. **예외 처리 & 관리자 알림 (Exception Handling & Alert)**
   * **RSS 수집 건수 0건 또는 피드 오류**: 시스템 에러로 처리하지 않고 로그 기록 후 안전하게 스킵(Skip).
   * **최종 연동 실패 (Notion/OpenAI 지속 오류)**: 예외 처리 브랜치(Error Handler)를 통해 에러 로그 기록 후 관리자 이메일/메신저로 **실패 알림** 발송.

---

## 🔒 6. 보안 및 규정 준수 (Security & Safeguards)

* **보안 키 관리**: OpenAI API Key 및 Notion Integration Token 등 민감 정보는 Make 환경 변수(Environment Variables)에 보관합니다.
* **산출물 마스킹**: README 문서 및 제출용 스크린샷 이미지 내 API Key, Secret Token, 개인 이메일 주소 등은 전면 **마스킹(`***`)** 처리하였습니다.
* **저작권 준수**: 공개된 공식 RSS 피드 데이터만을 활용하며, 저작권을 침해하는 강제 웹 크롤링 방식을 전면 금지합니다.

---

## 📂 7. 프로젝트 파일 구성

* [README.md](Team-project-session-1/README.md) : 프로젝트 명세 및 시스템 설계 문서
* [팀원별 역할.md](Team-project-session-1/팀원별 역할.md) : 팀원별 R&R 상세 명세서
* [project_mission.md](Team-project-session-1/project_mission.md) : [Project B] 뉴스 요약 자동화 워크플로우 만들기 미션 명세서
* [데이터_수집_및_조건_필터링_설계.md](Team-project-session-1/데이터_수집_및_조건_필터링_설계.md) : 김승현 담당 데이터 수집 & 조건 필터링 상세 기술 설계서
* [rss_url.md](Team-project-session-1/rss_url.md) : 연합뉴스 이미지에서 추출한 분야별 RSS 피드 주소 목록
* [seung_description.md](Team-project-session-1/seung_description.md) : 김승현 담당 RSS 수집 워크플로우 Make.com 임포트 및 실행 가이드 문서
* [seung_Integration RSS, OpenAI (ChatGPT, Sora, Whisper).blueprint.json](Team-project-session-1/seung_Integration%20RSS,%20OpenAI%20(ChatGPT,%20Sora,%20Whisper).blueprint.json) : 김승현 담당 데이터 수집/필터링 설계가 반영된 Make 블루프린트 파일
* [project2_blueprint.json](Team-project-session-1/project2_blueprint.json) : Make 자동화 워크플로우 블루프린트 설정 파일
* [Integration RSS, OpenAI (ChatGPT, Sora, Whisper).blueprint.json](Team-project-session-1/Integration%20RSS,%20OpenAI%20(ChatGPT,%20Sora,%20Whisper).blueprint.json) : RSS & OpenAI 통합 블루프린트 파일

---

## 📝 8. 노션 DB 저장 결과 예시 (Sample Output)

```text
제목: AI 에이전트의 미래와 전망
요약:
  - 최신 AI 에이전트는 자율적인 의사결정 능력이 강화되고 있음
  - 기업 생산성 도구와의 결합 사례가 급증하는 추세임
  - 보안 및 윤리적 가이드라인의 필요성이 대두됨
원문 링크: https://example.com/news/123
발행일: 2026년 7월 26일
감성 분석: 긍정 (Positive)
```
