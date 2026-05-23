---
title: "05. Agent Harness ① — MCP · Command · Skill"
weight: 5
date: 2026-05-22
draft: false
---

> **⏰ 시간**: TODO (예상 ~1시간 = 이론 20분 + 실습 40분)
> **🎯 목표**: TODO

---

## 📄 이론 슬라이드

> 이 챕터의 이론(MCP 개념·아키텍처·왜 필요한가 등)은 아래 슬라이드로 다룹니다.

{{< pdf-iframe src="05-mcp-integration_v1.0.pdf" >}}

---

## 🧭 한 줄 요약

> TODO

---

## 📚 학습 목표

이 챕터를 마치면 다음을 할 수 있습니다:

- [ ] **MCP(Model Context Protocol) 개념**을 한 문장으로 설명할 수 있다
- [ ] **Exa MCP**를 OpenCode에 연동하고 검색 명령을 실행할 수 있다
- [ ] **Firecrawl MCP** (community)를 OpenCode에 연동하고 웹 페이지를 스크랩할 수 있다
- [ ] **Penpot MCP**(원격 MCP 모드, SaaS)를 OpenCode에 연동하고 디자인 파일을 조회할 수 있다
- [ ] AGENTS.md에 **MCP 사용 정책**을 명시할 수 있다 (옵션)

---

## 🛠️ 사전 요구사항

| 항목 | 요구 사항 |
|------|---------|
| **OpenCode** | Chapter 01에서 설치 완료 |
| **omo (Oh-My-OpenAgent)** | `bunx oh-my-opencode doctor` 모든 항목 ✅ |
| **AGENTS.md** | Chapter 04에서 GLOBAL + Vault AGENTS.md 작성 완료 |
| **Vault** | Chapter 03에서 만든 Vault |
| **Exa** | 강의 기본은 *API key 없이* 시작 (Remote 무료 모드, 150 calls/day). 대용량 사용 시 <https://dashboard.exa.ai/api-keys>에서 키 발급 |
| **Firecrawl** | Self-host docker compose (강의 기본). **Docker Desktop 실행 필수** + 디스크 여유 5GB 이상. 강의 전 `git clone` + `docker compose build` 사전 권장 |
| **Docker** | Docker Desktop 설치·실행 (<https://www.docker.com/products/docker-desktop>) — Firecrawl self-host 위해 필수 |
| **Penpot 계정** | <https://design.penpot.app> 무료 회원가입 (Professional 플랜 — 사용자 8명·무제한 프로젝트·10GB) |

---

## 🪜 실습 시나리오

<!-- TODO: 실습 단계 작성 -->

### Step 1. Exa MCP 연동 (10분)

> **목표**: AI가 *직접 웹을 검색해 본문까지 통합 반환*하는 첫 경험. Exa는 *API key 없이도 즉시 시작*할 수 있는 원격 MCP를 제공하므로, 계정 가입·키 발급 절차 없이 opencode.json 3줄로 끝낸다.

#### 1-1. Exa 모드 선택 — 왜 Remote 무료부터 시작하는가

Exa MCP는 두 모드를 제공합니다.

| 모드 | URL / 명령 | 인증 | 제한 |
|---|---|---|---|
| **Remote (무료)** | `https://mcp.exa.ai/mcp` | 불필요 | 150 calls/day · 3 QPS |
| **Remote (유료)** | 같은 URL + `x-api-key` 헤더 | API key | 플랜 따름 |
| Local stdio | `npx -y exa-mcp-server` | `EXA_API_KEY` 필수 | 플랜 따름 |

본 강의는 **Remote 무료 모드**로 진입합니다. 하루 150회는 강의 실습은 물론 일상 PKM 사이클에도 충분합니다. 더 큰 사용량이 필요해지면 *나중에* API key를 추가합니다 (1-5 참고).

#### 1-2. opencode.json에 Exa 등록 (2분)

`~/.config/opencode/opencode.json`(또는 프로젝트 루트의 `opencode.json`)을 열어 `mcp` 섹션에 Exa를 추가합니다.

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "exa": {
      "type": "remote",
      "url": "https://mcp.exa.ai/mcp",
      "enabled": true
    }
  }
}
```

핵심 포인트:

- `type: "remote"` — Exa가 호스팅하는 원격 MCP를 사용 (Penpot과 같은 패턴)
- `url`은 `https://mcp.exa.ai/mcp` — *고정 endpoint*
- `headers`·`Authorization` 모두 불필요 (무료 모드는 익명 접근)

#### 1-3. OpenCode 재시작 → 등록 확인 (1분)

설정 변경 후 OpenCode를 재시작합니다.

```bash
# 실행 중이면 종료 후
opencode
```

TUI에서 MCP 연결 상태를 확인:

```
/mcp
```

`exa`가 *connected* 상태로 보여야 합니다. 노출된 tool 목록에 `web_search_exa`·`web_fetch_exa`가 있으면 OK.

> **💡 Tool 노출 정책**: 슬라이드 S3에서 다룬 *"Server는 Tools를 노출한다"*가 여기서 구체화됩니다. Exa는 기본으로 `web_search_exa`(검색)과 `web_fetch_exa`(URL 본문 추출) 두 개를 노출합니다.

#### 1-4. 첫 호출 — 자료 5개를 Vault INBOX로 (5분)

OpenCode TUI에서 다음을 실행:

```
"개인지식관리(PKM) 2026 트렌드"에 대한 최신 글 5개를 Exa로 찾고,
각 글의 제목·URL·발행일·핵심 요지(3줄)를 정리해서
~/AI-Knowledge/INBOX/pkm-trends-2026.md 에 저장해줘.

요구사항:
- H1: PKM 2026 트렌드 — 자료 5개
- H2: 글 제목
- bullet: URL / 발행일 / 핵심 요지 3줄
- 마지막에 "수집일자: <오늘 날짜>" 1줄

exa MCP의 web_search_exa를 사용해.
```

OpenCode가 다음 흐름으로 동작합니다:

1. `exa` MCP의 `web_search_exa` tool 호출 → 검색 결과 + 본문 통합 반환
2. AI가 받은 JSON을 markdown으로 정리
3. `INBOX/pkm-trends-2026.md` 파일로 저장 — **여기서 승인 게이트가 한 번 떠야 함**
4. 승인하면 Vault에 파일이 생성됨

> **✨ Aha moment**: ChatGPT라면 *수강생이 직접 검색 → 본문 복사 → 채팅창에 붙여넣기 → 결과 복사 → Vault에 붙여넣기* 4단계였던 것이, 한 번의 의도 표현과 한 번의 승인으로 끝났다. 슬라이드 S1의 *"복붙이 사라진다"*가 손에 잡혔다.

#### 1-5. (옵션) API key를 나중에 추가하는 법

하루 150회를 초과하거나 더 안정적인 성능이 필요하면 API key를 추가합니다.

1. <https://dashboard.exa.ai/api-keys> 로그인 (Google SSO 가능)
2. **Create Key** 버튼 → 키 이름 입력 → 생성된 Secret Key를 복사 (1회만 표시)
3. 환경변수 등록:

   ```bash
   # ~/.zshrc 또는 ~/.bashrc
   export EXA_API_KEY="여기에_복사한_키"
   ```

4. opencode.json에 `headers` 추가:

   ```json
   {
     "mcp": {
       "exa": {
         "type": "remote",
         "url": "https://mcp.exa.ai/mcp",
         "headers": {
           "x-api-key": "{env:EXA_API_KEY}"
         },
         "enabled": true
       }
     }
   }
   ```

5. OpenCode 재시작 → `/mcp`로 재연결 확인

무료 플랜은 월 1,000 요청. 가격은 <https://exa.ai/pricing> 참조.

### Step 2. Firecrawl MCP 연동 (15분)

> **목표**: 슬라이드 S4의 *데이터 주권 스펙트럼*을 손으로 체험. Firecrawl을 **자체 호스팅(docker compose)** 으로 띄우고, MCP 서버가 *내 노트북의 컨테이너*를 가리키게 한다. Exa·Penpot의 SaaS와 달리 *외부 클라우드 거치지 않는* 크롤링 라인이 생긴다.

#### 2-1. Firecrawl Self-host 모드 — 왜 docker compose인가

Firecrawl은 *Cloud SaaS*와 *Self-host* 두 모드를 제공합니다.

| 모드 | 인프라 | 인증 | 강의 사용 |
|---|---|---|---|
| **Self-host (docker)** | 내 노트북 컨테이너 4개 | API key 불필요 (인증 OFF) | ✅ 본 강의 기본 |
| Cloud SaaS | firecrawl.dev | API key 필수 | 참고 (트러블슈팅) |

본 강의는 ★ 셋 페어링(Exa SaaS · **Firecrawl Self-host** · Penpot SaaS)의 *데이터 주권 스펙트럼*을 신체로 체험하기 위해 **self-host**를 기본으로 진입합니다. 컨테이너 4개(`api` · `playwright-service` · `redis` · `nuq-postgres`)가 같이 뜨므로, *Docker Desktop이 실행 중*이어야 합니다.

> **💡 강사 사전 안내**: 첫 `docker compose build`가 5~10분 걸립니다 (이미지 다운로드 + 빌드). 강의 *시작 전*에 git clone + build를 미리 돌려두면 라이브에서는 `up`만 하면 됩니다.

#### 2-2. Firecrawl 본체 기동 (10분, 첫 1회만)

```bash
# 작업 디렉토리 (Vault 외부 권장)
mkdir -p ~/AI-Workstation/firecrawl-selfhost
cd ~/AI-Workstation/firecrawl-selfhost

# 공식 저장소 클론
git clone https://github.com/firecrawl/firecrawl.git
cd firecrawl

# 최소 .env 생성 (인증 OFF — 강의용)
cat > .env << 'EOF'
PORT=3002
HOST=0.0.0.0
USE_DB_AUTHENTICATION=false
BULL_AUTH_KEY=CHANGEME
EOF

# 빌드 + 기동 (백그라운드)
docker compose up -d --build
```

기동 확인:

```bash
# 컨테이너 4개가 모두 healthy/running이어야 함
docker compose ps

# health check
curl http://localhost:3002/test
# OK 응답이 오면 성공
```

> **💡 Self-host 메시지**: 슬라이드 S4가 *"Self-host = 내 노트북에서 처리"* 라 말한 것이 손에 잡힙니다. `docker compose ps`로 보이는 4개 컨테이너가 *전부 내 머신 안*에 있고, 크롤링하는 모든 페이지가 *외부 클라우드를 거치지 않습니다*.

#### 2-3. Firecrawl MCP 서버 등록 (2분)

Firecrawl 본체와 *별개로* MCP 서버(npm 패키지 `firecrawl-mcp`)가 있습니다. 이 패키지가 OpenCode와 self-host 백엔드 사이의 *어댑터*입니다.

`opencode.json`의 `mcp` 섹션에 추가:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "firecrawl": {
      "type": "local",
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_URL": "http://localhost:3002"
      }
    }
  }
}
```

핵심 포인트:

- `type: "local"` — Exa·Penpot의 `"remote"` 와 다름. *npm 패키지를 OpenCode가 직접 실행*하는 stdio 모드
- `FIRECRAWL_API_URL` — *내 노트북의 Firecrawl 본체*를 가리킴. SaaS를 쓴다면 이 값을 비우고 `FIRECRAWL_API_KEY`를 채움
- API key 환경변수 불필요 — `.env`에서 `USE_DB_AUTHENTICATION=false`로 인증을 껐기 때문
- 슬라이드 S3의 *"Server는 로컬 프로세스로 실행되거나 원격 URL로 연결된다, 같은 표준 다른 배포"* 가 여기서 구체화됨 (Penpot=remote URL · Firecrawl=local 프로세스 + self-host backend)

#### 2-4. OpenCode 재시작 → 등록 확인 (1분)

```bash
opencode
```

TUI에서:

```
/mcp
```

`firecrawl`이 *connected*로 보이고, tool 목록에 `firecrawl_scrape`·`firecrawl_search`·`firecrawl_map`·`firecrawl_crawl`이 있으면 OK.

#### 2-5. 첫 호출 — URL 1개를 Vault INBOX로 (2분)

OpenCode TUI에서:

```
https://docs.firecrawl.dev/introduction 의 본문을
markdown으로 가져와서
~/AI-Knowledge/INBOX/firecrawl-intro.md 에 저장해줘.

요구사항:
- onlyMainContent: true (네비게이션·푸터 제외)
- 원문 markdown 구조 유지
- frontmatter에 source URL과 캡처일자 추가

firecrawl MCP의 firecrawl_scrape를 사용해.
```

OpenCode가 다음 흐름으로 동작합니다:

1. `firecrawl` MCP의 `firecrawl_scrape` tool 호출
   - 인자: `url: "https://docs.firecrawl.dev/introduction"`, `formats: ["markdown"]`, `onlyMainContent: true`
2. *내 노트북의* Firecrawl 컨테이너가 페이지를 가져와 markdown으로 변환
3. AI가 결과를 받아 frontmatter 추가 후 파일로 저장 — **승인 게이트 1회**
4. 승인하면 Vault에 파일 생성

> **✨ Aha moment**: Exa가 *검색+본문*을 통합 반환했다면, Firecrawl은 *특정 URL의 본문만* 정확히 가져옵니다. 그리고 그 모든 처리가 *내 노트북 안의 컨테이너에서* 일어났습니다 — 외부 클라우드를 거친 페이지는 단 한 줄도 없습니다. 슬라이드 S4의 *"데이터 주권 스펙트럼"*이 손에 잡혔습니다.

#### 2-6. 트러블슈팅 핵심 3가지

| 증상 | 원인 | 해결 |
|---|---|---|
| `docker compose up`이 즉시 종료 | `.env` 누락 또는 포트 충돌 | `docker compose logs` 확인. 포트 3002가 이미 사용 중이면 `.env`의 `PORT=3003`으로 변경 후 `FIRECRAWL_API_URL`도 같이 조정 |
| MCP가 `connected`인데 호출 실패 | Firecrawl 본체가 죽은 상태 | `docker compose ps`로 4개 모두 running인지 확인. `curl localhost:3002/test`로 health check |
| `Unauthorized` 에러 | `.env`의 `USE_DB_AUTHENTICATION=false` 누락 | `.env` 재확인 후 `docker compose restart api` |

종료 / 재기동:

```bash
# 종료
cd ~/AI-Workstation/firecrawl-selfhost/firecrawl
docker compose down

# 다시 기동 (.env·이미지 유지된 상태)
docker compose up -d
```

### Step 3. Penpot MCP 연동 (10분)

> **목표**: 디자인 도구도 AI가 *직접 읽는다*. Penpot Cloud(SaaS) 무료 플랜 + 원격 MCP 모드로 *npm install · docker 없이* MCP 키 발급만으로 연결한다.

#### 3-1. MCP 키 발급 (5분)

Penpot은 자체 호스팅 *원격 MCP 서버*를 제공합니다. 사용자는 키만 받으면 됩니다.

1. <https://design.penpot.app> 로그인
2. 우측 상단 프로필 → **Your account**
3. 좌측 메뉴 **Integrations** → **MCP Server** 섹션
4. **Status** 토글을 ON으로
5. 표시되는 모달에서 **Generate MCP key** 버튼 클릭
6. **MCP KEY GENERATED** 모달이 뜨면 키를 즉시 복사해 안전한 곳에 보관

> **⚠️ "This unique MCP key is non-recoverable."** — 모달 상단 경고 그대로입니다. 모달을 닫으면 다시 볼 수 없으며, 잃어버리면 *Regenerate*로 새 키를 발급해야 하고 기존 키는 즉시 폐기됩니다.

발급된 키는 JWT 형태(약 400~600자), 만료일 없음 (`The MCP key has no expiration date`):

```
eyJhbGciOiJBMjU2S1ciLCJlbmMiOiJBMjU2R0NNIn0.sDQyQU...x77UxgWGSTQ
```

모달 하단에는 `~/.mcp.json` 표준 형식의 설정 예시도 같이 제공됩니다 (Claude Desktop 등 *MCP 표준 client* 용):

```json
{
  "mcpServers": {
    "penpot": {
      "url": "https://design.penpot.app/mcp/stream?userToken=YOUR_MCP_KEY"
    }
  }
}
```

본 강의는 OpenCode를 쓰므로 위 형식은 *참고*하고, 다음 단계(3-3)에서 OpenCode 전용 형식으로 변환해 등록합니다.

#### 3-2. 환경변수 등록 (1분)

키를 config 파일에 직접 박지 않고 환경변수로 분리합니다. (실수로 git commit 되는 사고 방지)

```bash
# ~/.zshrc 또는 ~/.bashrc에 추가
export PENPOT_MCP_KEY="eyJhbGciOiJBMjU2S1ciLCJlbmMi...여기에_복사한_키"
```

설정 후 적용:

```bash
source ~/.zshrc
echo $PENPOT_MCP_KEY | head -c 30
# eyJhbGciOiJBMjU2S1ciLCJlbmMi... 가 나오면 OK
```

#### 3-3. opencode.json에 등록 (2분)

Penpot 모달이 제시한 `~/.mcp.json` 형식은 MCP *표준 client*(Claude Desktop 등) 용입니다. OpenCode는 자체 schema를 따르므로 *형식 변환*이 필요합니다.

| Penpot 모달 (표준) | OpenCode (자체 schema) |
|---|---|
| `mcpServers.{name}.url` | `mcp.{name}.url` + `type: "remote"` |

`~/.config/opencode/opencode.json`(또는 프로젝트 루트의 `opencode.json`)을 열어 `mcp` 섹션에 추가합니다.

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "penpot": {
      "type": "remote",
      "url": "https://design.penpot.app/mcp/stream?userToken={env:PENPOT_MCP_KEY}",
      "enabled": true
    }
  }
}
```

핵심 포인트:

- `type: "remote"` — Penpot이 호스팅하는 원격 MCP 서버를 가리킴 (OpenCode 전용 필드, 표준 `mcp.json`에는 없음)
- `url`의 `{env:PENPOT_MCP_KEY}` — 3-2에서 등록한 환경변수를 OpenCode가 치환. *Penpot 모달은 키를 URL에 직접 박은 예시를 보여주지만*, 환경변수 분리가 안전
- `headers`나 `Authorization` 불필요 — userToken이 query string에 포함됨

이미 Step 1에서 Exa를 등록했다면 `mcp` 객체 안에 `penpot` 키만 *추가*하면 됩니다.

#### 3-4. 강의용 샘플 디자인 파일 준비 (1분)

Penpot 사용 경험이 없는 수강생을 위해 강사가 공유 링크를 제공합니다.

1. 강사가 공유한 Penpot 파일 링크 열기 (강의 자료의 링크 참고)
2. 우측 상단 **Duplicate** 버튼 → 본인 워크스페이스로 복제
3. 복제된 파일의 **이름과 URL의 file ID**를 메모해둡니다 (다음 단계에서 AI에게 알려줘야 함)

> **💡 본인 Penpot 파일이 있다면** 그걸 써도 됩니다. *파일 1개의 컴포넌트 목록 조회* 수준이라 무료 플랜으로 충분합니다.

#### 3-5. 첫 호출 — 컴포넌트 목록을 Vault로 (1분)

OpenCode TUI에서 다음을 실행:

```
방금 내 Penpot 워크스페이스에 복제된 "<파일 이름>"의 컴포넌트 목록을
~/AI-Knowledge/INBOX/penpot-<파일 이름>.md 에 정리해줘.

요구사항:
- H2: 페이지별로 묶기
- H3: 컴포넌트 이름
- bullet: 컴포넌트의 주요 속성 (있다면)
- 마지막에 "총 N개" 요약

penpot MCP를 사용해.
```

OpenCode가 다음 흐름으로 동작합니다:

1. `penpot` MCP의 `high_level_overview` tool 호출 → 파일 구조 반환
2. AI가 받은 JSON을 markdown으로 정리
3. `INBOX/penpot-...md` 파일로 저장 — **여기서 승인 게이트가 한 번 떠야 함**
4. 승인하면 Vault에 파일이 생성됨

> **✨ Aha moment**: 디자인 도구도 *AI가 직접 읽는다.* Figma 캡처를 복붙해 ChatGPT에 넣던 마찰이 사라졌다. 같은 패턴으로 자기 보고서·기획 문서의 시각자료를 Vault 노트로 *자동 정리*할 수 있다.

#### 3-6. 옵션 — Self-host 진입로 (참고용)

본 강의는 SaaS 원격 MCP로 진입하지만, *데이터를 외부로 보내고 싶지 않은* 경우 Penpot 전체를 docker-compose로 자체 호스팅할 수 있습니다.

```
https://help.penpot.app/technical-guide/getting-started/#start-penpot
```

자체 호스팅 후에도 같은 절차로 MCP 키를 발급받아 사용합니다. URL의 `design.penpot.app` 부분만 본인 도메인으로 바꾸면 됩니다.

### Step 4. AGENTS.md에 MCP 사용 정책 추가 (5분, 옵션)

<!-- TODO:
- 04에서 만든 GLOBAL/Vault AGENTS.md에 MCP 관련 1~2줄 정책 추가
- 예: "외부 검색은 exa MCP 우선", "URL 본문 fetch은 firecrawl MCP 사용", "디자인 파일 조회는 penpot MCP 사용"
- ball-of-mud 방지를 위해 짧게
-->

TODO

---

## ✅ 완료 체크

- [ ] `opencode.json`의 MCP 섹션에 Exa·Firecrawl·Penpot 세 개 모두 등록됨
- [ ] OpenCode TUI에서 Exa로 외부 검색 1회 성공
- [ ] Firecrawl self-host 컨테이너 4개 모두 healthy (`docker compose ps`) + `curl localhost:3002/test` OK
- [ ] OpenCode TUI에서 Firecrawl로 URL 스크랩 1회 성공 → Vault에 markdown 저장됨
- [ ] OpenCode TUI에서 Penpot으로 디자인 파일 컴포넌트 목록 조회 1회 성공 → Vault에 `.md`로 저장됨
- [ ] (옵션) AGENTS.md에 MCP 사용 정책 1줄 이상 추가됨

---

## 💡 Aha Moment

> TODO

---

## 🔗 참고 자료

- MCP 공식: <https://modelcontextprotocol.io>
- Exa MCP: <https://github.com/exa-labs/exa-mcp-server>
- Firecrawl 본체 (self-host 가이드): <https://docs.firecrawl.dev/contributing/self-host>
- Firecrawl GitHub: <https://github.com/firecrawl/firecrawl>
- Firecrawl MCP 서버 (npm `firecrawl-mcp`): <https://github.com/firecrawl/firecrawl-mcp-server>
- Penpot MCP 공식 가이드: <https://help.penpot.app/mcp/>
- Penpot + OpenCode 6단계 튜토리얼: <https://penpot.app/blog/set-up-penpot-mcp-with-openrouter-and-opencode-in-6-steps/>
- Penpot 가격(무료 플랜 포함): <https://penpot.app/pricing>
- OpenCode MCP 문서: <https://opencode.ai/docs/mcp>

---

---

# Part B — Command (28분)

> Part A에서 *외부 자료를 vault로 끌어왔다*면, Part B는 **opencode를 *내 손가락에 박제*** 합니다. 매일 하는 반복 동작을 *슬래시 명령*으로 만들어, 두세 단어로 호출하게 됩니다.
>
> 단, 모든 걸 Command로 만들 필요는 없습니다. **Obsidian이 할 수 있는 일은 Obsidian에게**. Command는 *AI가 도와줄 영역*만 담당합니다.

## 🧭 Part B 한 줄 요약

> **"Daily Note 빌드는 Obsidian, 항목 추가·정제는 Command. 도구가 단순할 수 있으면 단순한 도구로."**

## 📚 Part B 학습 목표

이 Part를 마치면 다음을 할 수 있습니다:

- [ ] **L1** — Obsidian Templates Core + Daily Notes Core를 활성화·설정하고, 단축키로 *매일 1초*에 오늘 노트 열기
- [ ] **L2** — opencode Command 파일을 *frontmatter + 프롬프트* 구조로 작성
- [ ] **L3** — `/todo-add`·`/todo-list`·`/todo-done`으로 Daily Note todo 관리, `/meeting-clean`으로 회의록 정제
- [ ] **L4** — *시장 통념* (회의록 처리 = Skill)을 *우리 맥락*에서 비판적으로 수용 (Command 선택)
- [ ] **L5** — *그냥 요약* vs `/meeting-clean` 결과 차이를 *자기 눈으로* 보고, "검증 대역폭 = 맥락 보호" 인식

## ⏰ 시간 분배 (30분)

| 단계 | 활동 | 시간 |
|---|---|---|
| 도입 박스 | Command란? Obsidian과의 역할 분리 | 1분 |
| **Step 6** | Obsidian Templates·Daily Notes Core + `/todo-{add,list,done}` 3 Command | **17분** |
| **Step 7** | `/meeting-clean` Command | **13분** |
| 마무리 | 검증 대역폭 회고 + 다음 챕터 예고 | 1분 |

## 💡 도입 — Command란?

opencode의 *Command* = **마크다운 파일 1장이 곧 슬래시 명령**입니다.

```yaml
---
description: 명령의 설명 (TUI에 보임)
agent: build
---

여기에 *AI에게 시킬 프롬프트*를 작성.
$ARGUMENTS는 사용자가 명령 뒤에 입력한 내용.
```

위 파일을 `~/.config/opencode/command/my-cmd.md`로 저장 → `/my-cmd` 슬래시 명령이 됩니다.

**핵심**: Command = *반복 동작의 명시화*. 매번 같은 프롬프트를 복붙하지 말고 *이름표*를 붙입니다.

> ⚠️ **모든 걸 Command로 만들지 마세요.** Obsidian이 *기본*으로 하는 일(파일 생성·이름 짓기·템플릿 적용)은 Obsidian에게 맡깁니다. Command는 *AI가 도와줄 영역*에만.

---

## Step 6 — Obsidian Daily Notes + `/todo-{add,list,done}` (17분)

> 매일 아침의 첫 동작을 *2개 도구의 협업*으로:
> 1. **Obsidian Daily Notes Core** — 오늘 노트 *자동 생성* (단축키 한 번)
> 2. **`/todo-add`, `/todo-list`, `/todo-done` Command 세트** — 그 노트의 *todo 관리* (AI가 형식 정리)
>
> Obsidian이 *구조 만들기*, Command 3개가 *항목 관리*. 같은 도메인이지만 *역할이 다릅니다*. 이것이 "과한 추상화 회피"의 첫 예시.

> [!info] **📍 Step 6~9 공통 — 터미널·vault 경로 약속**
>
> 이 챕터의 모든 터미널 명령은 **macOS / Windows WSL2 Ubuntu** 두 환경에서 *동일하게* 작동합니다.
>
> | 항목 | macOS | Windows |
> |---|---|---|
> | 터미널 | iTerm2 / Warp / 기본 터미널 | **Windows Terminal → Ubuntu 탭** (WSL2) |
> | 셸 | zsh / bash | bash (WSL Ubuntu 안) |
> | 프롬프트 표기 | `$` | `$` (= `ubuntu@...:~$`) |
> | vault 경로 | `~/Obsidian/dangtong-lecture` | `~/Obsidian/dangtong-lecture` (WSL 홈) |
>
> **본 챕터의 모든 `$` 명령은 *플랫폼 무관* 그대로 입력하면 됩니다.** PowerShell은 사용하지 않습니다.
>
> 시작 전 **vault 경로 환경변수**를 한 번 설정해두세요 (이후 모든 Step에서 사용):
>
> ```bash
> export VAULT_ROOT=~/Obsidian/dangtong-lecture
> echo 'export VAULT_ROOT=~/Obsidian/dangtong-lecture' >> ~/.bashrc  # zsh이면 ~/.zshrc
> ```
>
> ✅ **검증**: `echo $VAULT_ROOT` → 자기 vault 경로 출력.
>
> > 🪟 **Windows 학생 — WSL이 아직 없다면?** [01. 환경 설치]({{< ref "01-env-setup" >}})의 WSL2 설치 단계를 먼저 마치세요. *PowerShell native에서 이미 opencode를 깔았다면*, WSL Ubuntu 안에 opencode를 한 번 더 설치해 환경을 일원화하는 것을 권장합니다 — PowerShell쪽과 WSL쪽 opencode는 서로 다른 설치이며 설정·세션을 공유하지 않습니다.

### 6-A. Templates Core 활성화 (1분)

Obsidian의 *기본 내장* 플러그인. 별도 설치 불필요.

1. Obsidian 좌측 하단 톱니바퀴(⚙️) 클릭 → **설정**
2. 왼쪽 패널에서 **코어 플러그인** 클릭
3. **Templates** 항목을 찾아 우측 토글을 **ON**으로
4. 켜진 후, 왼쪽 패널에 **Templates** 항목이 *새로* 나타남 → 클릭
5. **템플릿 폴더 위치** 입력란에 다음 입력:

   ```
   00-System/Templates
   ```

✅ **검증**: 입력란 아래에 *"폴더가 존재합니다"* 메시지가 보여야 합니다 (또는 자동완성이 작동).

### 6-B. Daily Notes Core 활성화 (1분)

이것도 *기본 내장* 플러그인. **Templates와 *별개*** 입니다 (둘 다 켜야 함).

1. **설정 → 코어 플러그인** → **Daily Notes** 토글 **ON**
2. 왼쪽 패널의 **Daily Notes** 클릭
3. 다음 항목 설정:

   | 항목 | 값 |
   |---|---|
   | 새 파일 위치 | `06-Daily/{{date:YYYY}}/{{date:YYYY-MM}}` |
   | 날짜 형식 | `YYYY-MM-DD` |
   | 템플릿 파일 위치 | `00-System/Templates/Daily-Simple` |
   | 시작 시 오늘 노트 열기 | ☐ (선택, 끄는 게 깔끔) |

💡 **{{date:YYYY-MM}} 변수**: Obsidian이 *오늘 날짜를 자동 대입*합니다. 예: 2026-05-22에는 `06-Daily/2026/2026-05/` 폴더에 노트가 생성.

⚠️ **템플릿 파일 경로의 `.md` 생략**: Obsidian이 *알아서* 붙입니다.

### 6-C. Daily Note 템플릿 생성 (3분)

이제 Daily Notes Core가 사용할 *템플릿 파일*을 만듭니다.

1. `00-System/Templates/` 폴더에서 **새 파일** 생성
2. 파일명: `Daily-Simple.md`
3. 다음 내용 복사·붙여넣기:

```markdown
---
date: {{date:YYYY-MM-DD}}
type: daily
tags:
  - daily
---

# {{date:YYYY-MM-DD}} ({{date:dddd}})

## 📋 Todo

## 📝 메모

## 💡 배운 것

## 🎯 회고
```

4. 저장 (`Cmd+S`)

💡 **`{{date:dddd}}` 변수**: 요일 이름 (예: "Friday"). 영어로 출력됩니다. 한국어 요일을 원하면 `{{date:ddd}}` (예: "금") 또는 수동 작성.

✅ **검증**: 템플릿 파일이 `00-System/Templates/Daily-Simple.md` 경로에 저장되었는지 확인.

### 6-D. 단축키 설정 + 첫 노트 생성 (2분)

매일 1초로 오늘 노트를 여는 단축키.

1. **설정 → 단축키** (Hotkeys)
2. 상단 검색창에 **`daily`** 입력
3. **Daily Notes: Open today's daily note** 항목을 찾기
4. 우측 `+` 버튼 클릭 → 다음 키 입력:

   - **macOS**: `Cmd+Shift+D`
   - **Windows/Linux**: `Ctrl+Shift+D`

5. 설정 창 닫기

이제 **`Cmd+Shift+D`** 를 눌러보세요.

⭐ **Aha 모먼트**: 오늘 Daily Note가 *자동 생성*되어 열립니다. 폴더는 *알아서* `06-Daily/{YYYY}/{YYYY-MM}/{YYYY-MM-DD}.md` 경로에 만들어졌고, 템플릿이 *적용된* 상태입니다.

✅ **검증**:
- [ ] 단축키로 오늘 Daily Note가 열림
- [ ] 파일 경로가 `06-Daily/2026/2026-05/2026-05-22.md` 형태
- [ ] frontmatter에 오늘 날짜·`type: daily`·`tags: daily` 자동 채워짐
- [ ] `## 📋 Todo`·`## 📝 메모`·`## 💡 배운 것`·`## 🎯 회고` 4섹션 존재

> 💡 **여기까지가 *Obsidian 영역***. 외부 AI 도구 없이 Obsidian만으로 *Daily Note 빌드 시스템*을 완성했습니다. 이제 *Command*가 *그 위에 한 줄을 얹습니다*.

### 6-E. `/todo-add` Command 작성 (3분)

이제 opencode의 슬래시 명령 *3개*를 *세트로* 만듭니다. 첫 번째는 **항목 추가**.

1. 터미널에서 다음 디렉토리 확인 또는 생성:

   ```bash
   mkdir -p ~/.config/opencode/command
   ```

2. 새 파일 작성: `~/.config/opencode/command/todo-add.md`

   다음 내용 *그대로 복사·붙여넣기*:

   ```markdown
   ---
   description: 오늘 Daily Note의 ## 📋 Todo 섹션에 항목 추가
   agent: build
   ---

   오늘 날짜 기반 Daily Note (`06-Daily/{YYYY}/{YYYY-MM}/{YYYY-MM-DD}.md`)에서
   `## 📋 Todo` 섹션을 찾아 다음 항목을 *마지막에* 추가해줘:

   - [ ] $ARGUMENTS

   규칙:
   - 오늘 Daily Note가 *없으면* "`Cmd+Shift+D`로 먼저 오늘 Daily Note를 만드세요" 안내
   - `## 📋 Todo` 섹션이 *없으면* 섹션을 생성 후 추가
   - 항목 내용은 *간결한 동사형*으로 정리 (입력이 길면 핵심만)
   - 파일 수정 *직전* 사용자에게 변경 사항을 보여주고 *승인* 요청

   🌐 응답은 한국어로.
   ```

3. 저장.

💡 **이 Command는 *쓰기 + 승인 게이트* 패턴**입니다. `/daily-retro`(이전 학습 시나리오)와 같은 표준 패턴.

💡 **`$ARGUMENTS` 변수**: 명령 뒤에 입력한 내용을 *그대로* 받습니다. `/todo-add PKM 강의 본문 작성` → `$ARGUMENTS = "PKM 강의 본문 작성"`.

### 6-F. `/todo-list` Command 작성 (2분)

두 번째 — **읽기 전용** Command. *파일 수정 없음, 승인 게이트 불필요*.

1. 새 파일: `~/.config/opencode/command/todo-list.md`

   ```markdown
   ---
   description: 오늘 Daily Note의 todo 리스트를 번호 매겨서 표시 (파일 수정 없음)
   agent: build
   ---

   오늘 Daily Note의 `## 📋 Todo` 섹션을 *읽기만* 하고
   다음 형식으로 출력해줘:

   ```
   1. [ ] 첫 번째 할 일
   2. [x] 완료된 할 일
   3. [ ] 세 번째 할 일

   완료: 1 / 3
   ```

   규칙:
   - 파일을 *읽기만*. 수정 X. 승인 게이트 불필요.
   - 번호는 1부터 시작. *체크박스 상태와 무관* — 완료된 것도 번호 가짐.
   - 오늘 Daily Note 없으면 "오늘 Daily Note가 없습니다. `Cmd+Shift+D`로 먼저 생성하세요." 안내
   - `## 📋 Todo` 섹션은 있는데 항목이 없으면 "오늘 할 일이 없습니다." 안내

   🌐 응답은 한국어로.
   ```

2. 저장.

💡 **이 Command는 *읽기 전용* 패턴**입니다. AI가 *파일을 수정 안 함* → 승인 게이트 없음. 빠르게 결과 보여주는 용도.

> ⚠️ **번호 매기기 — 전체 순번 기준**: 체크박스(`[ ]` 또는 `[x]`)와 *무관*하게 1부터 순서대로. `/todo-done`에서 사용할 번호와 *동일*.

### 6-G. `/todo-done` Command 작성 (3분)

세 번째 — **상태 변경** Command. 인자로 *번호*를 받아 해당 항목 완료 체크.

1. 새 파일: `~/.config/opencode/command/todo-done.md`

   ```markdown
   ---
   description: 오늘 todo 리스트의 N번 항목을 완료 체크. /todo-list로 번호 확인 후 사용.
   agent: build
   ---

   오늘 Daily Note의 `## 📋 Todo` 섹션에서
   *$ARGUMENTS 번째* todo 항목의 `[ ]`를 `[x]`로 변경해줘.

   순서:
   1. `## 📋 Todo` 섹션의 todo 항목들을 *순서대로* 찾기 (1부터 시작, 체크박스 상태 무관)
   2. $ARGUMENTS 번 항목 위치 확인
   3. 해당 줄의 `- [ ]`를 `- [x]`로 교체
   4. (선택) 완료 시각 추가: `- [x] 항목 내용 ✓ {HH:MM}`
   5. 파일 수정 *직전* 사용자에게 변경 사항을 보여주고 *승인* 요청

   규칙:
   - $ARGUMENTS가 범위 밖이면 "N번 항목이 없습니다. /todo-list로 확인하세요." 안내
   - 이미 `[x]`로 완료된 항목이면 "이미 완료됐습니다." 안내
   - 번호는 `/todo-list` 출력의 번호와 *동일*

   🌐 응답은 한국어로.
   ```

2. 저장.

💡 **이 Command는 *상태 변경 + 인자 파싱* 패턴**입니다. `$ARGUMENTS`로 *번호*를 받아 해당 위치를 정확히 찾아 수정.

> 🎯 **3 Command 비교** — 같은 도메인(todo)이지만 *각자 다른 패턴*:
>
> | Command | 패턴 | 승인 게이트 |
> |---|---|---|
> | `/todo-add` | 쓰기 + 추가 | ✅ |
> | `/todo-list` | 읽기 전용 | ❌ (불필요) |
> | `/todo-done` | 상태 변경 + 인자 파싱 | ✅ |
>
> → **Command의 다양성** — 같은 도메인을 *3가지 다른 방식*으로 다룹니다.

### 6-H. 실행 + 검증 (2분)

이제 opencode TUI를 *새 세션*으로 열어 3 Command 모두 작동 확인.

1. 터미널에서 vault 루트로 이동:

   ```bash
   cd $VAULT_ROOT
   # = cd ~/Obsidian/dangtong-lecture (도입부 박스에서 설정한 값)
   ```

2. **opencode** 실행

3. 슬래시 입력: `/`

   → 명령 목록에 **`/todo-add`, `/todo-list`, `/todo-done`** *3개 모두* 보여야 합니다.

4. **첫 항목 추가** (2~3개):

   ```
   /todo-add PKM 강의 Chapter-05 본문 작성
   /todo-add 카탈로그 갱신
   /todo-add 회의록 정제 테스트
   ```

   각 호출마다 AI가 *변경 미리보기* 제안 → 승인.

5. **리스트 확인**:

   ```
   /todo-list
   ```

   → 화면에 번호 매겨진 출력:

   ```
   1. [ ] PKM 강의 Chapter-05 본문 작성
   2. [ ] 카탈로그 갱신
   3. [ ] 회의록 정제 테스트

   완료: 0 / 3
   ```

6. **2번 항목 완료**:

   ```
   /todo-done 2
   ```

   AI가 변경 미리보기 → 승인 → 파일 수정.

7. **다시 리스트 확인**:

   ```
   /todo-list
   ```

   ```
   1. [ ] PKM 강의 Chapter-05 본문 작성
   2. [x] 카탈로그 갱신 ✓ 21:45
   3. [ ] 회의록 정제 테스트

   완료: 1 / 3
   ```

⭐ **Aha 모먼트**: 단순한 todo 관리가 *3 Command의 협업*으로 작동합니다. 각 Command가 *자기 책임만* — 작은 책임, 명확한 의도.

✅ **검증**:
- [ ] TUI 자동완성에 `/todo-add`, `/todo-list`, `/todo-done` *3개 모두* 표시
- [ ] `/todo-add` 호출 시 *변경 미리보기 + 승인 게이트* 작동
- [ ] `/todo-list` 호출 시 *번호 매겨진 출력* (파일 수정 없음)
- [ ] `/todo-done <번호>` 호출 시 해당 항목만 `[x]` 변환
- [ ] Daily Note 파일의 `## 📋 Todo` 섹션이 *순서대로* 갱신됨

> 💡 **자기 시도**: 오늘 *진짜 할 일* 3~4개를 `/todo-add`로 추가하고, 하루 동안 `/todo-done`으로 체크해보세요.

### 6-I. 1줄 회고 (1분)

오늘 Daily Note의 `## 💡 배운 것` 섹션에 다음 1줄을 추가해보세요:

> 이전에 todo를 *어디다 적었는가*? Obsidian + `/todo-{add,list,done}` 3 Command 조합과의 차이는?  
> *역할 분리* (Obsidian = 구조, Command = 동작 / 한 도메인, 3가지 패턴)가 매일 워크플로우에 어떤 변화를 줄까?

> 📌 **다음 Step 예고**: 단순 동사들(`/todo-add`, `/todo-list`, `/todo-done`)은 *각각 프롬프트 1장*으로 충분했습니다. 하지만 *복잡한 정제*(욕설·없는 사람 비난 제거 등)도 Command로 가능할까요? Step 7에서 *결정적 사례*를 봅니다.

---

## Step 7 — `/meeting-clean` Command (13분)

> 회의록을 그냥 *"요약해줘"* 했을 때 vs `/meeting-clean`으로 정제했을 때의 **극적 대비**를 직접 봅니다.
>
> 이 Step의 *진짜 학습*은 Command 작성이 아닙니다. **"AI 정제는 *내용 축소*가 아니라 *맥락 보호*"** 라는 인식. 특히 *회의에 없는 사람*에 대한 발언을 어떻게 다루느냐가 핵심.

> 💡 **도입 박스 — 시장 통념과 우리의 다른 결정**
>
> 회의록 처리 도구를 GitHub에서 찾아보면 **5개 중 5개 모두 Skill로 만듭니다** (Otter.ai, Fireflies 등). 자동 매칭·schema·체이닝 가치가 있기 때문.
>
> 그런데 우리는 *Command로* 만듭니다. 왜?
>
> - 우리 강의 시점엔 LLM Wiki 구조 미설치 → schema 의존이 *부담*
> - 회의록 정리는 *내가 의도적으로* 호출 → 자동 매칭 가치 ↓
> - 단순 transcript 1종 → 입력 분기 불필요
> - **학습 적합성 우선** — 프롬프트 1장이면 충분
>
> → 메타 메시지: **"시장 통념을 *맥락 따라* 비판적으로 수용. 검증 대역폭은 도구 선택에도 적용된다."**

### 7-A. 사전 점검 (2분)

이번 Step에서 *입력*으로 쓸 회의록 샘플을 확인합니다.

1. 아래 토글에서 **회의록 전체 내용을 복사**한 뒤, vault의 *프로젝트 폴더*에 새 파일로 저장하세요.

   먼저 실습용 프로젝트 폴더 생성:

   ```bash
   mkdir -p $VAULT_ROOT/02_프로젝트/flowboard-kanban
   ```

   그 안에 파일 생성: `$VAULT_ROOT/02_프로젝트/flowboard-kanban/sample-meeting-kanban-kickoff.md`

   > 💡 **이 폴더 이름은 임의**입니다. *자기 진행 프로젝트*가 있으면 그 폴더에 저장해도 됩니다. 핵심은 *vault 안의 어딘가*에 .md 파일로 두는 것.

   <details>
   <summary>📂 회의록 전체 내용 펼치기 (252줄, 97개 발언, 약 35분 회의) — 클릭하면 복사용 텍스트가 펼쳐집니다</summary>

   `````markdown
   ---
   title: "Flowboard Kanban — MVP 기획 회의"
   date: 2026-05-22
   time: "14:00-14:40"
   duration_minutes: 40
   type: planning-meeting
   project: Flowboard
   sprint: pre-sprint-1
   attendees:
     - 강주연 (PM, 기획)
     - 김민호 (Frontend Lead)
     - 이지은 (Backend Lead)
     - 박서준 (Product Designer)
   location: 회의실 B / Zoom 하이브리드
   recorded_by: Zoom auto-transcript + 강주연 수동 정리
   source: zoom-transcript
   status: raw
   tags:
     - meeting
     - raw
     - kanban
     - flowboard
     - mvp-planning
   ---

   # Flowboard Kanban — MVP 기획 회의

   > 회의 transcript (raw). Zoom 자동 transcript를 강주연이 화자별로 정리한 1차본.
   > 노이즈·잡담·말줄임 포함. `/meeting-clean`으로 정제 예정.

   ---

   **14:00 강주연**: 자, 다들 들어오셨네요. 박서준 님은 사무실에서, 김민호 님이랑 이지은 님은 재택이시고. 시작할게요.

   **14:01 김민호**: 어 들리세요? 마이크 좀 이상한 것 같은데.

   **14:01 강주연**: 잘 들려요. 일단 오늘 안건 공유한 거 보셨죠? 세 가지인데요. MVP 범위 확정, 컬럼 구조, 그리고 1주차 작업 분배. 다 끝내면 40분 안에 끝낼 수 있을 거예요.

   **14:02 이지은**: 점심 뭐 드셨어요? 저는 아까 그 새로 생긴 베트남 쌀국수 갔는데 진짜 별로였어요.

   **14:02 박서준**: 아 거기 저도 가봤는데 첫 주에는 괜찮았는데 요즘 별로래요.

   **14:02 강주연**: 자자, 일단 회의부터요. 점심 얘기는 끝나고. 첫 번째 안건, MVP 범위인데요. 지난주에 제가 정리한 feature 리스트 다섯 개 보셨죠? 보드 CRUD, 카드 드래그앤드롭, 댓글, 알림, 그리고 통계. 이 다섯 개 중에 *MVP에 뭘 넣고 뭘 뺄지* 오늘 결정해야 해요.

   **14:03 김민호**: 다섯 개 다 한 달 안에는 절대 못 해요. **솔직히 X같은 일정이에요**, 이거.

   **14:03 이지은**: 백엔드도 마찬가지예요. 알림이랑 통계는 별도 서비스 만들어야 하거든요. 알림은 푸시 인프라가 또 따로 필요하고요.

   **14:04 강주연**: 그래서 제 제안은 — 보드 CRUD, 카드 드래그앤드롭, 댓글까지 *세 개*만 MVP. 알림이랑 통계는 v2로 미루기.

   **14:04 박서준**: 댓글까지는 디자인 시안도 가능할 것 같아요. 알림은 어차피 모바일 푸시 디자인까지 가야 하는 거라서 *별도 작업*이 맞고요.

   **14:04 김민호**: 좋아요. 저도 그게 맞다고 봐요. 댓글은 카드 안에 인라인 댓글 형태로? 아니면 모달?

   **14:05 박서준**: 인라인 펼치기로 갈게요. 카드 클릭하면 우측에 슬라이드 패널 열리고 거기에 댓글 영역.

   **14:05 이지은**: API 측에선 어차피 comments 테이블 따로니까 인라인이든 모달이든 상관없어요.

   **14:06 강주연**: 좋아요. *MVP = 보드 CRUD + 카드 드래그앤드롭 + 댓글*. 확정. 알림이랑 통계는 백로그로 빼둘게요.

   **14:06 김민호**: 잠깐, 댓글에 *@mention*도 들어가요?

   **14:07 강주연**: 음... 그건 알림이랑 묶이는 거잖아요. mention하면 알림 가는 게 정상인데, 알림 안 만들면 *mention만 텍스트로* 남는 거고.

   **14:07 이지은**: 그럼 v1에선 mention 자체는 *파싱만* 하고, 알림은 v2 들어올 때 연결하는 식으로?

   **14:08 박서준**: 그렇게 하면 텍스트에 `@강주연` 같은 형태로 나타나는데 클릭하면 아무것도 안 일어나는 거잖아요. 그건 좀 이상한데요.

   **14:08 김민호**: 그럼 v1에선 mention 빼고, 그냥 plain text 댓글만.

   **14:08 강주연**: 그게 깔끔하겠네요. 댓글 = plain text. mention은 v2로.

   **14:09 이지은**: 동의요.

   **14:09 강주연**: 근데 이거 마케팅팀에 보고 들어가면 또 뭐라고 할지 모르겠네요. **마케팅팀 김부장이 진짜 매번 발목 잡아요. 그 분 의사결정이 매번 그래요. 능력이 안 되는 건지 일부러 그러는 건지...**

   **14:10 김민호**: ㅋㅋ 또 시작이세요. 일단 우리끼리 정하고 보고는 나중에 하시죠.

   **14:10 강주연**: 알았어요. 두 번째 안건. 컬럼 구조. 지금 후보가 두 가지인데. 하나는 *고정 3개* (To Do / In Progress / Done). 다른 하나는 *사용자가 컬럼 추가·삭제 가능*. 뭐로 갈까요?

   **14:10 김민호**: 고정 3개로 가요. 동적 컬럼 하려면 board schema가 훨씬 복잡해져요. drag-drop 로직도 컬럼 추가/삭제 케이스 다 핸들해야 하고.

   **14:10 이지은**: 백엔드도 그래요. column entity 따로 만들어야 하고 ordering 관리해야 하고. MVP엔 무리.

   **14:11 박서준**: 디자인 측면에선 동적 컬럼이 좀 더 흥미롭긴 한데... 사용자가 *3개로 충분한가*는 의문이에요. 실제 칸반 보드 쓰는 사람들 보면 *5~7개* 컬럼 쓰는 경우도 많거든요.

   **14:11 강주연**: 음. 박서준 님 말도 맞는데... 일단 MVP는 *최소한*으로 가는 게 맞지 않나요? 사용자 피드백 받고 v2에서 동적으로 확장하면 될 것 같은데.

   **14:12 박서준**: 그건 그래요. 일단 3개로 출시하고 피드백 보죠.

   **14:12 김민호**: 근데 이름은 *고정*인가요? "To Do / In Progress / Done"으로?

   **14:12 강주연**: 영어로 갈지 한국어로 갈지...

   **14:13 박서준**: 영어로 갑시다. 보통 칸반 보드 자체가 영어 용어가 더 익숙해요. 우리 타겟 사용자가 *IT 팀*이니까.

   **14:13 이지은**: 동의.

   **14:13 김민호**: 동의.

   **14:13 강주연**: *고정 3개 컬럼: To Do / In Progress / Done*. 확정. 

   **14:14 이지은**: 어 잠깐. 누가 들어오셨어요. (다른 사람 화면 켜짐)

   **14:14 강주연**: 아 박서준 님 인턴 분 들어오셨네. 회의 끝나고 따로 인사하기로 하고, 일단 계속할게요.

   **14:15 강주연**: 세 번째 안건. 1주차 작업 분배. 다음 주 월요일부터 첫 스프린트 시작하는데, 1주차에 *각자 뭐 할지* 정해야 해요.

   **14:15 박서준**: 디자인은 *시안 v1*까지요. 메인 보드 화면, 카드 컴포넌트, 댓글 패널. 화면 3개.

   **14:16 강주연**: 언제까지 가능?

   **14:16 박서준**: 다음 주 금요일, 5월 29일까지요. Figma에 올릴게요.

   **14:16 김민호**: 프론트는 *카드 컴포넌트 prototype*요. 일단 디자인 안 나와도 placeholder로 시작할 수 있으니까. 박서준 님 시안 나오는 대로 스타일 입히고.

   **14:17 강주연**: 카드 prototype은 어디까지?

   **14:17 김민호**: 카드 자체 렌더링, 드래그 시 그림자 효과, 컬럼 간 이동까지. *5월 29일까지* 가능할 것 같아요. **죽을 것 같지만요**.

   **14:17 이지은**: 백엔드는 *Board/Card API spec 문서*. 엔드포인트 정의, request/response schema. 코드 작성은 그 다음 주.

   **14:18 강주연**: spec 문서는 *언제까지*?

   **14:18 이지은**: 화요일이면 충분해요. 5월 26일까지. 

   **14:18 강주연**: 좋아요. 그리고 제가 하나 더 하기로 한 게 있는데... 그게 뭐였더라.

   **14:19 김민호**: 실시간 동기화 부분 아니에요? 그게 정해져야 API 설계도 영향 가는데.

   **14:19 강주연**: 아 맞다. 그거. 실시간 동기화. WebSocket으로 갈지 polling으로 갈지.

   **14:19 이지은**: 그거 진짜 중요한 결정이에요. 백엔드 architecture가 완전 달라져요. WebSocket 가면 서버 상태 관리 따로 해야 하고, scaling도 까다롭고. polling은 단순하지만 *실시간성*이 떨어지죠.

   **14:20 김민호**: 사용자가 *드래그앤드롭*하면 다른 사람한테 *실시간으로* 보여야 하는 건데. polling으로 가면 1~2초 latency 있을 거고. 그게 UX적으로 괜찮을지 모르겠어요.

   **14:20 박서준**: 사용자 입장에선 *드래그가 부드럽게 동기화*되는 게 멋있긴 한데... 솔직히 *2초 latency*도 받아들일 수 있어요. 동시 편집 충돌 아니면.

   **14:21 이지은**: 동시 편집 충돌은 어떻게 처리할 거예요? 예를 들어 A가 카드를 "In Progress"로 옮기는 동시에 B가 같은 카드를 "Done"으로 옮기면? **이거 안 다루면 진짜 어이없어요**. 사용자가 작업 다 날아가는 거잖아요.

   **14:21 강주연**: 음... 그건 *last-write-wins*로 갈지 *충돌 알림*으로 갈지...

   **14:22 김민호**: MVP에선 last-write-wins로 가는 게 깔끔해요. 충돌 알림은 *UX 복잡도 폭증*이라.

   **14:22 이지은**: 그럼 polling으로 갈 만도 한데. 어차피 last-write-wins면 *정확한 실시간성*이 핵심은 아니니까.

   **14:23 강주연**: 잠깐, 이거 *오늘 결정 안 해도* 될 것 같은데. 일단 *내가 다음 주 수요일까지 조사*해서 옵션 정리해 올게요. WebSocket vs polling 트레이드오프, 우리 상황에 맞는 추천. 그거 보고 결정하죠.

   **14:23 이지은**: 좋아요. 그 결정 나기 전까지는 API spec에서 *동기화 부분*은 placeholder로 두고 시작할게요. 

   **14:23 이지은**: 아 근데 인프라 얘기 나온 김에. **그 협력업체 다시 부르지 마요. 지난 프로젝트 그쪽 때문에 다 망했잖아요. 진짜 무능했어요.** 약속한 시점에 한 번도 제대로 납기 못 맞췄고, 코드 품질도 솔직히... 아무튼 다시는 안 돼요.

   **14:24 강주연**: 그 얘기는 인프라 결정할 때 다시 하시죠. 일단 회의 진행할게요.

   **14:24 김민호**: OK. 프론트도 그쪽 인터페이스만 추상화해 두고 갈게요.

   **14:24 강주연**: 그럼 정리하면 — *1주차 작업*:
   박서준 님: 디자인 시안 v1, 5/29까지.
   김민호 님: 카드 컴포넌트 prototype, 5/29까지.
   이지은 님: Board/Card API spec, 5/26까지.
   저: 실시간 동기화 옵션 조사, 5/27까지.

   **14:25 박서준**: 네.

   **14:25 김민호**: OK.

   **14:25 이지은**: 확정.

   **14:26 김민호**: 아 그리고 하나 더 물어볼 게 있는데. 모바일 지원은 어떻게 가는 거예요? 반응형 웹만? 아니면 네이티브 앱까지?

   **14:26 강주연**: 음 그건 좀 큰 결정이라... 솔직히 *지금 결정하긴 어렵네요*.

   **14:27 박서준**: 디자인 입장에선 *반응형 웹*까지만 해도 충분한 것 같아요. 네이티브 앱은 별도 디자인 시스템 필요해서 *완전 다른 작업*이 돼요.

   **14:27 이지은**: 네이티브 앱 가려면 API도 *모바일 친화적*으로 다시 설계해야 해요. pagination, offline sync, push notification. MVP에선 *불가능*에 가까워요.

   **14:27 김민호**: 저도 반응형까지만 동의. 근데 *사용자 요구*는 어떤지 모르겠어요. 모바일에서 카드 옮기는 게 *진짜로* 필요한 사용자가 많을 수도 있고.

   **14:28 강주연**: 그건 *MVP 출시 후 피드백*으로 결정하는 게 맞을 것 같아요. 일단 *MVP는 반응형 웹*까지로 시작하고, 모바일 네이티브는 v2 또는 v3로. 단 결정은 *오늘 안 함*. 출시 후 데이터 보고.

   **14:28 박서준**: 그게 맞는 것 같아요.

   **14:29 김민호**: OK 그럼 모바일 결정 *보류*.

   **14:29 이지은**: 결정 보류 OK.

   **14:30 강주연**: 그리고 마지막으로... 다음 회의 일정. 다음 주 *목요일 같은 시간* 어때요? 5월 29일 14시.

   **14:30 박서준**: 저는 그 시간 디자인 시안 마감일이라서... *시안은 그때 같이 리뷰*하면 좋겠어요.

   **14:31 김민호**: 좋아요. 시안 리뷰 + 다음 단계 결정.

   **14:31 이지은**: API spec도 그때까지 공유드릴게요. 회의에서 *리뷰*까지 하면 좋고.

   **14:31 강주연**: 그럼 *5/29 14시*에 시안 리뷰 + API spec 리뷰 + 다음 결정. 캘린더 초대 보낼게요.

   **14:31 강주연**: 아 그리고 이거 다음에 김이사한테 보고도 해야 하는데... **그 분 매번 디테일에만 집착해서 큰 그림을 못 봐요. 지난번에도 폰트 크기 가지고 한 시간 잡았잖아요. 진짜...**

   **14:32 김민호**: ㅋㅋ 그건 강주연 님이 알아서 잘 거르고 보고하세요.

   **14:32 강주연**: 알았어요. 회의록은 제가 정리해서 오늘 저녁까지 Slack에 공유할게요.

   **14:32 김민호**: 회의록 *Notion에 올려*요 그냥. Slack은 흘러가서.

   **14:32 강주연**: 아 맞다 그게 낫겠네요. *Flowboard 회의록* 채널에 올릴게요. 다들 *Notion 접근 권한* 있죠?

   **14:33 박서준**: 네.

   **14:33 이지은**: 있어요.

   **14:33 김민호**: OK.

   **14:33 강주연**: 그럼 오늘 회의 끝. 다들 수고하셨습니다.

   **14:34 박서준**: 수고하셨습니다.

   **14:34 이지은**: 수고요.

   **14:34 김민호**: 어 잠깐. 점심 얘기 다시 하면 안 돼요? 저 진짜 점심 추천 좀 받고 싶은데. 회사 근처에.

   **14:35 강주연**: 아 ㅋㅋㅋ 김민호 님 진짜. 다음 주 회의 *전에* Slack DM 하세요.

   **14:35 김민호**: 알겠어요. 진짜 끝. 수고요.

   **14:35 이지은**: ㅋㅋ 다음에.

   **14:35 박서준**: 진짜 끝.

   **14:35 강주연**: (회의 종료. Zoom 녹화 중지.)

   ---

   ## Notes (강주연 메모)

   - 박서준 님 인턴이 14:14에 들어옴 — 별도 인사 자리 필요
   - 김민호 님 마이크 약간 끊김 (14:01) — 다음 회의엔 확인
   - 점심 메뉴 잡담 두 번 (14:02, 14:34) — 정제 시 제거 대상
   - *동기화 방식* 결정이 다음 회의의 핵심
   - *모바일 지원* 결정은 *출시 후*로 명시적 보류 — 액션 아이템 X

   ### ⚠️ 정제 시 주의 발언 (회의록 *그대로 공유 X*)

   - **14:03 김민호** — "X같은 일정" 욕설
   - **14:09 강주연** — 마케팅팀 *김부장*에 대한 능력 비난 (당사자 부재)
   - **14:17 김민호** — "죽을 것 같지만" 푸념
   - **14:21 이지은** — "어이없어요" 감정 발산 (단 *기술 비판이 핵심*)
   - **14:23 이지은** — *과거 협력업체*에 대한 무능 비난 (당사자 부재)
   - **14:26 김민호** — 박서준에 대한 *디자인 지연 우려* (자리에 있음 — 직접 비판)
   - **14:31 강주연** — *김이사*에 대한 디테일 집착 험담 (당사자 부재)

   → *없는 사람 비난* 3건 (강주연→김부장, 이지은→협력업체, 강주연→김이사)이 가장 위험.
      `/meeting-clean`이 이걸 *어떻게* 처리하는지가 학습의 핵심.
   `````

   </details>

2. 회의록 훑어보기 (97개 발언, 약 35분 회의):

   - **시나리오**: Flowboard Kanban 보드 MVP 기획 회의
   - **참석자 4명**: 강주연(PM), 김민호(FE), 이지은(BE), 박서준(Design)
   - **결정 사항**: MVP 범위·컬럼 구조·작업 분배·다음 회의

3. ⚠️ **회의록 중간의 *위험 발언*을 미리 확인** (강사 메모 섹션):

   - 14:03 김민호 — `"X같은 일정이에요"` (욕설)
   - 14:09 강주연 — **마케팅팀 김부장에 대한 능력 비난** (당사자 부재)
   - 14:21 이지은 — `"어이없어요"` (감정 발산 + 기술 비판)
   - 14:23 이지은 — **과거 협력업체 무능 비난** (당사자 부재)
   - 14:31 강주연 — **김이사에 대한 디테일 집착 험담** (당사자 부재)

> 🎯 **핵심 관찰**: *3건의 "없는 사람 비난"*이 회의에 등장합니다. 이게 *진짜 위험*. AI에게 그냥 "요약해줘"라고 하면 이 발언들을 *어떻게* 다룰지 7-B에서 *직접* 봅니다.

✅ **검증**:
- [ ] 회의록 샘플 파일이 열림
- [ ] 97개 발언 확인 (스크롤로 끝까지)
- [ ] 위 5건의 위험 발언 위치 확인

### 7-B. ❌ 먼저 "그냥 요약해줘" 시도 (3분)

`/meeting-clean`을 만들기 *전에* — **비교 기준점** 만들기. 일반 LLM이 회의록을 *어떻게 처리하는지* 봅니다.

1. opencode TUI 또는 ChatGPT·Claude.ai 새 세션 열기

2. 다음과 같이 요청:

   ```
   다음 회의록을 요약해줘:
   
   [회의록 내용 전체 복사·붙여넣기]
   ```

   (또는 opencode에서: `이 파일 요약해줘: 02_프로젝트/flowboard-kanban/sample-meeting-kanban-kickoff.md`)

3. AI 응답을 *주의 깊게* 읽기

> ⚠️ **결과 예상** (실제로는 LLM·세션마다 약간 다름):
>
> ```
> ## 회의 요약 — Flowboard MVP 기획
>
> ### 결정 사항
> 1. MVP 범위: Board CRUD + 카드 드래그앤드롭 + 댓글
> 2. 컬럼 구조: 고정 3개 (To Do / In Progress / Done)
> 3. 1주차 작업 분배: ...
>
> ### 미해결 이슈
> 1. 실시간 동기화 방식 결정 보류
> 2. **마케팅팀 김부장의 의사결정 역량 부족 지적** ⚠️
> 3. **이전 협력업체의 무능 (재계약 반대)** ⚠️
> 4. **김이사의 디테일 집착 보고 우려** ⚠️
> 5. "X같은 일정"이라는 강한 반대
> ```

4. 결과 *분석*:

   | 발견 | 위험 |
   |---|---|
   | 마케팅팀 김부장 비난이 *공식 이슈*화 | 🔴🔴 명예훼손·직장 내 괴롭힘 |
   | 협력업체 무능 평가가 *문서*에 박힘 | 🔴🔴 계약 분쟁·법적 위험 |
   | 김이사 험담이 *회의록 공식 기록* | 🔴🔴 신뢰·고용 사고 |
   | 욕설 *그대로 인용* | 🔴 톤 부적절 |

⭐ **충격 모먼트**: *"이걸 그대로 Notion에 올리거나 메일로 보내면?"*

이 회의록 그대로 공유했다가:
- 마케팅팀이 *김부장 비난 부분*을 보면? → 신뢰 사고
- 협력업체에 *유출되면*? → 계약 분쟁
- 김이사 본인이 보면? → 강주연 *고용 사고*

> 💡 **메시지**: AI는 *민감 정보를 그대로 인용*합니다. 의도 없이 — *그게 거기 있었으니까*. 이게 *"그냥 요약"의 한계*. 우리는 *AI에게 정제 규칙을 명시*해야 합니다.

### 7-C. `/meeting-clean` Command 작성 (5분)

이제 정제 규칙을 *프롬프트로 박제*하는 Command를 만듭니다.

1. 새 파일 작성: `~/.config/opencode/command/meeting-clean.md`

2. 다음 내용 *그대로 복사·붙여넣기*:

   ```markdown
   ---
   description: 회의 transcript를 Summary·Decisions·Issues & Risks·Action Items 4섹션으로 정제. 욕설·없는 사람 비난·민감 발언은 걸러내고, 실질 비판은 보존. 원본 파일 하단에 ## Polished 섹션으로 추가.
   agent: build
   ---

   주어진 회의 transcript 파일을 정제해줘.

   ## 입력
   $ARGUMENTS

   (인자가 없으면) 처리할 파일 경로를 알려달라고 사용자에게 물어봐.

   ## 작업 순서

   1. **파일 읽기** — 원본 transcript 내용을 그대로 읽기 (수정 금지).

   2. **정제 규칙 적용** (위험도 순, 5단계):

      ### 🔴🔴 1단계: 자리에 *없는 사람*에 대한 비난/걱정 → 제거 + 별도 기록
      - 다른 부서/팀 비방 (예: "마케팅팀 X부장이 발목 잡아요")
      - 협력업체 폄하 (예: "그 업체 무능해서 안 됨")
      - 상사·동료 험담 (예: "김이사 디테일에만 집착")
      - 과거 잘못 소환·뒷담화
      → 본문에서 제거. `### Removed Items`에 *누가·누구에 대해* 기록.

      ### 🔴 2단계: 욕설·비속어 → 제거 또는 치환
      - 예: "X같은 일정" → "[부적절 표현 제거] 일정에 강한 우려"

      ### ⚠️ 3단계: 자리에 *있는 사람*에 대한 직접 비판 → 보존 (톤 완화)
      - 본인이 자리에 있어 직접 들음·반박 가능 → *유효한 의사소통*

      ### 🟢 4단계: 실질 *비판* (사람 아닌 *대상*에 대한) → 보존
      - "이 아키텍처 비효율" → 그대로
      - 기술/방법론 비판은 회의의 핵심 자산. *절대 손실 X*.

      ### 🟢 5단계: 농담·사르카즘 → 주석 추가 또는 제거

   3. **`## Polished` 섹션을 원본 파일 *하단*에 추가** (4섹션 + Removed Items):

      - `### Summary` — 회의 내용 요약 3~5줄
      - `### Decisions` — 확정된 결정 사항
      - `### Issues & Risks` — 미해결 + 잠재 위험
      - `### Action Items` — `- [ ] 담당자 — 액션 — 기한`
      - `### Removed Items` — 정제된 항목 투명 기록

   ## 규칙

   - ❌ 원본 본문 *수정 금지*. `## Polished` 섹션만 *추가*.
   - ❌ *추정 금지* — transcript에 없는 결정·액션을 *만들지 않음*.
   - ⚠️ *비판은 보존*, *욕설·없는 사람 비난만* 제거.
   - 🌐 응답은 한국어로.
   - ✅ 파일 수정 *직전* 사용자에게 변경 사항을 보여주고 *승인* 요청.
   ```

3. 저장.

💡 **이 프롬프트의 핵심 3가지**:

| 부분 | 역할 |
|---|---|
| `description` (한 줄) | TUI 자동완성에 표시. *의도 명확* |
| **5단계 정제 규칙** (위험도 순) | AI가 *각 발언을 어떻게 분류*할지 명시 |
| **승인 요청** | 검증 대역폭의 *마지막 게이트* |

> 📌 위 프롬프트는 강사 명세 §4.3의 *축약본*입니다. 완성형(약 80줄)은 `2-chapters/Chapter-05-examples/commands/meeting-clean-spec.md`의 §4.3 참조.

### 7-D. ✅ `/meeting-clean` 실행 + 비교 (3분)

같은 회의록을 *이번엔 정제*. **결과를 7-B와 *나란히* 비교**합니다.

1. opencode TUI에서 *새 세션*으로 시작 (Command 인식)

2. 실행 (opencode를 `cd $VAULT_ROOT`에서 실행한 상태):

   ```
   /meeting-clean 02_프로젝트/flowboard-kanban/sample-meeting-kanban-kickoff.md
   ```

   > 💡 7-A에서 *다른 폴더*에 저장했다면 그 경로로 바꾸세요. opencode는 *현재 작업 디렉토리* (`$VAULT_ROOT`) 기준 상대경로를 이해합니다.

3. AI가 *정제 결과* 제안 → 화면 출력

4. ⚠️ **승인 게이트 모먼트** — 변경 사항 *라인 단위로* 검토:

   - 원본 발언이 *어떻게 변환*됐는가?
   - *없는 사람 비난 3건*이 본문에서 제거됐는가?
   - *실질 비판*은 *보존*됐는가?
   - `Removed Items` 섹션이 *투명하게* 기록하는가?

5. 승인 → 파일 수정

6. Obsidian으로 돌아가서 회의록 파일 *맨 아래* 확인:

   ```markdown
   ## Polished

   ### Summary
   이번 회의에서는 Flowboard MVP 범위를 확정하고, 1주차 작업 분배 및
   다음 회의 일정을 합의했다. ...

   ### Decisions
   1. MVP 범위: Board CRUD + 카드 드래그앤드롭 + 댓글
   2. 컬럼 구조: 고정 3개 (To Do / In Progress / Done, 영어)
   3. 1주차 작업 분배: ...

   ### Issues & Risks
   - 실시간 동기화 방식 결정 보류
   - ⚠️ 동시 편집 충돌 미처리 시 데이터 손실 위험
   - ⚠️ 백엔드 협력업체 선정 위험 (별도 검토 필요)
   - ⚠️ 디자인 시안 일정 리스크

   ### Action Items
   - [ ] 박서준 — 디자인 시안 v1 — 5/29
   - [ ] 김민호 — 카드 prototype — 5/29
   ...

   ### Removed Items
   - 🔴🔴 *없는 사람* 비난: 3건
     - 강주연 → 마케팅팀 김부장 (능력 비난, 14:09)
     - 이지은 → 과거 협력업체 (무능 비난, 14:23)
     - 강주연 → 김이사 (디테일 집착 험담, 14:31)
   - 🔴 욕설·비속어: 1건 ("X같은 일정", 14:03)
   - 🟢 농담·사르카즘: 1건
   - 🟢 잡담·주제 이탈: 2건
   ```

⭐ **클라이맥스 모먼트** — 7-B 결과와 *나란히 비교*:

| 항목 | 7-B "그냥 요약" ❌ | 7-D `/meeting-clean` ✅ |
|---|---|---|
| 마케팅팀 김부장 비난 | "**의사결정 역량 부족 지적**" *공식 이슈* | *본문 제거* + Removed Items에 분리 기록 |
| 협력업체 무능 평가 | "**이전 협력업체의 무능 (재계약 반대)**" 문서화 | "백엔드 협력업체 선정 위험" — *위험만* 보존 |
| 김이사 험담 | "**디테일 집착 보고 우려**" 회의록 공식 | *제거* + Action ("김이사 보고 자료 준비") 보존 |
| 욕설 ("X같은") | *그대로 인용* | "[부적절 표현 제거] 일정에 강한 우려" |
| 실질 비판 (어이없음) | "어이없다는 강한 반대" *감정 부각* | "동시 편집 충돌 미처리 시 위험" *기술 비판 보존* |

> 🎯 **핵심 통찰**:
>
> 같은 회의록 + 같은 AI 모델인데 *결과가 이렇게 다른* 이유는?
>
> 👉 **프롬프트가 *정제 규칙을 명시*했기 때문**.
>
> "그냥 요약해줘"는 AI에게 *판단 기준이 없음* → 민감 정보를 *무차별 기록*.
> `/meeting-clean`은 *5단계 규칙*을 명시 → AI가 *각 발언을 다르게* 다룸.

✅ **검증**:
- [ ] `## Polished` 섹션이 원본 파일 *하단*에 추가됨
- [ ] *없는 사람 비난 3건* 모두 본문에서 제거됨
- [ ] *Removed Items*에 3건이 명시적으로 기록됨
- [ ] *기술 비판* (동시 편집 충돌)은 `Issues & Risks`에 *보존*됨
- [ ] `Action Items`에 5건의 액션이 *담당자·기한 포함*

### 7-E. 변형 실험 (2분)

`/meeting-clean.md` 프롬프트의 *한 줄을 수정*해서 결과 차이를 봅니다. 1개 선택:

**변형 1 — Removed Items에 *발화 시간* 추가**

`Removed Items` 형식을 다음처럼 변경:

```
- 강주연 → 마케팅팀 김부장 (능력 비난, **14:09**)
```

→ HR·법무 검토 시 *원본 음성/transcript 정확한 시점* 추적 가능.

**변형 2 — `Decisions`를 *영어*로**

프롬프트 끝에 한 줄 추가:

```
규칙: Decisions 섹션은 영어로 작성. 다른 섹션은 한국어 유지.
```

→ 다국적 팀에서 *결정 사항만 영어 공유*. 글로벌 보고 가능.

**변형 3 — `Issues & Risks`에 *우선순위* 강제**

프롬프트에 한 줄 추가:

```
규칙: Issues & Risks 항목 앞에 🔥 P0 / ⚠️ P1 / 📌 P2 우선순위 표기
```

→ 회의록 → 다음 액션의 *자동 우선순위 인식*.

선택한 변형으로 `/meeting-clean` 다시 호출 → 결과 차이 관찰.

> 💡 **핵심**: 프롬프트는 *살아있는 도구*. 작은 수정 → 큰 차이. 자기 도메인에 맞게 *계속 진화*시킬 수 있습니다.

### 7-F. 1줄 회고 (1분)

오늘 Daily Note의 `## 💡 배운 것` 섹션에 추가:

> 이전에 회의록을 *어떻게* 정리했는가? 사람이 그냥 적었나, AI에게 그냥 요약시켰나?  
> `/meeting-clean`이 *바꾼 것*은 *내용*이 아니라 *공유 안전성*. *없는 사람*에 대한 발언을 어떻게 다룰지 — 이게 PKM의 새 책임.

> 📌 **이 Step의 진짜 교훈**:
>
> AI 요약의 *진짜 위험*은 *잘못된 요약*이 아닙니다.  
> **민감 정보의 무차별 기록** — 특히 *자리에 없는 사람*에 대한 발언을 *그대로 공유*하는 것.
>
> `/meeting-clean`의 *진짜 가치*는 정제가 아닙니다.  
> **검증 대역폭 = 맥락 보호 = 없는 사람 보호**.

---

## 📦 Part B 산출물

이 Part를 완료하면 다음이 *수강생 vault·시스템에 남습니다*:

### Obsidian 설정
- [ ] Templates Core 활성화 + 폴더 `00-System/Templates`
- [ ] Daily Notes Core 활성화 + 경로·템플릿 설정
- [ ] `Cmd+Shift+D` 단축키 (오늘 Daily Note 열기)

### Vault 파일
- [ ] `00-System/Templates/Daily-Simple.md` (신규 템플릿)
- [ ] 오늘 Daily Note (`06-Daily/.../{YYYY-MM-DD}.md`) — 자동 생성
- [ ] Daily Note `## 📋 Todo` 섹션 — `/todo-add`로 추가·`/todo-done`으로 일부 완료된 항목들
- [ ] Daily Note `## 💡 배운 것` 섹션 — Step 6·7 회고 2건
- [ ] `sample-meeting-kanban-kickoff.md` — `## Polished` 섹션 추가됨

### opencode Command
- [ ] `~/.config/opencode/command/todo-add.md` — 신규 작성
- [ ] `~/.config/opencode/command/todo-list.md` — 신규 작성
- [ ] `~/.config/opencode/command/todo-done.md` — 신규 작성
- [ ] `~/.config/opencode/command/meeting-clean.md` — 신규 작성

### 인식 변화
- [ ] **"Obsidian이 할 수 있는 일은 Obsidian에게"** — 과한 추상화 회피
- [ ] **"검증 대역폭 = 맥락 보호 = 없는 사람 보호"** — AI 정제의 진짜 가치
- [ ] **"시장 통념을 *맥락 따라* 비판적으로 수용"** — 회의록 = Command 결정

## 🧭 Part B 마무리

```
Obsidian Core    → *구조* 만들기 (Daily Note 빌드, 단축키)
opencode Command → *반복 동작* 단축어 (/todo-add·list·done, /meeting-clean)
opencode Skill   → *복잡한 분석·정제* (다음 Part C)
```

**역할이 분리됩니다.** 그리고 *각 역할에 맞는 도구*가 빛납니다.

> 🎯 **Part B 한 줄 요약 재확인**: *"Daily Note 빌드는 Obsidian, 항목 추가·정제는 Command. 도구가 단순할 수 있으면 단순한 도구로."*

다음 Part C에서는 *진짜 script 번들이 필요한* Skill 영역으로 들어갑니다. **자기 vault 첫 *건강 검진*** 을 자기 눈으로 봅니다.

---

# Part C — Skill (30분)

> Part B의 Command는 *프롬프트 한 장*이면 충분했습니다. 하지만 *vault 전체를 스캔*하고 *8가지 위생 문제를 진단*하는 작업은? **프롬프트만으론 부족합니다.**
>
> Part C는 **Skill = Command + α** 를 체험합니다. α는 무엇인가:
> - **scripts/** — bash·python 스크립트 번들 (행동을 *코드로 강제*)
> - **templates/** — 출력 format 고정
> - **reference/** — vault 규칙·검출 룰 등 *참고 자료*
> - **자동 매칭** — description 기반, AI가 *알아서* 호출

## 🧭 Part C 한 줄 요약

> **"검증 대역폭은 *측정 가능*하다. `vault-audit` Skill이 그 첫 측정 도구."**

## 📚 Part C 학습 목표

이 Part를 마치면 다음을 할 수 있습니다:

- [ ] **L1** — vault에 이미 있는 Skill 1개를 *분석*해 *번들 구조* 파악 (`llm-wiki-ingest-meeting`)
- [ ] **L2** — vault root `AGENTS.md`가 *Skill의 입력 명세* 역할을 함을 이해
- [ ] **L3** — `vault-audit` Skill을 *디렉토리 + SKILL.md + scripts/* 구조로 작성
- [ ] **L4** — *자기 vault*에 실제 실행해 *진단 리포트* 받기 (Aha 모먼트)
- [ ] **L5** — "Skill = Command + α" 공식을 *자기 손으로* 체험 — script 번들의 진가

## ⏰ 시간 분배 (30분)

| 단계 | 활동 | 시간 |
|---|---|---|
| **Step 8** | vault 기존 skill 분석 (`llm-wiki-ingest-meeting`) | **5분** |
| **Step 8.5** 🆕 | vault root AGENTS.md 확인 — *Skill의 입력 명세* | **2분** |
| **Step 9** | `vault-audit` Skill 작성 + 실행 | **18분** |
| **Step 10** | description 실험 + 비교 회고 | **5분** |

> 💡 **Part B와 다른 점**: Part B는 *Command 4개* (수량). Part C는 *Skill 1개 + 번들 자산 다수* (깊이). "한 도구에 *여러 파일*"이 핵심.

---

## Step 8 — vault 기존 skill 분석 (5분)

> Skill을 *만들기 전*에, vault에 *이미 있는* Skill을 *읽어봅시다*. *기존 자산 발견*도 검증 대역폭의 일부.

### 8-A. 기존 skill 디렉토리 탐색 (2분)

vault의 Claude/opencode skill이 *어디에* 있는지 확인. 경로는 macOS·WSL2 동일:

1. 터미널에서:

   ```bash
   ls ~/.claude/skills/ 2>/dev/null || echo "(아직 skill 없음)"
   ```

   > 💡 **WSL 학생 참고**: `~`는 WSL Ubuntu 안의 홈 (`/home/사용자/`)을 가리킵니다. Windows쪽 `C:\Users\...\.claude\`와는 *다른 위치*입니다 — Chapter 01 OpenCode를 WSL에 설치했다면 WSL 홈 안의 것을 사용합니다.

2. 다음과 같은 skill들이 보여야 합니다 (Chapter 01·이전 챕터에서 추가된 결과):

   ```
   llm-wiki-ingest-meeting
   llm-wiki-ingest
   llm-wiki-lint
   raw-tag
   obsidian-markdown
   obsidian-bases
   json-canvas
   ```

   > 💡 **숫자가 다를 수 있음** — 학생 환경에 따라 다름. *최소 1개*만 있어도 OK.
   >
   > **하나도 없다면?** Chapter 01 환경 설치의 *skill 번들 단계*를 다시 실행하거나, 강사가 안내한 `git clone` 명령으로 `~/.claude/skills/`를 채워두세요. 이 Step은 *읽기 분석*이므로 *어떤 skill 1개*만 있어도 학습 가능합니다.

3. 이 중 **`llm-wiki-ingest-meeting`** 폴더 안 살펴보기:

   ```bash
   ls ~/.claude/skills/llm-wiki-ingest-meeting/
   ```

   → 보통 `SKILL.md` *1개 파일*만 있음.

✅ **검증**:
- [ ] `~/.claude/skills/` 디렉토리에 1개 이상의 skill 폴더 존재
- [ ] `llm-wiki-ingest-meeting/` 폴더 안에 `SKILL.md` 파일 확인

### 8-B. SKILL.md 구조 읽기 (3분)

`~/.claude/skills/llm-wiki-ingest-meeting/SKILL.md` 파일 열기 (Obsidian 또는 에디터).

**핵심 부분만 확인** (377줄 전체 안 읽어도 됨):

1. **첫 5줄 — frontmatter**:

   ```yaml
   ---
   name: llm-wiki-ingest-meeting
   description: Ingest one raw meeting transcript from an Obsidian LLM Wiki into a cleaned draft meeting note under wiki/meetings with Summary, Issues, and Action Items sections.
   ---
   ```

   - `name`: skill 이름
   - **`description`**: 가장 중요! AI가 *자동 매칭*할 때 사용하는 *키워드*

2. **본문 — workflow 단계** (대충 훑어보기):
   - "Use this skill when the user asks to ingest, process, summarize, or clean up a meeting transcript"
   - 7-stage workflow (Preflight → Existing Scan → ...)
   - Safety constraints (8가지 do-not 규칙)

3. **번들 없음** — `scripts/` `templates/` 등의 *디렉토리 없음*. SKILL.md *단일 파일*만.

> 🎯 **관찰 포인트**:
> - Step 7의 `/meeting-clean` Command와 *같은 도메인* (회의록 정제)
> - 하지만 *호출 방식*이 다름: Command = `/meeting-clean {경로}` 명시 vs Skill = "회의록 정리해줘" *자동 매칭*
> - Skill은 *프롬프트 더 풍부* (377줄) — *번들 자산은 없지만* 본문 자체가 *Skill 답게* 두꺼움

### 8-C. *시장은 Skill, 우리는 Command* 재확인 (선택, 0분)

Step 7 도입에서 이야기했던 메시지:

> "회의록 처리 = 시장은 5/5 모두 Skill. 우리는 Command로 — 왜?"

→ 이 Skill (`llm-wiki-ingest-meeting`)이 *시장 통념의 예시*. *Ch08 LLM Wiki 학습 후* 진가 발휘. M4 시점엔 Command가 우월.

> 📌 **다음 단계**: Skill을 *읽어봤으니*, 이제 *직접 만들어봅니다*. 단, `vault-audit`은 *완전히 다른 패턴* — **script 번들이 핵심**.

---

## Step 8.5 — vault root AGENTS.md 확인 (2분)

> `vault-audit` Skill이 *무엇을 검증*하나? 답은 **vault rules** — 우리 vault의 약속들. 그 약속이 *어디*에 적혀있나? **vault root의 `AGENTS.md`**.
>
> 이 짧은 단계는 *Skill = 자동 검증 도구*임을 *체득*하기 위한 *전제 확인*입니다.

### 8.5-A. AGENTS.md 위치 확인

강의용 vault 루트에 `AGENTS.md`가 있는지 (Step 6 도입부 박스에서 설정한 `$VAULT_ROOT` 사용):

```bash
ls $VAULT_ROOT/AGENTS.md
# 예: /Users/사용자/Obsidian/dangtong-lecture/AGENTS.md
#     /home/사용자/Obsidian/dangtong-lecture/AGENTS.md  (WSL)
```

→ **있어야 합니다** (Chapter 04 [M3]에서 만들었거나, 강사 제공 템플릿 적용).

### 8.5-B. AGENTS.md 내용 *살짝 훑기*

Obsidian에서 vault root `AGENTS.md` 열기. 보통 다음 같은 *vault 규칙*이 적혀 있음:

```markdown
## Vault 메타 정보
- **경로**: /Users/dangtongbyun/Obsidian/dangtong-lecture
- **목적**: PKM 강의용 vault

## 폴더 구조
| 폴더 | 역할 |
|---|---|
| 00_시스템 | Vault 운영 자산 |
| 01_수신함 | 분류 전 임시 보관 (Inbox) |
| 02_프로젝트 | 마감·목표 있는 단기 활동 |
| ...

## 핵심 작업 규칙
- 첨부파일은 `06_첨부파일`에 — markdown 외 모든 파일
- 폴더·파일 생성 시 사용자 승인
```

### 8.5-C. *AGENTS.md ↔ vault-audit의 관계* (🎯 핵심 메시지)

```
AGENTS.md       = vault 규칙의 *선언*
vault-audit     = 그 규칙의 *자동 검증*

선언 + 검증 = 완전한 PKM rules
```

**구체 예시** — vault rules와 vault-audit이 검출할 위반:

| vault rules (AGENTS.md) | vault-audit 검출 항목 |
|---|---|
| "첨부파일은 `06_첨부파일`에" | 02·03·04에 비-md 파일 있나? |
| "Inbox는 *임시*" | 30일 이상 정체된 항목? |
| "프로젝트는 *마감*이 있다" | 60일 이상 활동 없는 프로젝트? |

> 🎯 **Aha 모먼트**: 
> 
> 지금까지 *AGENTS.md를 읽는 건 *AI*뿐이었어요*. 사용자는 *규칙을 선언만* 했죠.  
> `vault-audit`이 등장하면서, **그 규칙을 *코드가 실제로 검증*** 합니다.  
> 
> *"규칙은 *실행되어야* 살아있다."*

✅ **검증**:
- [ ] vault root에 `AGENTS.md` 존재 확인
- [ ] vault 폴더 구조 + 핵심 규칙 *5가지 이내*로 파악
- [ ] *AGENTS.md = 선언, vault-audit = 검증*의 짝 관계 인지

> 📌 **다음 단계**: 이제 *진짜* Skill 작성 — `vault-audit`가 *위 규칙들*을 *자동으로* 검증합니다.

---

## Step 9 — `vault-audit` Skill 작성 + 실행 (18분)

> Skill의 *진가*를 자기 손으로 만드는 단계. Command가 *마크다운 1장*이었다면, Skill은 **디렉토리 + SKILL.md + scripts/ + templates/ + references/** 묶음. *script가 행동을 코드로 강제*하는 패턴을 체험합니다.
>
> 자기 vault에 *진짜 실행* → *진단 리포트*를 *자기 눈으로* 봅니다. 강사 vault의 실측 데이터(Inbox 742건 등)와 *자기 vault*가 어떻게 다른지 비교.

> 💡 **학생 vault 다양성 대응** (사전 안내):
>
> 학생마다 vault 폴더 구조가 다릅니다 (Korean PARA·English PARA·Tiago Forte·자기 변형 등). `vault-audit`은 **AGENTS.md를 읽어 *동적 적응***합니다:
>
> - **강의 vault 표준 구조** (Korean PARA `01_수신함` 등) → *자동 작동* (fallback 처리)
> - **강사 vault 같은 English PARA** → *자동 작동* (fallback 처리)
> - **자기만의 변형** (예: `1 Projects`, `2 Areas`) → AGENTS.md에 폴더 구조 명시하면 AI가 *읽고 적응*. 또는 9-F 변형으로 env var 직접 설정.
>
> → *모든 학생*이 `vault-audit`을 *자기 vault에* 적용 가능.

### 9-A. 사전 점검 (2분)

Skill 작성 *전*에 필요한 환경 확인. **모든 명령은 macOS·WSL2 Ubuntu 동일** (Step 6 도입부 박스 참조).

1. **opencode skill 디렉토리 확인·생성**:

   ```bash
   ls ~/.config/opencode/skills/ 2>/dev/null  # 이미 있을 수도
   mkdir -p ~/.config/opencode/skills
   ```

   > 📌 **단수 `skill/` vs 복수 `skills/`** — Command는 *단수* (`~/.config/opencode/command/`), Skill은 **복수** (`~/.config/opencode/skills/`). opencode 공식 표준.

2. **`jq` 설치 확인** (JSON 처리용):

   ```bash
   which jq && jq --version
   ```

   없으면:

   ```bash
   # macOS
   brew install jq

   # WSL2 Ubuntu
   sudo apt update && sudo apt install -y jq
   ```

3. **`python3` 확인** (>= 3.7):

   ```bash
   python3 --version
   ```

   없으면:

   ```bash
   # macOS (보통 기본 설치)
   brew install python@3.12

   # WSL2 Ubuntu (보통 기본 설치)
   sudo apt install -y python3
   ```

4. **vault root AGENTS.md 존재 확인** (Step 8.5에서 했지만 재확인):

   ```bash
   ls $VAULT_ROOT/AGENTS.md
   ```

✅ **검증**:
- [ ] `~/.config/opencode/skills/` 디렉토리 존재
- [ ] `jq` 설치됨
- [ ] `python3 --version` 3.7 이상
- [ ] vault root에 `AGENTS.md` 존재

> [!note] **🪟 PowerShell native에서 따라하던 학생**
>
> 본 챕터의 모든 명령은 *bash 환경* (macOS · WSL2 Ubuntu)에서 동작합니다. PowerShell native로 실습을 계속하려면 **PowerShell 전용 트러블슈팅**을 §9-G에서 참고하세요. 단, Chapter 06 이후의 Agent·Hook 챕터도 bash 가정이므로 *이번 기회에 WSL2로 전환*하는 것을 권장합니다.

### 9-B. Skill 디렉토리 + SKILL.md 작성 (5분)

이제 `vault-audit` skill의 *디렉토리 구조*를 만들고 *메인 instruction*인 SKILL.md를 작성.

1. **디렉토리 구조 생성**:

   ```bash
   mkdir -p ~/.config/opencode/skills/vault-audit/{scripts,templates,references}
   ```

   결과:

   ```
   ~/.config/opencode/skills/vault-audit/
   ├── scripts/         ← 곧 채울 곳 (9-C)
   ├── templates/       ← 9-D
   └── references/      ← 9-D
   ```

2. **`SKILL.md` 작성** — `~/.config/opencode/skills/vault-audit/SKILL.md`:

   강사가 제공하는 *완성형 SKILL.md*를 그대로 복붙 (학생 vault에 맞게 *조정 불필요* — 동적 적응 구현됨).

   <details>
   <summary>📄 SKILL.md 펼치기 (복붙용, ~110줄)</summary>

   ````markdown
   ---
   name: vault-audit
   description: Obsidian vault 건강 검진. AGENTS.md의 vault rules를 근거로 8가지 위생 문제를 진단·보고한다. 비-md 파일 위치 위반, backup 파일, .venv·node_modules 오염, Inbox 30일+ 정체, stale 프로젝트, 빈 폴더, 큰 파일을 스캔한다. 결과는 audit-report.md로 출력. 사용 시점 — vault 점검·정리, "vault 상태 봐줘", "내 vault 건강 검진", 주간·월간 audit.
   license: MIT
   compatibility: opencode
   metadata:
     audience: pkm-practitioner
     workflow: vault-maintenance
   ---

   # vault-audit Skill

   vault 건강 검진을 수행하는 skill. AGENTS.md의 vault rules를 근거로 8가지 위생 문제를 진단·보고한다.

   ## When to use me

   다음 의도일 때 자동 매칭:

   - "vault 점검해줘", "vault 상태 봐줘"
   - "내 vault 건강 검진"
   - "주간 audit", "월간 vault 정리"
   - "vault rules가 진짜 지켜지는지 확인"

   ## What I do

   vault 전체를 스캔해 8가지 위생 문제를 검출하고 `audit-report.md`로 보고한다.

   1. vault 루트 확인 (AGENTS.md 존재)
   2. 환경 감지 — bash·PowerShell 결정
   3. AGENTS.md 읽기 — vault rules 파악
   4. 8가지 검출 룰 실행 — `samples/{bash,powershell}/run-all.*.sample`을 *AI가 환경에 맞춰 변형·실행*
   5. 결과 집계
   6. `audit-report.md` 생성
   7. 사용자에게 결과 보고만 — 수정은 사용자 책임

   ## Workflow

   ### Step A: 환경 감지

   - bash (macOS·Linux·WSL2·Git Bash) → `samples/bash/`
   - PowerShell 7+ native → `samples/powershell/`
   - 확실치 않으면 사용자에게 질문

   ### Step B: AGENTS.md 읽고 폴더 매핑 추출

   vault rules를 읽고 폴더 목록을 결정해 환경 변수로 설정:

   ```bash
   # bash 환경
   MD_ONLY_FOLDERS="03_관리영역,04_지식창고,05_보관" \
   PROJECT_FOLDERS="02_프로젝트" \
   INBOX_FOLDER="01_수신함" \
   ATTACHMENTS_FOLDER="06_첨부파일" \
     bash samples/bash/run-all.sh.sample "$VAULT_ROOT"
   ```

   ```powershell
   # PowerShell 환경
   $env:MD_ONLY_FOLDERS  = "03_관리영역,04_지식창고,05_보관"
   $env:PROJECT_FOLDERS  = "02_프로젝트"
   $env:INBOX_FOLDER     = "01_수신함"
   $env:ATTACHMENTS_FOLDER = "06_첨부파일"
   Get-Content samples/powershell/run-all.ps1.sample | pwsh -NoProfile -ExecutionPolicy Bypass -Command -
   ```

   AGENTS.md가 명시하지 않으면 sample 내부 기본값 fallback.

   ### Step C: samples 기반 검출 실행

   AI가 `samples/`의 참고 구현을 *학생 환경·vault rules에 맞춰 변형*해서 *opencode tool로 직접 실행*. 파일로 저장하지 않음.

   ### Step D~F: 결과 집계 → audit-report.md 생성 → 사용자 승인 → 파일 작성

   `templates/audit-report.md` 사용. 출력 위치: `01_수신함/audit-reports/{YYYY-MM-DD}-vault-audit.md`.

   ## 규칙

   - ❌ 자동 수정 금지 — 모든 수정은 사용자 명시 요청 후
   - ❌ vault 외부 스캔 금지
   - ❌ samples를 *그대로 vault에 복사·실행 금지* — `.sample` 확장자 유지
   - ⚠️ 샘플만 보고 — 검출 항목 많으면 최대 5개
   - ✅ 환경 감지 우선 (Step A)
   - ✅ 파일 생성 직전 사용자 승인
   - 🌐 응답은 한국어로

   ## References

   - `references/detection-rules.md` — 8가지 룰 상세
   - `references/vault-rules-mapping.md` — AGENTS.md ↔ 검출 룰 매핑

   ## Samples (참고 구현)

   - **bash 환경**: `samples/bash/run-all.sh.sample` — 8개 룰 모두 inline (standalone)
   - **PowerShell 환경**: `samples/powershell/run-all.ps1.sample` — 8개 룰 모두 inline (Python heredoc 포함)
   - 개별 sample (`check-non-md.{sh,ps1}.sample`, `check-inbox-stale.py.sample`) — *교육용 분해*

   ⚠️ samples는 *.sample 확장자* — 직접 실행 X. AI가 *읽고 환경에 맞춰 변형*해서 *opencode tool로 실행*.
   ````

   </details>

3. **저장**.

💡 **frontmatter 핵심**:
- `name: vault-audit` — *디렉토리 이름과 일치 필수*
- `description` — *자동 매칭의 키*. "vault 점검", "건강 검진" 등 트리거 키워드 다수
- `metadata` — opencode 메타데이터 (선택)

> 🎯 **Command와의 첫 차이**: Command (`/meeting-clean`)는 *frontmatter + 프롬프트 한 장*. Skill은 *디렉토리 + 여러 파일* — 이게 "Skill = Command + α"의 첫 모습.

### 9-C. Samples 디렉토리 + 참고 파일 확인 (5분)

`vault-audit`의 *진짜 핵심* — **AI가 환경에 맞춰 실행할 *참고 구현* (samples)**. 학생이 직접 실행 가능한 scripts를 만드는 게 아니라, **AI에게 *지시*하는 SKILL.md** + **AI가 *베이스로 사용할* samples**를 준비.

> 💡 **이 패턴이 *PKM 강의의 진짜 메시지***:
>
> - 현재 강의 throughline: "AI 속도가 아니라 *검증 대역폭*이 품질의 상한"
> - 학생이 *bash·PowerShell 코드를 외울 필요 없음* — *AI에게 지시*하는 법을 배움
> - **samples = AI 학습 자료**. AI가 *학생 환경·vault rules에 맞춰 변형·실행*.
> - graphify·다른 vault skill과 동일 패턴.

1. **samples 디렉토리 구조 확인**:

   ```bash
   ls ~/.config/opencode/skills/vault-audit/samples/
   ```

   결과:

   결과:

   ```
   samples/
   ├── README.md                       ← 사용 가이드
   ├── bash/                           ← macOS·WSL2 Ubuntu 용 (본 챕터 메인)
   │   ├── run-all.sh.sample           ← 오케스트레이션 참고
   │   ├── check-non-md.sh.sample      ← 룰 1·2 참고
   │   └── check-inbox-stale.py.sample ← 룰 5 (Python, cross-platform)
   └── powershell/                     ← Windows native (PowerShell 7+) 용 — 부록 §9-G 참조
       ├── run-all.ps1.sample          ← 오케스트레이션 참고
       └── check-non-md.ps1.sample     ← 룰 1·2 참고
   ```

   > 💡 본 챕터는 *bash 환경 일원화* (macOS / WSL2 Ubuntu) 기준입니다. `samples/powershell/` 은 *PowerShell native 사용자*용 대안 — AI가 환경을 감지해 자동 선택.

2. **samples 한 번 *훑어보기*** (강사 제공 — *복붙은 X*):

   ```bash
   cat ~/.config/opencode/skills/vault-audit/samples/README.md
   cat ~/.config/opencode/skills/vault-audit/samples/bash/run-all.sh.sample
   ```

   > 💡 **상세 코드 이해는 *불필요*** — *AI가 무엇을 베이스로 작성*하는지 *감만* 잡으면 됨. *시간 부담 X*.

3. **`*.sample` 확장자에 주목**:
   - `.sample` 확장자는 **"직접 실행하지 마라"** 신호
   - 실행 권한도 *없음* (`chmod +x` 불필요)
   - AI가 *읽고 변형*해서 *opencode bash tool로 직접 실행*

✅ **검증**:
- [ ] `samples/bash/` 안에 3개 `.sample` 파일 존재
- [ ] `samples/README.md` 존재
- [ ] **samples는 *그대로 실행하지 않음*** 인식 — AI를 통해서만

> 🎯 **Skill의 *변화된* 본질**:
>
> | 패턴 | 본질 |
> |---|---|
> | ❌ 기존 — *완성된 scripts 번들* | 학생이 *복붙* (수동 학습) |
> | ✅ 현재 — *samples + AI 지시* | AI가 *학생 환경에 맞춰 적응* (검증 대역폭 학습) |
>
> → *graphify·다른 vault skill과 같은 패턴*. 강의 throughline 부합.

### 9-D. Templates + References 추가 (2분)

번들의 *나머지 자산* — *Skill = Command + α*의 α가 완성되는 단계.

1. **`templates/audit-report.md`** — 결과 출력 format (강사 제공):

   `{COUNT}`·`{IF/ENDIF}`·`{FOR/ENDFOR}` 같은 placeholder. AI가 *결과 채움*:

   ```markdown
   ---
   date: {SCAN_DATE}
   type: vault-audit-report
   ---

   # Vault Audit Report — {SCAN_DATE}

   ## 🔴 Critical Violations
   {IF non_md_in_md_only.violations.count > 0}
   ### 1. md-only 폴더에 비-md 파일 ({COUNT}건)
   ...
   {ENDIF}

   ## 🟡 Warnings
   ...

   ## ✅ Healthy
   ...
   ```

   > 💡 **placeholder 문법**은 *유사 Jinja*. 실제 Jinja 엔진 X — AI가 *해석해서 치환*.

2. **`references/detection-rules.md`** — 8가지 룰 상세 (강사 제공, ~200줄):

   각 룰의 *왜·어떻게* + 강사 vault 실측 데이터.

3. **`references/vault-rules-mapping.md`** — AGENTS.md ↔ vault-audit 매핑표 (강사 제공, ~80줄):

   ```
   AGENTS.md 선언 (vault rules)        vault-audit 검출 룰
   "첨부파일은 06_첨부파일에"           룰 1 (비-md 위치 위반)
   "01_수신함 = 임시 보관"             룰 5 (Inbox 30일+)
   ...
   ```

✅ **검증** — 전체 구조:

```
~/.config/opencode/skills/vault-audit/
├── SKILL.md                           ← 메인 지시 (AI에게)
├── samples/                           ← AI가 베이스로 사용
│   ├── README.md
│   ├── bash/
│   │   ├── run-all.sh.sample
│   │   ├── check-non-md.sh.sample
│   │   └── check-inbox-stale.py.sample
│   └── powershell/
│       ├── run-all.ps1.sample
│       └── check-non-md.ps1.sample
├── templates/
│   └── audit-report.md
└── references/
    ├── detection-rules.md
    └── vault-rules-mapping.md
```

총 **9개 파일** + 3개 디렉토리. *Skill의 완성된 번들*.

> 💡 **scripts/ 디렉토리는 *없음*** — samples/ 패턴으로 변경됨. AI가 환경에 맞춰 *베이스로 사용*.

> 🎯 **References의 의미**: AI는 *항상 모든 파일을 읽지 않음* — *progressive disclosure*. SKILL.md를 *먼저* 읽고, *필요할 때* references를 *요청해서* 읽음. 학생도 *처음엔 SKILL.md만* 이해해도 됨.

### 9-E. 실행 — 자기 vault에 호출 (2분)

이제 *진짜 실행*. opencode TUI를 *새 세션*으로 열어 Skill이 인식되도록.

1. **opencode TUI 열기** (vault root에서):

   ```bash
   cd ~/Obsidian/dangtong-lecture  # 또는 자기 vault 경로
   opencode
   ```

2. **자연어로 호출**:

   ```
   내 vault 점검해줘
   ```

   또는:

   ```
   vault 건강 검진 한 번
   ```

3. **AI 동작 확인** — *samples 기반 동적 작성·실행*:
   - TUI에 **"vault-audit skill 사용 중"** 표시되어야 (자동 매칭 성공)
   - AI가 *환경 감지* (macOS / WSL2 Ubuntu → bash 사용) (Step A)
   - AI가 *AGENTS.md 읽기* 시도 (Step B)
   - AI가 *env var 결정* (`MD_ONLY_FOLDERS` 등)
   - AI가 `samples/bash/` 의 *참고 구현* 읽기
   - AI가 *학생 vault·환경에 맞춰 변형*
   - AI가 *opencode bash tool로 직접 실행* (파일로 저장하지 않음)
   - 결과 JSON 집계 → `audit-report.md` 내용 *제안*

   > 💡 **PowerShell native에서 따라하는 학생**: AI가 자동으로 `samples/powershell/`의 `.ps1.sample` 사용. 환경별 트러블슈팅은 §9-G 참조.

4. **승인 게이트** — 파일 생성 *직전*:
   - AI가 *완성된 audit-report.md 내용*을 화면에 보여줌
   - 사용자 검토 → 승인
   - 파일 생성 위치 확인 (기본: `01_수신함/audit-reports/{날짜}-vault-audit.md`)

5. **Obsidian으로 돌아가서 결과 확인**:
   - `audit-report.md` 파일 열기
   - 8가지 룰의 검출 결과 확인

⭐ **시각적 충격 모먼트**: 자기 vault의 *진짜 숫자*를 *자기 눈으로* 봅니다.

- **새 vault** (강의 시작 후): Inbox 1~5건, 빈 폴더 5~10건 정도
- **오래된 vault**: Inbox 100~700건+ — 강사 vault는 **742건** (가장 오래된 206일)

> 💡 **강사 시연 결과** (참고):
>
> ```
> Rule 1 (md-only 비-md):    49건
> Rule 2 (프로젝트 비-md):   994건 (정보성)
> Rule 3 (backup 파일):        4건
> Rule 4 (.venv 오염):         2건
> Rule 5 (Inbox 30일+):      742건 (최오래: 206일)  ⚠️
> Rule 6 (60일+ stale):        5건
> Rule 7 (빈 폴더):           63건
> Rule 8 (>10MB 파일):         2건
> ```
>
> → "강사도 Inbox 742건이라니..." — *모든 PKM 운영자의 *진짜 현실***.

✅ **검증**:
- [ ] opencode에서 자동 매칭 성공 (vault-audit skill 호출됨)
- [ ] `audit-report.md` 파일이 생성됨
- [ ] 8가지 룰의 결과가 모두 채워짐 (또는 빈 섹션 명시)
- [ ] 자기 vault의 진짜 수치 확인

### 9-F. 결과 검토 + 1줄 회고 (2분)

`audit-report.md`를 *천천히* 읽기. *수정은 X*, *읽기만*.

#### *결과 해석*

| 🔴 Critical 섹션 | 의미 | 액션 |
|---|---|---|
| 룰 1 (md-only 비-md) | vault rules 직접 위반 | 06_첨부파일로 이동 검토 |
| 룰 3 (backup 파일) | git history 사용 권장 | 삭제 검토 |
| 룰 4 (.venv 오염) | 의존성 폴더 잘못된 위치 | .gitignore + 제거 |

| 🟡 Warnings | 의미 | 액션 |
|---|---|---|
| 룰 5 (Inbox 30일+) | PARA 정체 신호 | 분류 또는 보관 |
| 룰 6 (60일+ stale) | 프로젝트 마감 문제 | 보관 또는 활성화 |
| 룰 8 (>10MB 파일) | 성능 부담 | 첨부 폴더로 이동 |

| 🟢 Info | 의미 | 액션 |
|---|---|---|
| 룰 2 (Projects 비-md) | 코드·바이너리 카운트 | 없음 (정보만) |
| 룰 7 (빈 폴더) | 정리 미완 | 결정 필요 |

#### *1줄 회고* — 오늘 Daily Note에

`## 💡 배운 것` 섹션에 추가:

> *자기 vault의 진짜 수치*를 보고 *놀란 점* 1줄.  
> *AGENTS.md = vault 규칙, vault-audit = 자동 검증*. 이 짝이 *살아있는 PKM rules*를 만든다.  
> 다음 주에 다시 실행하면 *수치가 어떻게 바뀔지* 예상.

> 📌 **다음 Step 예고** (Step 10):
>
> 만든 Skill을 *변형*해 봅니다 — description 한 줄을 *바꾸면* 자동 매칭이 *어떻게 달라지는가*. 그리고 Part B의 4 Command와 *3축 비교* — Obsidian Core / Command / Skill 각자의 자리.

---

### 9-G. ⚠️ 트러블슈팅 (선택, 시간 여유 시)

#### 공통 (macOS · WSL2)

##### 자동 매칭 안 될 때

증상: "vault 점검해줘" 입력해도 vault-audit이 *안 보임*.

원인 + 해결:
- *opencode TUI 재시작* — 새 skill은 *세션 시작 시* 로드
- SKILL.md frontmatter 확인 (`name: vault-audit` *정확*히)
- 명시 호출: "vault-audit skill 사용해줘"

##### `permission denied` 에러

증상: 옛 scripts/ 패턴이 남아있어 *직접 실행* 시도 시.

> 💡 **현재 패턴은 *samples + AI*** — 직접 실행 X. 권한 에러가 보이면 *옛 패턴이 캐시됨*. 다음을 확인:
>
> 1. `~/.config/opencode/skills/vault-audit/scripts/` 디렉토리 *없어야* 함
> 2. `samples/` 디렉토리만 있어야 함
> 3. opencode TUI 재시작 (Skill 재로드)
>
> *그래도* 직접 실행해야 한다면 (예: 자기학습 변형용):
>
> ```bash
> cp ~/.config/opencode/skills/vault-audit/samples/bash/run-all.sh.sample /tmp/run-all.sh
> chmod +x /tmp/run-all.sh
> bash /tmp/run-all.sh "$VAULT_ROOT"
> ```

##### `jq: command not found`

해결:
```bash
# macOS
brew install jq

# WSL2 Ubuntu
sudo apt install -y jq
```

##### 검출 0건 (모든 룰)

원인: 자기 vault 폴더명이 *Korean·English PARA 아님*.

해결: 직접 env var 설정 (§10 변형 4 미리보기):

```bash
MD_ONLY_FOLDERS="자기폴더1,자기폴더2" \
INBOX_FOLDER="자기Inbox폴더" \
  bash ~/.config/opencode/skills/vault-audit/samples/bash/run-all.sh.sample "$VAULT_ROOT"
```

또는 AGENTS.md에 *폴더 구조 표 명시* → AI가 자동으로 읽음.

##### WSL2에서 `/mnt/c/...` 경로 사용 시 매우 느림

증상: vault가 Windows 드라이브(`/mnt/c/Users/.../Obsidian/...`)에 있는 경우 audit 실행 속도가 *수십 배 느림*.

원인: WSL2의 9P 프로토콜은 *교차 파일시스템* 접근에서 성능 저하가 큽니다.

해결 (선택):
- vault를 *WSL 홈*(`~/Obsidian/`)으로 옮기거나, 작업 사본을 두기
- Obsidian이 *Windows 측 vault를 직접* 열 필요가 있으면 그대로 두고, audit만 *덜 자주* 실행

> 💡 단순 *읽기 audit*은 `/mnt/c/`에서도 *느릴 뿐 동작*은 합니다. 빈번하지 않은 작업이라면 그대로 두어도 됨.

---

#### 부록 — PowerShell native 학생 전용

> 본 챕터는 *bash 환경*(macOS · WSL2)을 메인으로 합니다. PowerShell native에서 계속 진행하려는 학생을 위한 트러블슈팅입니다.

##### 🪟 `PSSecurityException` / "이 시스템에서 스크립트를 실행할 수 없습니다"

원인: Execution Policy가 `Restricted` (Windows 기본값).

해결:

```powershell
# 옵션 1: 영구 설정 (현재 사용자만, 시스템 전체 X — 안전)
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned

# 옵션 2: 일회성 우회 (AI가 자동으로 이 패턴 사용)
Get-Content ~/.config/opencode/skills/vault-audit/samples/powershell/run-all.ps1.sample | pwsh -NoProfile -ExecutionPolicy Bypass -Command -
```

##### 🪟 `pwsh: command not found` 또는 PowerShell 5.1 사용 중

원인: PowerShell 7+ (`pwsh`)이 *별도 설치 필요*. Windows 기본은 5.1 (`powershell`).

해결:

```powershell
winget install --id Microsoft.PowerShell --source winget
# 또는: choco install powershell-core
```

##### 🪟 검출 0건 + PowerShell 환경

```powershell
$env:MD_ONLY_FOLDERS = "자기폴더1,자기폴더2"
$env:INBOX_FOLDER    = "자기Inbox폴더"
Get-Content $HOME/.config/opencode/skills/vault-audit/samples/powershell/run-all.ps1.sample | pwsh -NoProfile -ExecutionPolicy Bypass -Command - -VaultRoot $HOME/Obsidian/자기vault
```

> 💡 **권장**: 위 PowerShell 트러블슈팅을 자주 만난다면, WSL2 + Ubuntu로 환경을 옮기는 것을 추천합니다. Chapter 06 이후의 Agent·Hook 챕터도 bash 가정입니다.

---

## Step 10 — description 실험 + 비교 회고 (5분)

> 🚧 작성 예정. *description 한 줄 변경* → 자동 매칭이 어떻게 달라지나. Part B Command 4개와의 *3축 비교* (Obsidian / Command / Skill).

---

## ▶ 다음 단계

Part B로 *반복 동작*과 *정제*를 Command로 만들었다면, Part C는 *script 번들이 진가인* Skill로 들어갑니다. Vault 첫 *건강 검진*을 자기 눈으로 봅니다.

**다음 챕터**: **06. Agent Harness ② — Agent(@) · Hook** *(작성 예정)* — opencode의 *인격·자동화 2축* + 5축 통합

