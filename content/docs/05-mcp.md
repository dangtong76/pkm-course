---
title: "05. MCP 연동"
weight: 5
date: 2026-05-10
draft: false
---

> **⏰ 시간**: TODO (예상 ~1시간 = 이론 20분 + 실습 40분)
> **🎯 목표**: TODO

---

## 📄 이론 슬라이드

> 이 챕터의 이론(MCP 개념·아키텍처·왜 필요한가 등)은 아래 슬라이드로 다룹니다.

<iframe src="/pdf/05-mcp-integration_v1.0.pdf"
        width="100%"
        height="600px"
        style="border: 1px solid #ddd; border-radius: 4px;">
  PDF를 표시할 수 없습니다. <a href="/pdf/05-mcp-integration_v1.0.pdf">다운로드</a>하세요.
</iframe>

> 📥 [PDF 다운로드](/pdf/05-mcp-integration_v1.0.pdf) · [전체 화면 보기](/pdf/05-mcp-integration_v1.0.pdf)

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

## ▶ 다음 단계

MCP로 OpenCode가 자료를 *능동적으로* 가져오는 라인을 확보했다면, 이제 *수동적으로* — 브라우징 중 즉석 캡처하는 라인을 추가할 차례입니다.

**다음 챕터**: [06. Web Clipper + LLM Proxy]({{< ref "06-web-clipper" >}}) — Docker proxy + Obsidian Web Clipper로 클립 시 자동 번역
