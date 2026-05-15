---
title: "07. LLM Wiki"
weight: 7
date: 2026-05-10
draft: false
---

> **⏰ 시간**: 40~45분
> **🎯 목표**: PARA의 Resources가 묘지가 되지 않도록 Knowledge 레이어를 만들고, LLM Wiki 방식으로 첫 번째 갱신 사이클을 경험합니다.

---

## 📄 이론 슬라이드

> 이 챕터의 이론은 아래 슬라이드로 다룹니다.

{{< pdf-iframe src="07-llm-wiki_v1.0.pdf" >}}

---

## 🧭 한 줄 요약

> **R은 입구이고, K는 자산입니다. LLM Wiki는 K를 계속 살아 있게 만드는 운영 방식입니다.**

---

## 📚 학습 목표

이 챕터를 마치면 다음을 할 수 있습니다:

- [ ] PARA의 Resources가 왜 쉽게 묘지가 되는지 설명할 수 있습니다.
- [ ] 기존 Vault 자료를 PARA+K 5계층으로 다시 분류할 수 있습니다.
- [ ] Knowledge 레이어 안에 `raw/`, `wiki/`, `schema/` 3계층을 만들 수 있습니다.
- [ ] LLM에게 `schema/`와 `wiki/`를 보여주고 새 `raw` 자료 반영 초안을 요청할 수 있습니다.
- [ ] LLM의 제안을 그대로 붙여 넣지 않고, 검증 후 Knowledge 페이지에 반영할 수 있습니다.

---

## 🛠️ 사전 요구사항

| 항목 | 요구 사항 |
|------|---------|
| **OpenCode** | Chapter 01에서 설치 완료 |
| **omo (Oh-My-OpenAgent)** | `bunx oh-my-opencode doctor` 모든 항목 ✅ |
| **AGENTS.md** | Chapter 04에서 GLOBAL + Vault AGENTS.md 작성 완료 |
| **MCP 연동** | Chapter 05에서 Exa + Firecrawl MCP 연동 완료 |
| **Web Clipper + Proxy** | Chapter 06에서 LLM proxy + Web Clipper로 캡처 라인 완성 (`_Feed/`로 raw 자료 유입 가능 상태) |
| **Vault** | Chapter 03에서 만든 Vault |

> [!tip] 오늘 필요한 재료
> 완벽한 Vault가 필요하지 않습니다. 대신 **최근에 캡처한 글 1개**, **수업 중 만든 사이클 노트 1개**, **앞으로 반복해서 써먹고 싶은 주제 1개**가 있으면 충분합니다.

---

## 🪜 실습 시나리오

오늘 실습은 “수집한 자료를 지식 자산으로 바꾸는 첫 사이클”입니다. 핵심은 LLM이 알아서 정리하게 만드는 것이 아니라, **내가 검증한 구조를 LLM이 계속 갱신하게 만드는 것**입니다.

결과물은 수강생 Vault 안에 남습니다.

```text
Vault/
├── 00-Inbox/ 또는 _Feed/
├── 10-Projects/
├── 20-Areas/
├── 30-Resources/
├── 40-Knowledge/
│   └── my-domain/
│       ├── raw/
│       ├── wiki/
│       └── schema/
└── .retro/
```

폴더 이름은 여러분의 Vault 규칙에 맞게 바꿔도 됩니다. 중요한 것은 이름이 아니라 **역할 분리**입니다.

---

### Step 0. 사전 점검 — 내 Vault에 재료가 있는가

먼저 오늘 변환할 자료를 고릅니다.

| 확인할 것 | 예시 | 체크 |
|---|---|---|
| 캡처된 원본 자료 | Web Clipper로 저장한 글, PDF 메모, 검색 결과 | [ ] |
| 수업 중 만든 사이클 노트 | Chapter 02에서 만든 조사·정리·결론 노트 | [ ] |
| 반복해서 쓰고 싶은 주제 | 코드 의사결정, 보고서 문장, 리서치 주제 | [ ] |

좋은 주제는 “한 번 보고 끝나는 자료”가 아니라 “앞으로도 계속 다시 쓰일 판단 기준”입니다.

예시는 다음과 같습니다.

- 개발자 민우: `React Server Components 의사결정`, `OpenCode 에이전트 설계`, `API 인증 방식 비교`
- 현업 지은: `분기 보고서 핵심 메시지`, `경쟁사 리서치`, `고객 인터뷰 인사이트`

> [!warning] 주제 선택 기준
> 너무 큰 주제는 피하세요. “AI”보다 “우리 팀 주간보고서에 반복해서 들어가는 고객 VOC 분류 기준”이 좋습니다.

---

### Step A. PARA+K 5계층으로 재분류하기

기존 PARA는 자료를 관리하는 데 강하지만, `Resources`가 오래 쌓이면 다시 보지 않는 창고가 되기 쉽습니다. 그래서 이 강의에서는 PARA에 `Knowledge` 레이어를 하나 더 둡니다.

| 레이어 | 역할 | 오늘의 판단 질문 |
|---|---|---|
| **P — Projects** | 마감과 결과물이 있는 일 | 이 자료가 지금 진행 중인 결과물에 직접 쓰이나? |
| **A — Areas** | 계속 관리해야 하는 책임 영역 | 이 자료가 장기적으로 돌봐야 하는 영역과 연결되나? |
| **R — Resources** | 참고 가능한 원본·자료 | 아직 검증 전이거나, 나중에 볼 수도 있는 자료인가? |
| **A — Archive** | 비활성 자료 | 지금은 쓰지 않지만 보존할 필요가 있나? |
| **K — Knowledge** | 검증을 통과한 재사용 지식 | 다음에도 판단 기준으로 다시 쓸 수 있나? |

다음 순서로 자료를 이동하거나 링크합니다.

1. 원본 캡처 자료는 `Resources` 또는 `_Feed`에 둡니다.
2. 프로젝트 결과물에 바로 쓰이는 노트는 `Projects`에 둡니다.
3. 지속 관리 영역과 연결되는 노트는 `Areas`에 둡니다.
4. 반복해서 재사용할 판단 기준·개념·패턴은 `Knowledge` 후보로 표시합니다.

예시:

```text
30-Resources/web-clips/2026-05-rsc-article.md
10-Projects/frontend-refactor/rsc-decision-note.md
40-Knowledge/frontend-architecture/wiki/react-server-components.md
```

> [!info] 복사보다 링크 우선
> 같은 내용을 여러 곳에 복사하면 금방 어긋납니다. 원본은 R에 두고, K 페이지에서는 원본을 링크하거나 요약해 가져오는 방식을 권장합니다.

---

### Step B. `지식창고`를 LLM Wiki 구조로 만들기

이제 Obsidian 안에 `지식창고` 폴더를 만들고, 여기에 작은 LLM Wiki를 구성합니다. Karpathy식 LLM Wiki의 핵심은 **원본**, **정제된 지식**, **운영 규칙**을 섞지 않는 것입니다.

| 폴더 | 역할 | 사람이 검증해야 할 것 |
|---|---|---|
| `raw/` | 원본 자료, 캡처, 발췌 | 출처와 맥락이 남아 있는가? |
| `wiki/` | 정제된 지식 페이지 | 다음에도 재사용할 만큼 정확한가? |
| `schema/` | LLM이 갱신할 때 지켜야 할 규칙 | 어떤 형식과 기준으로 업데이트할지 명확한가? |

이번 실습에서는 `지식창고` 폴더를 다음 3-layer 구조로 만듭니다.

```text
지식창고/
├── raw/
│   ├── articles/
│   ├── notes/
│   ├── books/
│   └── papers/
├── wiki/
│   ├── entities/
│   ├── concepts/
│   ├── sources/
│   └── synthesis/
└── schema/
```

터미널이나 OpenCode에서 아래처럼 생성할 수 있습니다.

```bash
mkdir -p raw/articles raw/notes raw/books raw/papers
mkdir -p wiki/entities wiki/concepts wiki/sources wiki/synthesis
mkdir -p schema
```

그리고 최상위에는 전체 네비게이션 역할을 하는 `MOC.md`를 만듭니다.

```text
지식창고/
├── MOC.md
├── AGENTS.md
├── raw/
├── wiki/
└── schema/
```

`MOC.md`에는 다음 내용을 포함합니다.

- 이 지식창고의 목적
- `raw/`, `wiki/`, `schema/`의 역할
- 관심 도메인 링크
- wiki 분류 기준
- 다음 작업 큐

> [!tip] MOC는 목차가 아니라 작업대입니다
> MOC는 “어떤 문서가 있는지”만 보여주는 목록이 아닙니다. 지금 이 지식창고가 어떤 기준으로 운영되고, 다음에 무엇을 갱신해야 하는지 알려주는 작업대입니다.

`schema/`에는 LLM에게 줄 운영 지침을 짧게 적습니다.

예시 파일: `schema/update-rule.md`

```markdown
# Frontend Architecture Wiki 운영 규칙

이 위키는 프론트엔드 아키텍처 의사결정을 반복 재사용하기 위한 Knowledge 레이어다.

LLM은 새 raw 자료를 받으면 다음 순서로 갱신한다.

1. 기존 wiki 페이지와 충돌하는 주장 여부를 먼저 확인한다.
2. 새 자료에서 재사용 가능한 판단 기준만 추출한다.
3. 출처가 불명확하거나 검증되지 않은 내용은 "검증 필요"로 표시한다.
4. 최종 반영 전에는 변경 요약과 근거 링크를 제시한다.
5. 사람의 승인 없이 기존 결론을 삭제하지 않는다.
```

> [!tip] schema는 길 필요가 없습니다
> schema의 목적은 멋진 문서가 아니라 **LLM의 행동 경계**를 정하는 것입니다. “무엇을 바꾸면 안 되는가”가 특히 중요합니다.

마지막으로, 모든 wiki 페이지가 같은 형식을 갖도록 `schema/wiki-page-template.md`를 만듭니다.

```yaml
---
title: ""
type: concept
domain: ""
layer: wiki
status: draft
created: YYYY-MM-DD
updated: YYYY-MM-DD
aliases: []
source_notes: []
related: []
tags:
  - llm-wiki
---
```

도메인별 네비게이션은 `schema/<도메인명>-MOC.md`로 둡니다. 예를 들어 관심 도메인이 `AI`, `백앤드개발`, `프런트앤드개발`, `클라우드`라면 다음 파일을 만듭니다.

```text
schema/
├── AI-MOC.md
├── 백앤드개발-MOC.md
├── 프런트앤드개발-MOC.md
└── 클라우드-MOC.md
```

이제 이 폴더에서 LLM이 작업할 때 참고할 `AGENTS.md`도 “관심 도메인 목록”이 아니라 “wiki 운영 규칙”으로 업데이트합니다.

`AGENTS.md`에는 최소한 다음 규칙이 있어야 합니다.

- raw 자료와 wiki 지식을 섞지 않는다.
- 모든 wiki 페이지에는 frontmatter를 둔다.
- LLM은 최종 작성자가 아니라 갱신 보조자다.
- wiki 갱신 요청에는 `schema/`, 기존 `wiki/`, 새 `raw`를 함께 제공한다.
- 출처 없는 내용은 검증된 지식처럼 추가하지 않는다.

---

### Step C. 프롬프트 실습: `지식창고`를 직접 구축하고 자동화하기

이 절은 완성된 결과를 읽는 파트가 아니라, 수강생이 OpenCode에 프롬프트를 넣어 **자기 Vault 안에서 같은 흐름을 재현하는 실습**입니다. 아래 프롬프트를 순서대로 실행하고, 각 단계가 끝날 때마다 생성된 파일을 Obsidian에서 직접 확인합니다.

##### Prompt 1. `지식창고` LLM Wiki 초기화

먼저 OpenCode를 `지식창고`로 사용할 폴더에서 열고 아래 프롬프트를 실행합니다.

```text
옵시디안 내의 ‘지식창고' 폴더를 LLM wiki 개념을 도입해서 구축 및 운영 하려고 한다.
아래 URL 내용 참고해서 LLM wiki 를 위한 초기 디렉토리 구조 및 파일을 생성 해줘.
https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md

- 폴더는 raw, wiki, schema 3-layered 구조로 가자
- wiki 밑에는 entities, concepts, sources, synthesis 디렉토리를 만들자.
- raw 밑에는 articles, notes, books, papers 디렉토리 만들자
- 모든 wiki 페이지에는 관리를 위한 frontmatter 를 포함하자
- 저장된 지식의 네비게이션 역할을 하는 MOC.md 파일을 최상위에 만들고 도메인 지식에 대한 <도메인명>-MOC.md 파일은 schema 디렉토리에 만들자.

그리고 현재 워킹디렉토리에 관심 도메인만 기록해둔 AGENTS.md 파일을 wiki 운영을 위한 용도로 내용을 업데이트 해자.
```

실행 후 확인할 것:

- `raw/`, `wiki/`, `schema/`가 분리되었는가?
- `wiki/entities/`, `wiki/concepts/`, `wiki/sources/`, `wiki/synthesis/`가 생겼는가?
- `MOC.md`와 `schema/<도메인명>-MOC.md`가 네비게이션 역할을 하는가?
- `AGENTS.md`가 단순 관심 도메인 목록이 아니라 wiki 운영 규칙이 되었는가?

> [!note] Karpathy식 LLM Wiki를 Obsidian에 맞게 바꾸기
> 이 실습에서는 `raw/`를 원본 입력, `wiki/`를 검증된 재사용 지식, `schema/`를 LLM이 따라야 할 운영 규칙으로 해석합니다. Karpathy의 예시에는 `index.md`와 `log.md`가 등장하지만, Obsidian에서는 `MOC.md`를 네비게이션 중심으로 쓰고, 변경 이력은 각 wiki 페이지의 `업데이트 로그`에 남겨도 됩니다.

##### Prompt 2. Web Clipper로 raw 자료 넣기

다음으로 Obsidian Web Clipper를 사용해 관심 있는 문서 1~2개를 `raw/articles/`에 저장합니다. 예시는 다음과 같습니다.

```text
Obsidian web clipper 로 엔트리픽, 오픈에이아이 문서 클리핑
```

수업에서는 이 단계의 핵심이 “어떤 문서를 골랐는가”가 아니라, clipping된 문서가 아직 **검증된 wiki 지식이 아니라 raw 입력**이라는 점입니다.

##### Prompt 3. raw ingest용 OpenCode skill 만들기

raw 문서를 매번 수동으로 정리하지 않기 위해 ingest skill을 만듭니다. 이 단계에서는 바로 구현만 시키지 말고, 먼저 Oracle에게 설계 검토를 받게 합니다.

```text
raw 폴더에 있는 문서를 llm wiki 의 ingest 작업으로 처리하기위한 opencode skill 을 만드는 것을 @oracle 에 컨설팅 받아서 만들자

- ingest 시 source 파일 자동 작성
- concept 후보 탐색 후 (추천, 가능, 보류) 및 추천 concept 만 승격
- entity 후보 탐색 및 승격 (추천, 가능, 보류) 및 추천 entity 만 승격
```

skill이 만들어진 뒤에는 사용법을 물어봅니다.

```text
Skill 사용법 알려줘
```

이 skill의 핵심은 “자동 정리”가 아니라 “source card를 만들고, concept/entity 승격 후보를 사람이 검토할 수 있게 만드는 것”입니다.

##### Prompt 4. 회의록 ingest용 OpenCode skill 만들기

이번에는 article/clipping과 별도로, 회의록 전사본을 정리하는 전용 skill을 만듭니다. 이 실습의 목적은 회의 내용을 곧바로 지식으로 승격하는 것이 아니라, `raw/meetings/`의 원본을 `wiki/meetings/`의 **검토 가능한 정리본**으로 바꾸는 흐름을 만드는 것입니다.

실습 전에 `raw/meetings/`에 회의록 샘플 1개를 준비합니다. 예: `raw/meetings/2026-05-16-pkm-llm-wiki-강의설계-회의.md`

아래 프롬프트를 그대로 넣어 실행합니다.

```text
# 작업: 회의록 ingest 스킬 생성

## 배경
- 지식창고 구조: `raw/`(원본), `schema/`(MOC, 규칙), `wiki/`(정리본)
- 기존 스킬 `/llm-wiki-ingest` 가 articles, clippings 를 정리 중
  - 경로: `.config/opencode/skills/llm-wiki-ingest/SKILL.md`
  - 이 스킬의 frontmatter, 위키링크 규칙, 파일명 컨벤션을 그대로 계승할 것
- 회의록은 별도 처리 흐름이 필요해서 새 스킬을 만든다

## 산출물
- **스킬 파일**: `.config/opencode/skills/llm-wiki-ingest-meeting/SKILL.md`
- **커맨드 이름**: `/llm-wiki-ingest-meeting`
- 기존 `/llm-wiki-ingest` 와 동일한 SKILL.md 포맷(YAML frontmatter + 본문)을 따른다

## 스킬의 동작
**입력**: `raw/meetings/` 의 회의록 전사본 (단건 또는 다건)
**출력**: `wiki/meetings/` 에 정리본 생성 (폴더가 없으면 생성)

### 출력 파일명
- 원본 파일명 유지: `raw/meetings/2025-11-15-backend-sync.md` → `wiki/meetings/2025-11-15-backend-sync.md`

### Frontmatter (필수)
```yaml
---
type: meeting
date: YYYY-MM-DD
attendees: [이름1, 이름2]
source: raw/meetings/<원본파일명>
tags: []
---
```

### 본문 섹션 (정확히 이 순서, 이 헤더로)

#### `## Summary`
- 핵심 결정사항과 맥락을 3~7개 불릿으로 요약
- 누가 무엇을 결정했는지가 드러나야 함

#### `## Issues`
- 논의된 안건별로 소제목 `### <안건>`
- 각 안건 아래에 참석자별 입장을 `- **<이름>**: <입장>` 형식으로
- 합의/미합의 여부 명시

#### `## Action Items`
- 항목이 하나라도 있으면 다음 체크박스 형식:
  ```
  - [ ] **<담당자>** — <내용> (마감: YYYY-MM-DD)
  ```
- 마감일이 회의록에 없으면 `(마감: 미정)`
- 액션 아이템이 0개면 섹션 자체를 생략

## 위키 링킹 규칙
- 회의에서 언급된 사람/프로젝트/기술 용어 중 `wiki/entities/`, `wiki/concepts/` 에 존재하는 항목은 `[[이름]]` 으로 링크
- 존재하지 않으면 링크하지 않음 (새 노드를 임의로 만들지 말 것)
- 관련 MOC(`schema/백앤드개발-MOC.md` 등) 가 해당되면 frontmatter `tags` 에 MOC 키 추가

## 처리 정책
- `raw/meetings/` 에 이미 `wiki/meetings/` 동일 파일명이 존재하면 기본은 skip, `--force` 플래그가 있으면 덮어쓰기
- 다건 처리 시 각 파일별로 위 절차 반복, 마지막에 처리 결과 요약 출력 (성공/스킵/실패)

## 참고
- 회의록 원본 예시: `raw/meetings/` 의 파일들
- 한국어 회의록은 한국어로 정리, 위키링크 대상명은 wiki 에 등록된 표기 그대로 사용
```

결과를 볼 때는 다섯 가지만 확인합니다.

- `raw/meetings/` → `wiki/meetings/` 흐름이 분리되었는가?
- 출력 파일명이 원본 파일명을 그대로 유지하는가?
- 정리본이 `Summary`, `Issues`, `Action Items` 순서를 정확히 지키는가?
- 애매한 발언은 `Action Items`가 아니라 `Issues`에 남겼는가?
- 이미 존재하는 정리본에 대해 skip/force 정책이 분명한가?

> [!tip] 이 실습의 핵심 메시지
> 회의록 ingest의 목적은 “회의 내용을 AI가 알아서 결론내리는 것”이 아닙니다. 누가 어떤 입장을 냈는지, 무엇이 합의되었는지, 무엇이 아직 액션으로 확정되지 않았는지를 구분해서 **검토 가능한 정리본**으로 만드는 것이 핵심입니다.

##### Prompt 5. `@wiki/`로 wiki에 질문하기

이제 `wiki/` 폴더가 생겼다면 OpenCode에게 “이 위키 안에서 이미 검증된 페이지를 우선 참고해서 답하라”는 방식으로 질문할 수 있습니다. 여기서 `@wiki/`는 마법 명령이 아니라, 현재 작업 폴더의 `wiki/` 경로를 컨텍스트로 붙여 주는 표기라고 이해하면 됩니다.

```text
@wiki/ 이 위키에서 React Server Components 판단 기준을 5줄로 요약해줘.
근거로 참고한 wiki 페이지 이름도 함께 적어줘.
```

질문 예시는 다음처럼 조금씩 바꿔 쓸 수 있습니다.

- `@wiki/ raw와 wiki를 구분하는 운영 규칙을 설명해줘. 관련된 source page도 같이 알려줘.`
- `@wiki/ 이 주제와 연결된 concept page, source page, 검증 필요 항목을 찾아줘.`
- `@wiki/ 내가 방금 읽은 raw 문서를 어느 wiki 페이지와 연결해서 검토하면 좋을지 추천해줘.`

답을 받을 때는 세 가지만 확인합니다.

- 답변이 `raw/`가 아니라 `wiki/` 기준으로 정리되었는가?
- 어떤 wiki 페이지나 source page를 참고했는지 이름을 함께 적었는가?
- 중요한 판단이라면, 인용된 페이지를 내가 직접 열어 다시 검증했는가?

> [!warning] `@wiki/`를 이해하는 방식
> `@wiki/`는 Obsidian의 wikilink 문법이 아닙니다. 이 실습에서는 “`wiki/` 폴더를 먼저 보고 답해 달라”는 컨텍스트 힌트로 사용합니다. 중요한 답변은 최종 결론으로 복사하지 말고, 인용된 wiki/source 페이지를 직접 열어 검증한 뒤 반영하세요.

##### Prompt 6. ingest 완료 체크박스 속성 추가

ingest가 끝난 raw 문서를 다시 처리하지 않도록 frontmatter에 완료 상태를 남깁니다.

```text
raw 파일에서 ingest 완료시 frontematter 에 ingest 완료 여부를 체크하는 체크박스로 추가하자.

- AGENTS.md 에도 반영하자
- Skill 에도 반영 하자
```

완료 표시 예시는 다음과 같습니다.

```yaml
ingest_completed: true
```

이 값은 “검증된 wiki 지식으로 승격 완료”가 아니라, **해당 raw 문서에 대한 ingest 처리가 한 번 완료되었다**는 운영 메타데이터입니다.

##### Prompt 7. frontmatter 없는 raw 문서를 위한 tag skill 만들기

마지막으로 ingest보다 앞단에 둘 tag skill을 만듭니다. 학생들은 아래처럼 짧게 요청하면 됩니다.

```text
raw 문서에 태그를 붙이는 OpenCode skill을 만들자.

상황:
- raw 폴더에 Web Clipper나 복사/붙여넣기로 만든 문서가 많이 쌓인다.
- 이 파일들은 frontmatter가 없거나 tags가 비어 있다.

목적:
- 각 raw 파일이 무엇에 관한 문서인지 판단해서 frontmatter에 tags를 추가한다.
- 태그를 정할 때는 현재 Vault에서 이미 쓰고 있는 태그 목록을 먼저 참고한다.
- 신규 태그가 필요 하다고 판단 되면 신규 태그를 작성한다.

중요 원칙:
- raw 파일은 wiki로 승격하지 않는다.
- 파일 이동, 삭제, 이름 변경은 하지 않는다.
- 본문은 수정하지 않고 frontmatter만 수정한다.
- 새 태그가 필요하면 바로 만들지 말고 후보로 보고한다.
- skill 에 대한 내용을 AGENTS.md 에도 업데이트 한다.
```

결과를 볼 때는 네 가지만 확인합니다.

- raw 본문을 건드리지 않았는가?
- frontmatter의 `tags:`만 추가하거나 보강했는가?
- 기존 Vault 태그를 우선 재사용했는가?
- 새 태그 후보를 따로 보고했는가?

##### Prompt 8. LLM Wiki 건강검진 lint skill 만들기

마지막으로 `지식창고`가 커졌을 때 주기적으로 상태를 점검할 lint skill을 만듭니다. v1은 욕심내지 않고 **구조적 문제를 보고하는 건강검진 리포트**로 제한합니다. semantic 모순 검출이나 자동 수정은 나중 단계로 미룹니다.

```text
LLM Wiki의 건강검진을 위한 OpenCode skill을 만들자.
skill 이름은 llm-wiki-lint 로 하자.

상황:
- 이 Vault는 raw, wiki, schema, AGENTS.md 구조로 운영된다.
- raw는 원본 자료이므로 절대 수정하면 안 된다.
- wiki는 검증 후 승격된 노트다.
- schema와 AGENTS.md는 LLM이 따라야 할 운영 규칙이다.
- MOC.md 또는 _index.md는 wiki 네비게이션 역할을 한다.

v1 lint 규칙:
1. broken-wikilink — [[target]] 파일이 없는 경우
2. orphan-page — inbound link가 0인 wiki 페이지
3. missing-index-entry — MOC.md 또는 _index.md에 없는 wiki 페이지
4. stale-feed — 30일 이상 미정리된 raw 또는 _Feed 파일
5. missing-frontmatter — 필수 frontmatter 필드 누락
6. invalid-domain — 허용된 domain 값 밖의 값
7. duplicate-title — 동일 title 또는 alias가 여러 페이지에 존재
8. broken-attachment-link — 99-Attachments/linked-files/ 링크가 실제로 없는 경우

v1 범위 밖:
- semantic 모순 검출은 하지 않는다.
- stale claim 검출은 하지 않는다.
- claim의 사실 여부를 판단하지 않는다.
- 자동 수정은 하지 않는다. 보고만 한다.

안전 규칙:
- raw 폴더는 절대 수정하지 않는다.
- 삭제, 이동, 이름 변경은 하지 않는다.
- broken link는 고치지 말고 후보 파일만 제안한다.
- orphan 노트는 연결하지 말고 연결 후보만 제안한다.
- index/MOC 누락도 직접 고치지 말고 추가 후보만 제안한다.

결과 리포트에는 다음 섹션을 포함하자.
1. Summary
2. Rule Results
3. Broken Wikilinks
4. Orphan Pages
5. Missing Index / MOC Entries
6. Stale Feed / Raw Files
7. Frontmatter / Domain Issues
8. Duplicate Titles / Aliases
9. Broken Attachment Links
10. Safety Check

전역 OpenCode skill과 Vault-local skill 양쪽에 만들어줘.
```

결과를 볼 때는 다섯 가지만 확인합니다.

- `llm-wiki-lint` skill이 생성되었는가?
- `raw/` 수정 금지 규칙이 명확한가?
- v1 범위가 구조적 lint로 제한되어 있는가?
- broken link와 orphan은 수정이 아니라 후보 제안으로 처리되는가?
- semantic 모순 검출, stale claim 검출, 자동 수정이 v1 범위 밖으로 잠겨 있는가?

---

### Step D. 사람이 검증하고 반영하기

이 강의의 핵심 메시지는 “AI 속도”가 아니라 **검증 대역폭**입니다. LLM이 초안을 빠르게 만들 수는 있지만, 무엇을 K에 올릴지는 사람이 결정해야 합니다.

반영 전 마지막으로 세 가지만 확인합니다.

- 이 내용은 원본 자료에 근거가 있는가?
- 다음에도 다시 쓸 판단 기준인가?
- 지금 내 프로젝트나 책임 영역과 연결되는가?

세 질문에 모두 “예”라고 답할 수 있으면 `wiki/` 페이지에 반영합니다. 하나라도 애매하면 `raw/`에 남겨두고 “검증 필요” 표시를 합니다.

추천 반영 형식:

```markdown
## 업데이트 로그

### 2026-05-15

- 추가: 서버/클라이언트 경계 판단 기준 보강
- 근거: [[../../raw/articles/2026-05-new-rsc-case]]
- 검증: 프로젝트 A 적용 여부는 아직 검토 필요
```

---

### Step E. 회고 남기기

마지막으로 M1 회고를 남깁니다. 이 회고는 나중에 PKM 평가 챕터에서 다시 사용합니다.

파일 예시: `.retro/M1.md`

```markdown
# M1 회고 — 검증 대역폭과 LLM Wiki

## 오늘 만든 K 레이어

- 도메인: frontend-architecture
- raw 자료: 1개
- wiki 페이지: 1개
- schema 규칙: 1개

## 한 줄 회고

내 R이 묘지였다면, K 레이어는 검증된 자료만 다시 살아나는 작업대다.
```

> [!question] 스스로에게 묻기
> “오늘 만든 wiki 페이지를 다음 주에도 다시 열어볼 이유가 있는가?”
>
> 없다면 아직 K가 아니라 R입니다.

---

## ✅ 완료 체크

- [ ] 오늘 다룰 도메인 1개를 선택했습니다.
- [ ] 원본 자료 1개를 `raw/` 또는 Resources 위치에 보관했습니다.
- [ ] Knowledge 레이어에 도메인 폴더를 만들었습니다.
- [ ] 도메인 폴더 안에 `raw/`, `wiki/`, `schema/` 역할을 구분했습니다.
- [ ] `schema/`에 LLM Wiki 운영 규칙과 도메인 MOC를 만들었습니다.
- [ ] 프롬프트 실습을 통해 raw ingest용 skill을 만들었습니다.
- [ ] 회의록 전용 ingest skill이 `raw/meetings/`를 `wiki/meetings/` 정리본으로 바꾸는지 확인했습니다.
- [ ] `@wiki/`로 wiki 기준 질문 1개를 하고, 인용된 페이지를 직접 확인했습니다.
- [ ] raw ingest 완료 표시(`ingest_completed`) 규칙을 추가했습니다.
- [ ] tag skill의 목적과 성공 기준을 정리했습니다.
- [ ] lint skill의 목적과 안전한 자동 수정 기준을 정리했습니다.
- [ ] LLM이 만든 결과를 검증한 뒤 반영하거나 보류했습니다.
- [ ] `.retro/M1.md`에 한 줄 회고를 남겼습니다.

---

## 💡 Aha Moment

> **LLM Wiki는 “AI가 대신 기억하는 시스템”이 아닙니다. 내가 검증한 지식 구조를 LLM이 계속 갱신하도록 만드는 운영 방식입니다.**

자료를 많이 모으는 것만으로는 PKM이 좋아지지 않습니다. 품질은 결국 내가 무엇을 K로 승격시켰는지, 그리고 그 승격 기준을 얼마나 일관되게 유지했는지에 달려 있습니다.

---

## 🔗 참고 자료

- Karpathy LLM Wiki gist: <https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f>
- Tiago Forte, Building a Second Brain / PARA Method
- Chapter 02. Methodology — 자유 주제 PKM 사이클
- Chapter 03. Vault Structure — Vault 계층과 폴더 설계
- Chapter 06. Web Clipper — raw 자료 유입 파이프라인

---

## ▶ 다음 단계

이제 여러분의 Vault에는 “그냥 저장한 자료”와 “검증 후 재사용할 지식”이 분리되어 있습니다.

다음 단계에서는 이 K 레이어를 더 많이 자동화할 수 있습니다.

- MCP를 통해 LLM이 Vault와 외부 자료를 함께 참고하게 만들기
- Web Clipper로 들어온 raw 자료를 K 후보로 분류하기
- 에이전트에게 `schema/`를 읽히고 wiki 갱신 초안을 반복 생성하게 만들기

**다음 챕터**: 준비 중
