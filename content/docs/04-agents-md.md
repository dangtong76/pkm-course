---
title: "04. AGENTS.md 만들기"
weight: 4
date: 2026-05-10
draft: false
---

> **⏰ 시간**: 1시간 (이론 슬라이드 30분 + 실습 30분)
> **🎯 목표**: *작고 강력한* AGENTS.md를 직접 작성해 본인의 PKM Vault·코드 프로젝트에 적용

---

## 📄 이론 슬라이드

> 이 챕터의 이론(AGENTS.md의 정체·Instruction Budget·Progressive Disclosure·Monorepo 전략·안티패턴)은 아래 슬라이드로 다룹니다. 슬라이드를 먼저 보거나 강의를 들은 뒤 아래 실습으로 넘어가세요.

{{< pdf-iframe src="04-agents-md.pdf" >}}

---

## 🧭 한 줄 요약

> **이상적인 AGENTS.md는 가능한 한 작아야 한다.**
> Instruction Budget(150~200개 명령어) 안에서, **본질만 루트에 두고 나머지는 Progressive Disclosure로 가리키기.**

---

## 📚 학습 목표

이 챕터를 마치면 다음을 할 수 있습니다:

- [ ] **AGENTS.md의 정체와 역할**을 동료에게 한 문장으로 설명할 수 있다
- [ ] **Instruction Budget** 개념을 이해하고, 자기 AGENTS.md가 예산을 초과했는지 진단할 수 있다
- [ ] 자기 프로젝트의 AGENTS.md를 **Progressive Disclosure 원칙**으로 리팩토링할 수 있다
- [ ] **Monorepo**에서 루트 + 패키지 AGENTS.md 계층 구조를 설계할 수 있다
- [ ] 자동 생성·구조 문서화 등 **안티패턴을 식별**하고 회피할 수 있다

---

## 🛠️ 사전 요구사항

| 항목 | 요구 사항 |
|------|---------|
| **OpenCode** | Chapter 01에서 설치 완료 (`opencode --version` OK) |
| **omo (Oh-My-OpenAgent)** | `bunx oh-my-opencode doctor` 모든 항목 ✅ — 특히 **Prometheus·Sisyphus** 에이전트가 `/agents`에 보여야 함 |
| **Obsidian Vault** | Chapter 03에서 만든 Vault (없으면 임의 경로의 Vault 1개) |
| **강의 교재 PDF** | 이 페이지 상단의 `04-agents-md.pdf` (브라우저 우상단 다운로드 버튼 또는 [📥 직접 다운로드](/pdf/04-agents-md.pdf)) |

---

## 🪜 실습 시나리오 — *"슬라이드 PDF를 내 두 개의 AGENTS.md로 변환한다"*

> 슬라이드를 한 번 본 것만으로는 내 시스템이 바뀌지 않습니다.
> 이 실습에서는 **방금 본 강의 PDF 그 자체를** 입력으로 삼아, omo에게 두 종류의 AGENTS.md를 만들게 합니다.
>
> | 산출물 | 경로 | 역할 |
> |---|---|---|
> | **GLOBAL AGENTS.md** | `~/.config/opencode/AGENTS.md` | 모든 OpenCode 세션에 자동 주입되는 *나만의 전역 선호*(언어·페르소나·안티패턴 등) |
> | **Vault AGENTS.md** | `<vault>/AGENTS.md` | PKM 작업 시에만 적용되는 *Vault 컨텍스트*(폴더 구조·노트 규칙·태그 정책 등) |

### Step 1. 강의 PDF → Markdown 변환 (10분)

OpenCode는 PDF를 직접 읽을 수 있습니다(`Read` tool이 PDF 멀티모달 지원). 외부 변환 도구 없이 **omo의 `multimodal-looker` 에이전트**에게 시키는 게 가장 깔끔합니다.

#### 1-1. 작업 디렉토리 준비

```bash
mkdir -p ~/AI-Knowledge/_inbox/agents-md-guide
cd ~/AI-Knowledge/_inbox/agents-md-guide
# 다운로드 받은 PDF를 이 폴더로 이동
mv ~/Downloads/04-agents-md.pdf .
opencode
```

#### 1-2. 변환 프롬프트

OpenCode TUI에서:

```
이 폴더의 04-agents-md.pdf를 읽고 슬라이드 내용 전체를
agents-md-guide.md 라는 마크다운 파일로 정리해줘.

요구사항:
- 슬라이드 번호 + 제목 + 핵심 메시지를 H2/H3 위계로
- 본문은 슬라이드의 bullet/표를 그대로 보존
- 코드 블록·표·예시는 빠짐없이 포함
- 마지막에 "핵심 5원칙 한눈에" 요약 표 추가

multimodal-looker로 위임해줘. ultrawork.
```

#### 1-3. 결과 확인

```bash
ls -la agents-md-guide.md
wc -l agents-md-guide.md  # 대략 200~400줄 예상
```

> ✅ **성공 기준**: 슬라이드 1~24장의 내용이 하나의 마크다운으로 압축돼 있고, 핵심 개념(Instruction Budget·Progressive Disclosure·Ball of Mud·Monorepo 전략 등)이 모두 포함되어 있다.

---

### Step 2. Prometheus에게 두 AGENTS.md 작성 *계획* 요청 (10분)

> ⚠️ **여기서 코드를 바로 짜라고 하면 안 됩니다.**
> Prometheus는 **plan-only 에이전트**입니다. 인터뷰로 사용자의 의도·제약·우선순위를 끌어낸 뒤, 실행 가능한 계획을 `.sisyphus/plans/*.md`에 저장합니다. 이 분리가 곧 "검증 대역폭" 확보입니다 (02 챕터 메시지와 직결).

#### 2-1. Prometheus 호출 프롬프트

같은 OpenCode 세션에서:

```
@prometheus

방금 정리한 agents-md-guide.md를 기반으로,
나의 두 종류 AGENTS.md를 작성하기 위한 계획을 세워줘.

산출물 2개:
1. GLOBAL AGENTS.md  →  ~/.config/opencode/AGENTS.md
2. Vault AGENTS.md   →  <내 Vault root>/AGENTS.md

요구사항:
- 두 파일의 역할을 명확히 분리할 것 (전역 선호 vs Vault 컨텍스트)
- agents-md-guide.md의 핵심 원칙 적용:
  · Instruction Budget 150~200 명령어 이내
  · Progressive Disclosure (Skills/하위 AGENTS.md로 분리)
  · 구조 문서화 금지, 역량 중심 기술
  · 자동생성 안티패턴 회피
- 계획에는 다음을 포함할 것:
  a) 두 파일에 각각 들어갈 섹션 목록 + 예상 명령어 수
  b) Skill 또는 하위 AGENTS.md로 빼야 할 항목
  c) 인터뷰로 나에게 추가로 물어봐야 할 질문들
  d) 작성 순서와 검증 방법

계획만 만들고 실제 파일 작성은 아직 하지 마.
```

#### 2-2. Prometheus 인터뷰 응답

Prometheus가 다음과 같은 질문을 던질 가능성이 높습니다 (예시):

| 카테고리 | 예상 질문 |
|---|---|
| **언어/스타일** | 응답 언어(한국어/영어), 어조(격식/캐주얼), 코드 주석 언어 |
| **도메인** | 본업 분야 — 인프라/AI/프론트/백엔드 등 |
| **PKM 방법론** | PARA / 제텔카스텐 / 하이브리드 중 어떤 걸로 Vault 구성했나 |
| **금기 사항** | AI가 절대 하면 안 되는 행동 (예: 무단 commit, 외부 API 호출) |
| **Skill 분리 후보** | 특정 도메인 워크플로우(예: 슬라이드 제작·노트 임포트)가 별도 Skill로 빠질 가치 있나 |

→ **솔직하게 답하세요.** 답변이 곧 본인의 AGENTS.md 차별화 요소가 됩니다.

#### 2-3. 계획 산출물 확인

Prometheus는 계획을 `.sisyphus/plans/<timestamp>-agents-md.md` 같은 경로에 저장합니다.

```bash
ls -lt .sisyphus/plans/ | head -5
```

> ✅ **성공 기준**: 계획 파일이 생성되고, 두 AGENTS.md의 *섹션 목차 + 명령어 예산 + Skill 분리 후보 + 인터뷰 답변 요약*이 모두 들어 있다. 이 시점에서 본인이 계획을 한 번 읽고 *"이대로 만들어지면 OK"* 라는 판단이 서야 합니다.

---

### Step 3. 계획 실행 — Sisyphus + `ultrawork`로 두 파일 작성 (10분)

계획이 마음에 들면 이제 실행입니다.

#### 3-1. 실행 프롬프트

```
방금 prometheus가 만든 .sisyphus/plans/<파일명>.md 계획을 그대로 실행해줘.

실행 규칙:
- 계획에 적힌 두 파일을 정확히 그 경로에 작성
  · ~/.config/opencode/AGENTS.md (GLOBAL)
  · <내 Vault 절대경로>/AGENTS.md   (Vault)
- 기존 파일이 있으면 백업(.bak-YYYYMMDD)부터 만들고 덮어쓰기
- Skill로 빼기로 한 항목은 ~/.config/opencode/skills/<name>/SKILL.md 로 같이 생성
- 작성 후 두 파일의 명령어 수를 카운트해서 150 이하인지 자체 검증

ultrawork.
```

#### 3-2. 결과 검증

```bash
# 1. 파일이 의도한 위치에 만들어졌는가?
ls -la ~/.config/opencode/AGENTS.md
ls -la "<내 Vault 절대경로>/AGENTS.md"

# 2. 백업 파일이 생성됐는가? (기존 파일이 있었다면)
ls ~/.config/opencode/AGENTS.md.bak-* 2>/dev/null

# 3. 명령어 수가 예산 안에 들어왔는가? (대략적 측정)
grep -cE '^[-*0-9]' ~/.config/opencode/AGENTS.md
grep -cE '^[-*0-9]' "<내 Vault 절대경로>/AGENTS.md"

# 4. omo doctor가 새 GLOBAL AGENTS.md를 인식하는가?
bunx oh-my-opencode doctor
```

#### 3-3. 실제 동작 테스트

OpenCode를 재시작한 뒤, **새 세션**에서:

```
지금 너에게 적용된 GLOBAL AGENTS.md의 핵심 규칙 5개를 한 줄씩 요약해줘.
```

→ Step 2 인터뷰에서 답한 본인의 선호가 반영된 답이 나오면 성공.

같은 방식으로 Vault 안에서 OpenCode를 실행하고 묻기:

```bash
cd <내 Vault>
opencode
```

```
지금 작업 컨텍스트(Vault)의 AGENTS.md 핵심 규칙을 요약해줘.
GLOBAL AGENTS.md와 어떻게 다른가?
```

→ 두 파일의 **역할 분리**가 명확히 드러나면 완료.

---

## 📋 강사의 실제 AGENTS.md 예시

> 본인 결과물이 어떻게 생겼는지 감을 잡고 싶다면 아래 두 예시를 참고하세요.
> **그대로 복사하지 마세요** — Step 2의 *Prometheus 인터뷰* 답변에서 본인의 도메인·선호·Vault 구조에 맞게 도출되어야 합니다. 아래는 *"Red Hat 컨설턴트 + PARA·LLM Wiki 하이브리드 Vault"* 라는 본인 컨텍스트에 맞춰 만들어진 결과입니다.

### 예시 1 — GLOBAL AGENTS.md (`~/.config/opencode/AGENTS.md`)

- **명령어 수**: 약 33개 (예산 150~200 대비 22% 사용 — 매우 슬림)
- **구성 4섹션**: Personal Preferences / Professional Persona / File Safety Rules / AGENTS.md Maintenance
- **역할**: 어디서 `opencode`를 실행해도 항상 적용되는 *"나"* 에 대한 정보

> **🎯 슬라이드 원칙 적용 포인트**
> - **역량 중심** (Slide 4.4): "Red Hat 아키텍트, AI Harness Engineering 주력" — *역할*만 명시, 파일 경로 X
> - **Personal scope** (Slide 2.2): 언어·어조·페르소나 같은 **개인 선호**를 모은 글로벌 레이어
> - **Stale 방지** (Slide 3.4): "AGENTS.md Maintenance" 섹션에서 *"자주 바뀌는 정보(파일 경로) X"* 자기 규제

<details>
<summary>📂 전체 내용 펼치기</summary>

`````markdown
# Global Rules - dangtongbyun

## 1. Personal Preferences

### 1.1 Language
- 설명·주석: 한국어
- 코드·기술 용어·변수명: 영어
- 통용되는 영어 기술 용어는 번역 없이 그대로 (예: deployment, rollback, agent)

### 1.2 Communication Style
- 간결·직설. 핵심 먼저
- 사족 금지: 인사말, 메타 설명, 마무리 멘트
- 불확실성 명시 ("추정", "확실하지 않음")
- 이의 제기 명시 (yes-man 금지)
- 답변 형식: 표·리스트·코드블록

## 2. Professional Persona

### 2.1 Role & Domain
- Role: Red Hat 아키텍트/컨설턴트
- 주력 (AI): AI Harness Engineering, Agentic AI 개발환경 구축, AI/MLOps platform
- 주력 (Platform): Kubernetes/OpenShift, Virtualization migration
- 주력 (Methodology): Personal Knowledge Management
- 인접: DevOps, Cloud Migration, Container Security, Site Reliability

### 2.2 Working Context
- 활동 (Enterprise): 고객 engagement (assessment, PoC, 컨설팅), 기술 발표, 내부 enablement
- 활동 (AI/PKM): AI agent harness 구축·운영, Agentic 개발환경 셋업, PKM 시스템 설계·운영, 기술 문서 작성
- 산출물: 아키텍처 다이어그램, 발표 슬라이드, migration 가이드, technical assessment 보고서, demo 코드, AGENTS.md/skills 정의, 지식 베이스 노트
- 이해관계자: 엔터프라이즈 고객 IT/플랫폼 팀, 내부 SA/CSM/PM, 파트너

### 2.3 Scope (편향 방지)
- 기술·업무 질문 → 이 페르소나를 기본 컨텍스트로 가정
- 개인 학습·취미·일상 질문 → 페르소나 적용 안 함
- 명시적 다른 컨텍스트 제공 시 → 그쪽 우선

## 3. File Safety Rules

### 3.1 Files
- 새 파일 생성·수정 전 승인 필수
- 임의 생성 금지 ("조사 정리", "문서화" 명목 X)
- 사용자 명시 요청 시 즉시 진행 가능
- 예외: `.sisyphus/` 디렉토리 (플래닝 워크플로우)

### 3.2 Git Operations
- commit은 명시 요청 시에만
- commit 전 변경 파일·메시지 제시 → 승인

### 3.3 Secrets
- `.env`, credentials, API keys 등은 commit·로그·응답에 노출 X
- 우연히 발견 시 즉시 사용자에게 알리고 처리 방안 협의

### 3.4 Git Safety
- `git push --force`, hard reset 등 destructive 명령은 명시 요청 시에만
- main/master 브랜치 force push는 거듭 확인

## 4. AGENTS.md Maintenance

- 추가 신중 (instruction budget, ball of mud 방지)
- 자주 바뀌는 정보(파일 경로 등) X — capabilities만 명시
- stale 발견 시 즉시 사용자에게 알리고 정리 협의
`````

</details>

### 예시 2 — Vault AGENTS.md (`<vault>/AGENTS.md`)

- **명령어 수**: 약 50개 (예산 150~200 대비 33% 사용)
- **구성 5섹션**: Vault Overview / Folder Structure / File Conventions / Note Writing Protocol / Course Workflow
- **역할**: PKM 작업에서만 추가로 적용되는 *Vault*의 구조·규약·운영 규칙

> **🎯 슬라이드 원칙 적용 포인트**
> - **Project scope** (Slide 2.2): 이 Vault를 위한 *프로젝트 결정사항* — Personal scope(GLOBAL)와 명확히 분리
> - **Progressive Disclosure** (Slide 4.3): 폴더 맵에 *"`07-Knowledge/` = 자체 AGENTS.md"* 만 표기. 실제 LLM Wiki 운영 규칙은 sub-AGENTS.md에 위임
> - **역량 중심** (Slide 4.4): *"PARA + LLM Wiki 하이브리드"*, *"단방향 publish: Education → Knowledge"* — **개념·규약**으로 기술. 개별 파일 경로 나열 X
> - **Anti-stale** (Slide 3.4): symlink 실제 위치는 *"`readlink`로 확인"* 으로 우회 — 절대 경로를 문서에 박지 않음

<details>
<summary>📂 전체 내용 펼치기</summary>

`````markdown
# Obsidian Vault Rules - dangtong-note

## 1. Vault Overview

### 1.1 Path
- `/Users/dangtongbyun/Obsidian/dangtong-note`

### 1.2 Vault Philosophy
- 구조: PARA + LLM Wiki/Zettelkasten 하이브리드
- `.md` = 정제된 콘텐츠, 원본 = `99-Attachments`
- `04-Resources` 미사용 (정리 대기)
- `07-Knowledge` = LLM Wiki 패턴 (자체 AGENTS.md)
- 일부 폴더는 외부 symlink (cloud sync 또는 로컬 shortcut) — 상세는 3번 섹션

## 2. Folder Structure

### 2.1 Top-level Folder Map


| 폴더              | 용도                                                       |
| ------------------- | ------------------------------------------------------------ |
| `00-System/`      | 시스템 (Dashboard, MOCs, Templates, external symlinks)     |
| `01-Inbox/`       | 임시 수집 (메일·클리핑·raw drop·RSS·미팅 등 다중 채널) |
| `02-Areas/`       | 지속적 관심 영역 (Health, People, Personal, Work)          |
| `03-Projects/`    | 프로젝트 (`Inprogress/{##-name}/`, `Completed/{##-name}/`) |
| `04-Resources/`   | 미사용 (정리 대기)                                         |
| `05-Archive/`     | 비활성 항목                                                |
| `06-Daily/`       | Daily notes (`{YYYY}/{YYYY-MM}/`)                          |
| `07-Knowledge/`   | LLM Wiki 지식 베이스 (자체 AGENTS.md)                      |
| `98-Excalidraw/`  | Excalidraw 그림 파일                                       |
| `99-Attachments/` | 원본 파일 (`linked-files/` symlink)                        |
| `RSS articles/`   | 미사용 (정리 대기)                                         |

### 2.2 Excluded folders (이 AGENTS.md 미적용)
- `00-System/agents/`, `00-System/opencode/`, `99-Attachments/`

### 2.3 Sub-folder with own AGENTS.md
- `07-Knowledge/` (자체 AGENTS.md — vault root 룰과 merge)

### 2.4 Special Notes
- `04-Resources/`, `RSS articles/`: 사용 보류 (정리 대기) — 신규 노트 생성 X
- `02-Areas/Personal/Education/`: 강의 진행 추적 (자유 구조, LLM Wiki 미적용)

## 3. File Conventions

### 3.1 Markdown-only Folders
- `02/03/06/07`은 `.md` 파일만 (정제된 콘텐츠)
- 원본 파일(PDF, 이미지, 영상, 녹음 등)을 vault 내부에 직접 두지 않음

### 3.2 Originals → 99-Attachments
- 모든 원본 파일은 `99-Attachments/linked-files/{type}/` 안에 type별 분류 저장
- vault 내 `.md`에서 wikilink로 참조

### 3.3 External Symlinks


| Vault path                     | 실제 위치                 | 비고                    |
| -------------------------------- | --------------------------- | ------------------------- |
| `00-System/agents/`            | Google Drive (cloud sync) | 다중 기기 공유          |
| `00-System/opencode/`          | 로컬 shortcut             | OpenCode 설정 빠른 접근 |
| `99-Attachments/linked-files/` | Google Drive (cloud sync) | 원본 파일 저장소        |

- 큰 파일 작업 시 cloud sync 영향 인지
- 절대 경로는 stale 위험으로 명시 X — 필요 시 `readlink` 확인

### 3.4 Linked Files Wikilink 패턴

일반 형태: `![[99-Attachments/linked-files/{type}/{subfolder}/{filename}.{ext}]]`

주요 type 카테고리:
- `images/` (PNG, JPG 등)
- `docs/` (PDF, PPTX, DOCX, MD 등)
- `recordings/` (M4A, WAV + transcript MD)
- `video/` (MP4, MOV 등)
- `code/`, `embed/`, `markdown/`, `archive/`

상세 type 가이드: `99-Attachments/linked-files/README.md` 참고

## 4. Note Writing Protocol

### 4.1 작성 절차
- 한 번에 전체 작성 X — 점진적 진행
- 작성시 obsidian skill 사용할것
- 절차:
  1. 목차·아젠다·소제목 구성안 먼저 제시
  2. 사용자 확인·수정 반영
  3. 확정된 구조에 따라 섹션별 작성

### 4.2 저장 위치 제안
- 신규 노트 저장 시: 후보 폴더 3곳 제시 → 1곳 Recommended 명시
- 사용자 선택 후 저장

## 5. Course/Learning Workflow

### 5.1 3종 분산 원칙


| 콘텐츠                                   | 위치                                         | 역할      |
| ------------------------------------------ | ---------------------------------------------- | ----------- |
| 강의 원본 (영상/PDF/슬라이드 등)         | `99-Attachments/linked-files/` (type별 분류) | raw       |
| 강의 진행 활동 (lesson 노트, 진도, 회고) | `02-Areas/Personal/Education/{강의명}/`      | action    |
| 강의에서 추출한 atomic 개념              | `07-Knowledge/{도메인}/{개념}.md`            | knowledge |

### 5.2 02-Areas/Personal/Education 운영
- 일반 PARA Area — 자유 구조
- LLM Wiki 패턴 미적용 (frontmatter schema·concern 분류·atomic notes 강제 X)
- 그냥 working space (lesson 메모, 진도 추적, 회고)

### 5.3 단방향 publish: Education → Knowledge
- Education raw 메모 → `07-Knowledge/`로 publish (기준·형식은 07-Knowledge AGENTS.md 따름)
- 역방향 (Knowledge → Education) X
`````

</details>

### 보너스 — Progressive Disclosure 실전: Sub-AGENTS.md

위 Vault AGENTS.md를 자세히 보면 한 가지 사실이 눈에 띕니다:

```text
07-Knowledge/      ← 자체 AGENTS.md를 가진 폴더 (Vault root 룰과 자동 merge)
```

루트 AGENTS.md에 LLM Wiki 운영 규칙(concern-based 분류·frontmatter schema·8개 가드레일 등)을 **모두 욱여넣지 않고**, *그 폴더에 들어갔을 때만* 추가로 로드되도록 sub-AGENTS.md로 분리한 것입니다 — 이게 곧 슬라이드 6.1의 **Monorepo 자동 병합 패턴**입니다.

| 위치 | 적용 시점 | 명령어 수 |
|---|---|---|
| `~/.config/opencode/AGENTS.md` (GLOBAL) | 모든 OpenCode 세션 | ~33 |
| `<vault>/AGENTS.md` (Vault root) | Vault에서 작업할 때 | ~50 |
| `<vault>/07-Knowledge/AGENTS.md` (Sub) | LLM Wiki 폴더 안에서 작업할 때 | ~40 |

세 파일을 다 쳐도 **합계 약 123개** — Instruction Budget(150~200) 안에서 *컨텍스트별 자동 분할 로딩*으로 매번 LLM에게 필요한 만큼만 전달합니다.

> 💡 **본인에게 주는 질문**:
> 본인 Vault에도 *전혀 다른 워크플로우를 가진 폴더*가 있나요? (예: 강의 노트 / 고객 자료 / 개인 일기 / 코드 프로젝트)
> 있다면 그 폴더 전용 sub-AGENTS.md 분리를 검토하세요. **루트 AGENTS.md를 비대하게 만들기 전에.**

---

## ✅ 완료 체크

- [ ] 강의 PDF가 `agents-md-guide.md`로 변환되어 핵심 24장의 내용이 모두 들어 있다
- [ ] Prometheus가 만든 계획 파일이 `.sisyphus/plans/`에 존재한다
- [ ] `~/.config/opencode/AGENTS.md` (GLOBAL) 작성 완료, 명령어 수 ≤ 150
- [ ] `<vault>/AGENTS.md` (Vault) 작성 완료, 명령어 수 ≤ 150
- [ ] 두 파일의 역할이 명확히 분리됨 (전역 선호 vs Vault 컨텍스트)
- [ ] 새 OpenCode 세션이 GLOBAL AGENTS.md 규칙을 따른다
- [ ] Vault에서 실행한 OpenCode가 Vault AGENTS.md까지 함께 인식한다

**7개 모두 ✅** 면 본인의 AGENTS.md 시스템이 가동된 것입니다. 다음 챕터부터는 이 두 파일이 *모든 작업의 기본값*이 됩니다.

---

## 💡 Aha Moment

> **"AI가 못 알아듣는 건 모델 문제가 아니라 내 AGENTS.md 문제다."**

Instruction Budget을 넘으면 LLM은 후반부 명령을 무시합니다. 내가 *"왜 안 듣지?"* 했던 대부분의 순간은, 사실 **내 지시문이 너무 많았기 때문**입니다. 작게 만들수록 잘 듣습니다.

---

## 🔗 참고 자료

- 원본 글: Matt Pocock, *"A Complete Guide To AGENTS.md"* (2026-05-01)
- AGENTS.md 공식 사이트: <https://agents.md>
- OpenCode Skills 문서: <https://opencode.ai/docs/skills>

---

## ▶ 다음 단계

본인 PKM/코드 프로젝트에 AGENTS.md를 적용했다면, 이제 **omo의 자동화 능력**을 본격적으로 활용할 차례입니다.

**다음 챕터**: [05. Agent Harness ① — MCP · Command · Skill]({{< ref "05-agent-harness-1" >}}) — opencode를 손에 길들이는 *수동 호출 3축*
