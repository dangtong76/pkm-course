---
title: "06. 콘텐츠 캡처 자동화 (Web + Video)"
weight: 6
date: 2026-05-10
draft: false
---

> **⏰ 시간**: 약 65분 (이론 15분 + 실습 50분 = Step 1: 15분 · Step 2: 13분 · Step 3: 12분 · Step 4: 10분, omo 위임 포함)
> **🎯 목표**: *입력 채널 자동화* — 영어 웹 글과 YouTube 영상을 *시청·복붙 없이* Vault Inbox로 끌어들이는 두 라인을 구축한다. ChatMock proxy(Step 1, omo 위임) → Web Clipper + 압축 번역 Template(Step 2~3) → YouTube transcript + omo 정리(Step 4).

---

## 📄 이론 슬라이드

> 이 챕터의 이론(Web Clipper 개념·LLM 통합 패턴·왜 proxy가 필요한가)은 아래 슬라이드로 다룹니다.

<div style="border: 2px dashed #cbd5e1; border-radius: 8px; padding: 48px 24px; text-align: center; background: #f8fafc; margin: 16px 0;">
  <div style="font-size: 48px; margin-bottom: 12px;">📊</div>
  <div style="font-size: 18px; font-weight: 600; color: #475569; margin-bottom: 8px;">
    슬라이드 PDF 준비 중
  </div>
  <div style="font-size: 14px; color: #64748b; margin-bottom: 16px;">
    이론 슬라이드는 별도 PDF로 제공될 예정입니다.<br>
    PDF 파일을 받으셨다면 <code style="background:#e2e8f0;padding:2px 6px;border-radius:4px;">static/pdf/06-web-clipper.pdf</code> 위치에 저장해 주세요.
  </div>
  <div style="font-size: 12px; color: #94a3b8;">
    PDF가 추가되면 이 자리에 자동으로 슬라이드가 표시됩니다.
  </div>
</div>

---

## 🧭 한 줄 요약

> **수집되지 않으면 검증할 것도 없다.** 영어 글은 *압축 번역된 채로*, 영상은 *자막 + omo 정리된 채로* Vault에 자동 진입한다. PKM 사이클의 *입력 단계 병목*이 해소되는 자리.

---

## 📚 학습 목표

이 챕터를 마치면 다음을 할 수 있습니다:

- [ ] **OpenAI 호환 LLM proxy** (ChatMock 기반, ChatGPT 구독 OAuth 활용)를 docker-compose로 띄울 수 있다
- [ ] **Obsidian Web Clipper** 브라우저 확장에 LLM endpoint를 연결해 글 클립 시 자동 번역·요약을 수행할 수 있다
- [ ] **YouTube Summary 확장**으로 영상 transcript를 텍스트로 캡처해 Vault에 적재할 수 있다
- [ ] 캡처된 raw 자료가 다음 챕터의 LLM Wiki `_Feed/`로 자연스럽게 흘러가도록 라우팅할 수 있다 (옵션)

---

## 🛠️ 사전 요구사항

| 항목 | 요구 사항 |
|------|---------|
| **OpenCode** | Chapter 01에서 설치 완료 |
| **AGENTS.md** | Chapter 04에서 GLOBAL + Vault AGENTS.md 작성 완료 |
| **MCP 연동** | Chapter 05에서 Exa + Firecrawl + Penpot MCP 연동 완료 |
| **Vault** | Chapter 03에서 만든 Vault |
| **Obsidian 1.8.0 이상** | Step 2-5의 *Download attachments for current file* 명령에 필요 (2024-12-18 릴리스). `Settings → About → Current version` 확인 |
| **Docker** | Docker Desktop / Colima / OrbStack 등 (compose v2 지원) |
| **API 키** | ChatGPT Plus / Pro / Max 구독 (1장 OAuth 로그인 완료) |
| **브라우저** | Chrome / Edge / Firefox / Safari 중 하나 (Web Clipper 확장 지원) |
| **YouTube Summary 확장** | Chrome 계열 권장 ([Chrome Web Store 설치 링크](https://chromewebstore.google.com/detail/youtube-summary-with-chat/nmmicjeknamkfloonkhhcjmomieiodli) — 무료, transcript 추출만 쓸 거면 API 키 불필요) |

---

## 🪜 실습 시나리오

<!-- TODO: 실습 단계 작성 -->

### Step 1. ChatMock Proxy를 docker-compose로 띄우기 (15분)

ChatGPT Plus/Pro/Max 구독을 OpenAI 호환 `/v1` endpoint로 expose하는 wrapper. 별도 OpenAI API 키 발급·과금 없이 1장에서 이미 로그인한 ChatGPT 계정으로 Web Clipper의 *Custom Provider*를 채운다.

> **🤖 본 Step의 핵심 시연**: 이 챕터는 본 강의에서 *처음으로 omo(OpenCode + Oh-My-OpenCode)에게 인프라 셋업을 위임*하는 자리입니다. 슬라이드 S3에서 다룬 *"AI가 손을 가진다"*가 docker-compose 셋업이라는 *실제 시스템 작업*에 적용됩니다 — 검증·승인은 당신이.

#### 사전 점검

| 항목 | 확인 방법 |
|---|---|
| Docker | `docker --version` + `docker compose version` (v2 이상) |
| ChatGPT 구독 | Plus / Pro / Max 중 1개 + 1장 `/connect` OAuth 완료 |
| 포트 8000 | `lsof -i :8000` 로 점유 프로세스 없음 (있으면 `.env`의 `PORT` 변경) |
| 인터넷 | `chatgpt.com` 접근 가능 |

> 📋 **검증 환경**
>
> | 항목 | 값 |
> |---|---|
> | 동작 확인 일시 | 2026-05-11 (KST) |
> | 검증 OS | macOS (darwin) — Linux/WSL2 도커 호환 |
> | Docker | Compose v2 (Desktop / OrbStack / Colima 호환) |
> | ChatMock image | `storagetime/chatmock:latest` (sha256:757c65c8…) |
> | 권장 모델 | `gpt-5.4` (실측 호출 가능) |
> | 테스트 기사 | https://en.wikipedia.org/wiki/Knowledge_management |
>
> 이 환경 외 동작은 검증되지 않음. ChatMock SHA·OpenAI backend API는 시간 흐름에 따라 변경 가능 — 실패 시 [ChatMock GitHub Issues](https://github.com/RayBytes/ChatMock/issues)에서 최신 상태 확인.

---

#### Plan A — omo 위임 흐름 (권장, 10분)

> **에이전트에게 맡기는 범위**: 파일 생성 · 디렉토리 이동 · `docker compose up` · 헬스체크 · 검증 `curl` 까지. **OAuth 로그인만 당신이 직접** (보안·OpenAI ToS·OAuth 표준 설계 모두 사람의 의식적 승인을 요구).

##### A-1. omo에게 위임 프롬프트 1개 (3분)

OpenCode TUI를 열고 다음 프롬프트를 *그대로* 입력:

```text
ChatMock proxy를 docker compose로 ~/AI-Workstation/chatmock-selfhost 에 셋업해줘.

요구사항:
1. 작업 디렉토리: ~/AI-Workstation/chatmock-selfhost (없으면 생성)
2. 다음 docker-compose.yml을 그대로 작성:

```yaml
services:
  chatmock:
    image: ${CHATMOCK_IMAGE:-storagetime/chatmock:latest}
    container_name: chatmock
    command: ["serve"]
    env_file:
      - path: .env
        required: false
    environment:
      - CHATGPT_LOCAL_HOME=/data
      - CHATGPT_LOCAL_REASONING_EFFORT=${CHATGPT_LOCAL_REASONING_EFFORT:-medium}
    ports:
      - "${PORT:-8000}:8000"
    volumes:
      - chatmock_data:/data
    healthcheck:
      test: ["CMD-SHELL", "python -c \"import urllib.request,sys; sys.exit(0 if urllib.request.urlopen('http://127.0.0.1:8000/health').status==200 else 1)\""]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

  chatmock-login:
    image: ${CHATMOCK_IMAGE:-storagetime/chatmock:latest}
    profiles: ["login"]
    command: ["login"]
    environment:
      - CHATGPT_LOCAL_HOME=/data
      - CHATGPT_LOCAL_LOGIN_BIND=0.0.0.0
    volumes:
      - chatmock_data:/data
    ports:
      - "1455:1455"

volumes:
  chatmock_data:
```

3. 빈 .env 파일 1개 생성 (기본값 사용)
4. `docker compose up -d` 로 serve 컨테이너 기동
5. 12초 대기 후 `docker compose ps` 결과 보여줘 — STATUS가 healthy인지 확인
6. ⚠️ OAuth 로그인 단계(`docker compose run --rm --service-ports chatmock-login`)는 *내가 직접 할 거니까 실행하지 마*. 명령만 출력하고 멈춰.

ultrawork. 단계별로 보여주고 각 명령 실행 전 승인 요청.
```

> **💡 왜 한 프롬프트인가**: omo는 *전체 컨텍스트가 있을 때* 더 정확하게 동작합니다. 단계를 쪼개면 매번 *"이전에 어디까지 했는지"* 를 재설명해야 함. 위 프롬프트 1개에 *목표·파일 내용·실행 명령·금지 사항*이 모두 들어있어 omo가 한 번에 시퀀스를 짭니다.

##### A-2. omo의 실행을 따라가며 승인 (3분)

omo가 다음 순서로 진행하며 *각 단계마다 승인을 요청*합니다:

1. `mkdir -p ~/AI-Workstation/chatmock-selfhost` — **승인**
2. `docker-compose.yml` 파일 생성 — **승인** (내용을 한 번 훑고)
3. `.env` 빈 파일 생성 — **승인**
4. `cd` + `docker compose up -d` — **승인**
   - 첫 실행 시 이미지 pull로 1~2분 (네트워크 환경 따라)
5. `sleep 12 && docker compose ps` — **승인**

기대 출력 (5번 단계 후):

```text
NAME       IMAGE                          STATUS                  PORTS
chatmock   storagetime/chatmock:latest    Up X seconds (healthy)  0.0.0.0:8000->8000/tcp
```

> **✨ Aha moment #1**: 슬라이드 S2가 *"마무리 = 100% 사람 → 90% AI + 10% 취향"* 이라 말한 그 90%가 docker-compose 셋업입니다. *당신이 키보드로 친 명령은 0개*, 검증·승인 클릭만 5번. 검증 대역폭이 *입력 시간*이 아니라 *읽고 승인하는 시간*으로 바뀌었습니다.

##### A-3. OAuth 로그인 (사람 직접, 3분)

OAuth는 *반드시 당신이 직접* 수행합니다. omo에게 시키지 않는 이유:

| 이유 | 설명 |
|---|---|
| **자격증명 보호** | ChatGPT 비밀번호·MFA·생체인증은 *당신만의 것* — 에이전트가 다룰 수 없는 영역 |
| **BAN 위험 가속** | OAuth를 자동화하면 OpenAI의 *비정상 트래픽 탐지*를 정면으로 트리거 (Step 1 🚨 박스의 위험이 격발) |
| **OAuth 표준 설계** | OAuth는 *사람의 의도 명시 승인*을 전제로 설계된 보안 프로토콜 — 자동화는 보안 모델 위반 |
| **슬라이드 S5 메시지** | *"AI가 호출할 때마다 당신이 승인한다"* — OAuth는 그 승인의 *원형* |

omo가 멈춘 자리에서 *당신이 직접* 실행:

```bash
cd ~/AI-Workstation/chatmock-selfhost
docker compose run --rm --service-ports chatmock-login
```

진행 순서:
1. 터미널에 OAuth 인증 URL이 출력됨 → 브라우저에 복사·붙여넣기
2. ChatGPT 계정 로그인 → 권한 승인
3. 브라우저가 `localhost:1455` 콜백으로 리디렉션 → 컨테이너가 토큰 받음
4. 터미널에 성공 메시지 출력 → 컨테이너 자동 종료

> **⚠️ 회사·학교 네트워크에서 실패할 때**: 브라우저가 `localhost:1455` 에 접근 못 하면 브라우저 주소창의 전체 URL을 그대로 복사해 터미널에 붙여넣기. 컨테이너가 그 URL의 query param에서 토큰을 추출합니다.

토큰은 Docker volume `chatmock_data:/data/auth.json` 에 영속 저장됩니다. 컨테이너 재시작·재기동 후에도 유지됨.

##### A-4. omo에게 검증 위임 (1분)

OAuth 완료 후 OpenCode TUI로 돌아와 다음 프롬프트:

```text
ChatMock OAuth 로그인 완료했어. 다음을 검증해줘:

1. http://localhost:8000/v1/chat/completions 에 model=gpt-5.4 로
   "한국어로 ping이라고만 답해주세요" curl 호출
2. 응답 JSON의 choices[0].message.content 보여줘
3. http://localhost:8000/v1/models 에서 model id 목록도 같이 가져와줘

ultrawork.
```

omo가 다음 두 curl을 실행하고 결과를 정리해줍니다:

```bash
# 검증 1
curl -sf -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer dummy" \
  -d '{"model":"gpt-5.4","messages":[{"role":"user","content":"한국어로 ping이라고만 답해주세요"}]}' \
  | jq .

# 검증 2
curl -s http://localhost:8000/v1/models | jq -r '.data[].id'
```

기대 응답:

```json
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "핑",
        "role": "assistant"
      }
    }
  ],
  "model": "gpt-5.4",
  "object": "chat.completion"
}
```

응답 시간은 평균 2~6초. 모델 목록은 `gpt-5`, `gpt-5.1`, `gpt-5.2`, `gpt-5.4`, `gpt-5.3-codex` 등 11개. **단 ChatGPT 구독 등급에 따라 일부 모델은 호출 시 HTTP 400 `Upstream error` 를 반환** — `gpt-5.4` 가 가장 안정적.

> **💡 `Authorization: Bearer dummy` 헤더**: ChatMock은 헤더의 API 키 값을 검사하지 않음 (실제 인증은 컨테이너 내 OAuth 토큰으로 수행). Web Clipper에서도 API Key 필드에 임의 값을 넣으면 됩니다.

> **✨ Aha moment #2**: 인프라 셋업(*Plan A의 A-1~A-4*) 전체에서 *당신이 직접 타이핑한 명령*은 OAuth `docker compose run` 한 줄뿐. 나머지는 모두 omo + 승인 클릭. *손이 가는 자리*가 *판단·검증의 자리*로 정확히 이동했습니다.

---

#### Plan B — 수동 흐름 (Fallback)

omo가 실패하거나 *직접 단계를 따라가고 싶을 때* 사용. Plan A와 *결과가 100% 동일*하며, 한 명령씩 직접 입력합니다.

```bash
# 1. 작업 디렉토리
mkdir -p ~/AI-Workstation/chatmock-selfhost
cd ~/AI-Workstation/chatmock-selfhost

# 2. docker-compose.yml 작성 (위 Plan A의 docker-compose.yml 내용을 그대로 저장)
nano docker-compose.yml   # 또는 vim / VSCode

# 3. .env 빈 파일
touch .env

# 4. 기동
docker compose up -d
sleep 12
docker compose ps

# 5. OAuth 로그인 (Plan A의 A-3와 동일)
docker compose run --rm --service-ports chatmock-login

# 6. 검증 (Plan A의 A-4 curl 두 개 그대로)
curl -sf -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer dummy" \
  -d '{"model":"gpt-5.4","messages":[{"role":"user","content":"한국어로 ping이라고만 답해주세요"}]}' \
  | jq .
```

> **💡 어느 흐름이 PKM 강의 가치를 더 신체화하는가**: Plan A. 슬라이드 throughline(*"AI가 직접 손을 가진다 + 검증 대역폭이 결정점"*)을 *셋업 단계*부터 체화합니다. Plan B는 이론은 같지만 *그 메시지는 다음 step에서 늦게 도착*합니다.

> 🚨 **반드시 읽고 결정하세요 — 사용자 책임**
>
> ChatMock은 OpenAI 공식 제품이 아닙니다. ChatGPT 구독 OAuth 토큰으로 OpenAI 내부 endpoint(`chatgpt.com/backend-api/codex`)를 호출하는 reverse-proxy로, **OpenAI Services Agreement §3.3(h)·(i) "rate limit·보호 조치 우회 금지" 조항이 적용될 수 있는 회색지대**입니다.
>
> 메인테이너 본인이 README에서 경고합니다:
>
> - *"Use responsibly and at your own risk. This project is not affiliated with OpenAI."*
> - *"This is an unofficial wrapper. Your account may be subject to action by OpenAI."*
> - 출처: <https://github.com/RayBytes/ChatMock>
>
> 즉 — **OpenAI가 비정상 트래픽을 감지하면 본인 ChatGPT 구독 계정이 정지될 수 있습니다.** 자기 책임 하에 결정하세요.
>
> 강사도 동일 경로를 본인 책임으로 사용 중이며 같은 위험을 감수합니다. *"이런 방법도 있다"* 는 정보 제공이며, 강제 아님. 일상은 1장 옵션 1·2를 우선 사용하고 본 6장 Web Clipper 시나리오에서만 ChatMock을 씁니다.

**무엇이 회색지대인가** (투명성):

- ChatGPT 구독 OAuth 토큰은 본래 ChatGPT 클라이언트용으로 발급된 자격.
- ChatMock은 그 토큰을 *"OpenAI 호환 `/v1/chat/completions`"* 형태로 재포장(wrap).
- 토큰의 *원래 목적*과 *실제 사용 형태*가 어긋나 OpenAI가 ToS 위반으로 해석할 가능성 있음.
- BAN 사례가 광범위하게 보고되진 않았으나, 정책·탐지 로직은 언제든 변경 가능.
- <https://github.com/RayBytes/ChatMock/issues> 에서 *"401 unauthorized"*·*"account flagged"* 키워드로 다른 사용자 사례 확인 가능. 사용 전 한 번 훑어보세요.

**위 위험을 감수하면** 진행, 거부 시 아래 대안 중 선택.

**대안 경로** (위험 감수 거부 시):

- **Anthropic Claude (Web Clipper native)** — Step 2에서 *Anthropic preset* + Claude API 키(console.anthropic.com). ToS 합법, BAN 가능성 거의 없음. 글 1편당 \$0.001~\$0.01 수준 종량제.
- **Ollama 로컬 모델** — *Ollama preset* + `ollama run llama3.1`. 비용 0원, ToS 합법. 로컬 GPU/CPU 자원 필요, 번역 품질은 ChatGPT 대비 낮을 수 있음.
- **정식 OpenAI API 키** — <https://platform.openai.com> 종량제 키 + Web Clipper *OpenAI preset*. 가장 안정·합법. ChatGPT 구독과 별개 결제, 글 50편/월 가정 \$0.5 미만.

**감수 시 운영 권고** (BAN 가능성 최소화):

- **호출 빈도**: 1~2분당 1회 이하, 사람이 직접 클리핑하는 페이스 유지.
- **연속 호출 금지**: 5회 후 10분 이상 휴식.
- **모델 선택**: `gpt-5.4` 등 텍스트 모델만. 이미지·음성 모델 회피.
- **로그 모니터링**: `docker compose logs -f chatmock` 으로 401·403·429 추이 확인, 발생 시 즉시 중단.
- **백업 준비**: 정식 OpenAI API 키 또는 Anthropic API 키 preset을 Web Clipper에 미리 저장.
- **민감 콘텐츠 우회**: 사내 비공개·미공개 자료는 ChatMock 경로로 처리하지 말 것 — Ollama 로컬 또는 데이터 보호 옵션 켠 정식 API 키만 사용.
- **단기 운영**: 강의 실습·단기 프로젝트로 한정, 장기 상시 운영은 정식 API 키로 전환.
- **이상 징후 공유**: 401·429 급증 또는 ChatGPT 웹 로그인 거부 시 강의 채널에 즉시 공유 — OpenAI 정책 변경 신호일 수 있음.

> ⚠️ **자동화 트래픽 자제 권고**
>
> ChatMock은 OAuth 토큰을 그대로 쓰므로 단기 다량 호출은 자동화 탐지에 걸리기 쉽습니다. **omo의 `ultrawork`·Ralph Loop 등 무한 루프는 ChatMock + Web Clipper에 적용 금지.** 일반 흐름(글 1편 → 확인 → 다음 글)은 안전 트래픽이지만, 자동 스크립트로 묶어 돌리는 패턴은 위험. *"언제든 계정 잃을 수 있다는 전제로"* 백업 결제 수단·대안 경로를 준비하세요.

**문제 발생 시 대응**:

- **401 Unauthorized 반복**: 토큰 무효화 신호. `docker compose down` + ChatMock 중단, 대안 경로로 전환. 재로그인 시도는 비권장.
- **429 Too Many Requests**: rate limit. 호출 간격 늘리고 1시간 휴식. 반복 시 패턴 노출됐을 가능성.
- **ChatGPT 웹 로그인 거부**: 계정 정지 신호. 백업 계정·정식 API 키로 즉시 전환 (소명 가능하나 복구 보장 X).
- **컨테이너 자동 재시작 비활성화**(`restart: "no"`) — 무효 토큰 자동 재호출이 추가 탐지 트리거.

> 💡 **안전 경로는 옵션 1·2 또는 정식 API 키**
>
> - 1장 옵션 1 (ChatGPT OAuth + OpenCode CLI)은 OpenAI 공식 지원 경로 — opencode CLI 내 사용은 ToS 합법. Web Clipper 시나리오만 회색지대.
> - 1장 옵션 2 (OpenCode Zen)도 합법 — 별도 토큰 멀티벤더, BAN 가능성 사실상 없음.
> - 두 경로는 opencode CLI가 주체라 외부 endpoint로 노출되지 않음. Web Clipper에서는 정식 OpenAI/Anthropic API 키 사용을 우선 검토하세요.
> - 본 step에서 ChatMock을 쓰더라도 일상은 옵션 1·2 또는 정식 API 키로 회귀 권장.

#### 다음 Step 안내

이로써 ChatMock proxy가 `http://localhost:8000/v1/chat/completions` 에서 OpenAI 호환 응답을 제공합니다. **Step 2** 에서 Obsidian Web Clipper의 *Custom Provider*에 이 endpoint를 등록하고 실제 영문 기사 → 한국어 번역+요약 클리핑을 진행합니다.

### Step 2. Obsidian Web Clipper 설치 + ChatMock 연결 + 이미지 경로 분리 (13분)

> **목표**: 브라우저에서 보는 영어 콘텐츠를 *클립 한 번*에 한국어로 번역해 Vault Inbox로 보내는 라인을 구축한다. Step 1에서 띄운 ChatMock proxy(`http://localhost:8000`)가 LLM 백엔드 역할을 한다.

#### 2-1. Web Clipper 확장 설치 (2분)

Obsidian이 직접 만든 *공식 확장* 입니다 (Obsidian/Dynalist Inc., MIT 오픈소스). 별도 계정·가입 없음.

Chrome / Brave / Edge:

<https://chromewebstore.google.com/detail/obsidian-web-clipper/cnjifjpddelmedmihgijeibhnjfabmlf>

Firefox / Safari / 기타: <https://obsidian.md/clipper> 에서 본인 브라우저용 링크 클릭.

설치 후 브라우저 우측 상단에 Obsidian 아이콘이 보이면 OK. 단축키 권장:

- macOS: `Cmd + Shift + O`
- Windows / Linux: `Ctrl + Shift + O`

> **💡 Obsidian 앱이 실행 중이어야 합니다**. Web Clipper는 `obsidian://` URI 스킴 + 클립보드 조합으로 노트를 만듭니다 — Obsidian 앱이 그 URI를 받아 처리해야 동작.

#### 2-2. Vault 연결 (2분)

브라우저 툴바의 Obsidian 아이콘 우클릭 → **Options**(또는 좌측 톱니바퀴) → **General** 패널.

1. **Vaults** 섹션에서 **Add vault** 클릭
2. **Vault name** 필드에 *Obsidian Vault 이름*을 정확히 입력 (예: `dangtong-note`)
   - ⚠️ 경로가 아니라 *이름*입니다. Obsidian 사이드바 좌측 상단에 보이는 이름.
3. **Default folder** 에 `01-Inbox/` 입력 (끝의 `/` 필수)

저장 시 Web Clipper가 호출하는 URI 형식 (참고):

```
obsidian://new?file=01-Inbox%2F<note-name>&vault=dangtong-note&clipboard
```

#### 2-3. Interpreter 활성화 + ChatMock Provider 등록 (4분)

Obsidian 아이콘 우클릭 → **Options** → **Interpreter** 탭으로 이동.

1. **Enable Interpreter** 토글을 ON으로
2. **Add provider** 클릭 → 다음 정보 입력:

   | 필드 | 값 |
   |---|---|
   | **Provider Name** | `ChatMock` |
   | **Base URL** | `http://localhost:8000/v1/chat/completions` |
   | **API Key Required** | OFF (또는 비활성) |
   | **API Key** | `dummy` (Web Clipper UI가 키를 요구하면 임의값) |

   > **⚠️ Base URL은 `/v1/chat/completions`까지 포함합니다**. `/v1`만 입력하면 동작 안 함. 공식 문서: *"use their chat completions endpoint for the Base URL — it typically ends with `/chat/completions`."*

3. **Add model** 클릭:

   | 필드 | 값 |
   |---|---|
   | **Provider** | `ChatMock` (위에서 만든 것) |
   | **Display Name** | `ChatMock GPT-5.4` |
   | **Model ID** | `gpt-5.4` |

   > **💡 모델 ID 정합성**: Step 1의 `curl http://localhost:8000/v1/models | jq -r '.data[].id'` 로 확인한 모델 중 *가장 안정적인 것*을 골랐습니다. 다른 모델도 등록 가능하지만 일부는 ChatGPT 구독 등급 따라 400 에러를 냄.

#### 2-4. 연결 검증 (2분)

영어 웹 페이지 아무거나 열고 (예: <https://en.wikipedia.org/wiki/Knowledge_management>) 브라우저 아이콘 클릭하거나 단축키. 팝업에서:

1. 우측 상단 **Template** 드롭다운에 *기본 Template*이 보임
2. 하단 **Interpret** 버튼을 클릭

응답이 30초 이내에 돌아오면 ChatMock 연결 성공. 아무 메시지 없이 무한 대기하면:

| 증상 | 원인 | 해결 |
|---|---|---|
| 즉시 에러 | Base URL 오타 | `/v1/chat/completions` 끝까지 정확히 |
| Timeout | ChatMock 미기동 | `docker compose ps`로 healthy 확인 |
| 401/403 | Step 1의 OAuth 토큰 무효 | Step 1의 *위험 감수 운영 권고* 박스 참고. 대안 경로 검토 |
| CORS 에러 (콘솔) | Manifest V3 정책 | ChatMock이 이미 `chrome-extension://*` 허용 — 다른 확장이 간섭 중일 수 있음. 시크릿 창에서 재시도 |
| **`thinking 80s+`로 지연** | Template에 *전체 번역* prompt가 있어 출력 토큰 폭증 | Step 3-1의 *압축 번역* Template 사용. 또는 Template 번역 prompt에 `"Maximum 1500 words"` 같은 길이 제한 추가 |

> **🚨 Step 1의 위험 감수 박스 재확인**: 본 step부터 *ChatMock + 브라우저 자동 클립*은 자동화 트래픽으로 오인될 수 있습니다. 1~2분당 1회 이하 페이스 유지, 5회 후 휴식 권장.

#### 2-5. 이미지 저장 경로 분리 (3분)

> **왜 필요한가**: Web Clipper 기본 동작은 *이미지를 다운로드하지 않고 외부 URL 링크만* 노트에 박는 것입니다. 나중에 *원격 이미지가 깨지는 위험*을 피하려면 별도 명령으로 다운로드해야 하는데, 그 시점에 *이미지가 어디로 저장될지*는 **Web Clipper가 아니라 Obsidian 본체 설정**이 결정합니다. 강의 Vault 컨벤션은 *노트 = 01-Inbox/, 이미지 = 99-Attachments/* 분리이므로 본 단계에서 그 설정을 고정합니다.

**Obsidian 본체 설정 변경**:

1. Obsidian 좌측 하단 ⚙️ 톱니바퀴 → **Settings**
2. 좌측 메뉴 **Files & links** (또는 *Files and links*) 클릭
3. **Default location for new attachments** 항목 찾기 (드롭다운 4가지 옵션 노출)

   | 옵션 | 동작 |
   |---|---|
   | Vault folder | 모든 첨부를 Vault 루트에 |
   | **In the folder specified below** ⭐ | *지정한 폴더* 한 곳에 (이번에 선택) |
   | Same folder as current file | 노트와 같은 폴더에 (Web Clipper 기본 경험) |
   | In subfolder under current folder | 노트 폴더 하위 서브폴더에 |

4. **In the folder specified below** 선택
5. 아래 *Attachment folder path* 필드에 입력:

   ```
   99-Attachments/web-clipper-images
   ```

6. 설정 화면 닫기 (자동 저장)

이제 클립 후 *Download attachments* 명령으로 가져오는 모든 이미지는 `99-Attachments/web-clipper-images/`로 들어갑니다. 노트 파일은 그대로 `01-Inbox/`에 남음.

> **💡 Web Clipper 측 설정이 없는 이유**: 공식 문서 [help.obsidian.md/web-clipper/capture](https://help.obsidian.md/web-clipper/capture) — *"Images are not automatically downloaded when you use Web Clipper. Instead, images link to their web-based URL."* Web Clipper Template에는 *이미지 저장 경로* 필드 자체가 없습니다. Obsidian 본체 설정이 진실 공급원.

> **⚠️ 사전 요구**: Obsidian **1.8.0 이상**이어야 *Download attachments for current file* 명령이 존재합니다 (2024-12-18 릴리스). `Settings → About → Current version` 으로 확인. 이전 버전이면 업데이트 필요.

#### 2-6. 99-Attachments 폴더 사전 생성 (30초)

Obsidian이 이미지 다운로드 시 폴더가 없으면 자동 생성하지만, *경로 컨벤션*을 미리 박아두는 게 깔끔합니다.

Obsidian 좌측 파일 트리에서:

1. Vault 루트 우클릭 → **New folder**
2. 이름: `99-Attachments`
3. `99-Attachments` 우클릭 → **New folder**
4. 이름: `web-clipper-images`

결과 폴더 구조:

```
<Vault>/
├── 01-Inbox/             # 노트 파일
└── 99-Attachments/
    └── web-clipper-images/   # Web Clipper 클립 이미지
```

### Step 3. 번역 Template 작성 + 영어 글 실전 클립 (12분)

> **목표**: 영어 기술 글 1편을 Inbox에 *한국어로 번역된 상태*로 자동 저장. Template 1개를 만들고 평생 재사용한다.

#### 3-1. Template 신규 작성 (5분)

Web Clipper **Options** → **Templates** 탭 → 좌측 하단 **+ Add new template**.

기본 Template이 생성되면 다음 6개 필드를 채웁니다.

**Template Name**: `Translate to Korean`

**Behavior**: `Create new note` (드롭다운)

**Vault**: `dangtong-note` (Step 2-2에서 등록한 이름)

**Path / Folder**: `01-Inbox/`

**Note name**:

```
clip-{{date|date:"YYYY-MM-DD"}}-{{title|safe_name}}
```

**Properties** (각 행마다 *+ Add property*):

| Key | Value |
|---|---|
| `source` | `{{url}}` |
| `captured` | `{{date|date:"YYYY-MM-DD HH:mm"}}` |
| `original_language` | `en` |
| `translated_by` | `ChatMock GPT-5.4` |
| `tags` | `clip/web/en→ko` |

**Note content** (본문 영역):

```
# {{title}}

> **원문**: <{{url}}>
> **저자**: {{author}}
> **발행일**: {{published}}

---

## 🇰🇷 한국어 요약 (3~5줄)

{{"이 글의 핵심을 한국어로 3~5개의 bullet point로 요약해줘. 각 bullet은 한 줄."}}

---

## 🇰🇷 한국어 번역 (압축)

{{"이 글을 한국어로 *압축 번역*해줘. 규칙:
- 마크다운 구조(H2/H3/bullet/code block)는 그대로 유지
- 각 문단은 *핵심만* 한두 문장으로 압축 — 직역 X
- 기술 용어는 영어 원어 + 한국어 괄호 병기 (첫 등장 시만)
- **전체 출력 최대 1500 단어**"}}

---

## 🔖 원문 (English)

{{content}}
```

저장 (우측 상단 disk 아이콘 또는 자동 저장).

> **💡 Prompt 문법 핵심**: `{{"..."}}` — *큰따옴표로 감싼 문자열 자체*가 LLM 호출 신호입니다. 큰따옴표 없는 `{{title}}` / `{{url}}` / `{{content}}`은 *변수*(페이지에서 자동 추출). Web Clipper는 page HTML을 ChatMock에 보내면서 *"이 prompt들에 답하라"*고 한 번에 묶어서 요청합니다.

> **⚡ 왜 *압축 번역*인가 — 출력 토큰이 응답 시간을 결정한다**: ChatMock + GPT-5.4로 *전체 번역*을 요구하면 출력 토큰이 5000+로 늘어나 응답이 80~90초 걸립니다. *각 문단을 핵심만*으로 압축하면 출력 1500 단어 이내로 줄어 응답 시간이 **30~50초로 단축**됩니다. PKM 관점에서도 자연스러움: *원문은 검증·인용용으로 보존*(아래 🔖 섹션), *한국어는 핵심만 빠르게*. 슬라이드 S2의 *Last-mile Finisher*(90% AI 압축 + 10% 당신의 깊이 읽기) 패턴.

#### 3-2. 영어 글 실전 클립 (5분)

영어 기술 글 1개를 브라우저에서 열기. 예시 후보:

- <https://maggieappleton.com/garden-history> (PKM 영역의 *"디지털 가든"* 영어 원전)
- <https://fortelabs.com/blog/para/> (PARA 원전)
- <https://karpathy.github.io/2025/01/03/llm-os/> (Karpathy LLM OS)

진행:

1. 우측 상단 Obsidian 아이콘 클릭 (또는 `Cmd/Ctrl + Shift + O`)
2. Template 드롭다운에서 **Translate to Korean** 선택
3. *Properties* 미리보기에서 `source`·`captured`·`tags`가 자동으로 채워졌는지 확인
4. *Note content* 영역에 `{{"..."}}` 자리가 보이면 — 아직 빈 칸. 하단 **Interpret** 버튼 클릭
5. **30~50초 후** *요약*과 *번역* 자리가 한국어로 채워짐. 진행 상태는 popup 하단 `ChatMock GPT-5.4 thinking XX.YYs` 카운터로 보임
6. 결과가 만족스러우면 우측 하단 **Add to Obsidian** 클릭
7. **이미지 다운로드** — Obsidian으로 돌아와 방금 생성된 노트를 열고:
   - Command Palette 열기 (`Cmd/Ctrl + P`)
   - `Download attachments for current file` 입력 → 실행
   - 노트의 외부 URL 이미지가 모두 `99-Attachments/web-clipper-images/`로 다운로드되고, 마크다운 링크가 자동으로 *로컬 경로*로 교체됨

Obsidian 앱이 포커스를 받고 `01-Inbox/clip-2026-05-14-Knowledge_management.md` 같은 파일이 생성됩니다. 7번 단계 후에는 *오프라인에서도* 이미지가 깨지지 않습니다.

> **💡 자동화 옵션**: 매번 명령어를 치기 귀찮다면 Obsidian 단축키에 *Download attachments for current file*을 매핑(`Settings → Hotkeys`)하거나, 커뮤니티 플러그인 [obsidian-auto-download-imgs-after-web-clipping](https://github.com/chenxiccc/obsidian-auto-download-imgs-after-web-clipping)으로 클립 직후 자동 실행 가능. 강의 기본은 *수동 명령*으로 *AI 호출과 같은 승인 게이트*를 유지.

> **⏱️ 응답 시간이 80초+로 길어진다면**: Template의 번역 prompt가 *전체 번역*을 시키고 있을 가능성. Step 3-1의 *압축 번역* Template(`"각 문단을 핵심만"` + `"Maximum 1500 words"`)을 그대로 사용했는지 확인하세요. 또는 더 짧은 글(1000단어 이내)로 먼저 흐름 익히기.

#### 3-3. 결과 노트 검증 (2분)

생성된 노트를 열어 다음 6가지 확인:

- [ ] Properties (frontmatter): `source` / `captured` / `original_language` / `translated_by` / `tags` 모두 채워짐
- [ ] H1 제목이 원문 제목 그대로
- [ ] 🇰🇷 요약 섹션: 3~5개 한국어 bullet
- [ ] 🇰🇷 번역 섹션: 자연스러운 한국어 본문, 기술용어 영어 병기
- [ ] 🔖 원문 섹션: 영문 원본 markdown 보존 (검증·인용용)
- [ ] **이미지** (있다면): `99-Attachments/web-clipper-images/`에 다운로드됨 + 노트의 마크다운 링크가 *로컬 경로*로 교체됨 (`![](https://...)` → `![](99-Attachments/web-clipper-images/...)`)

> **✨ Aha moment**: 영어 글을 *보면서* 클립 한 번. *읽는 동안* AI가 한국어로 번역해 Vault에 *완성된 상태로* 들어옵니다. ChatGPT 탭 따로 열고 → 본문 복사 → 붙여넣기 → "번역해줘" → 결과 복사 → Obsidian에 붙여넣기 — 6단계 마찰이 단축키 한 번으로 줄었습니다. 슬라이드 S1의 *"복붙이 사라진다"*가 *입력 채널*까지 확장됐습니다.

#### 3-4. Template 재사용·확장 팁

- **다른 언어**: prompt의 `한국어`만 바꾸면 일본어·중국어 번역 Template으로 즉시 재사용
- **번역 분량 조정**: 번역 prompt 끝의 `"Maximum 1500 words"`를 늘리거나 줄여 응답 시간·정보량 trade-off 직접 조절. *500단어 → 더 빠름·핵심만* / *3000단어 → 더 느림·자세함*
- **요약 길이 조정**: 요약 prompt에 *"5~10줄로"* 또는 *"한 단락으로"* 같은 길이 지시 추가
- **태그 자동 분류**: properties에 `tags` 행 하나 더 추가하고 `{{"이 글에 어울리는 Obsidian 태그 3개를 #foo/bar 형식으로 콤마 구분"}}` 같은 prompt 사용
- **여러 모델 비교**: Step 2-3에서 다른 모델을 추가 등록해두면 Template마다 *Interpreter Settings → Default Model* 다르게 지정 가능
- **원문 검증 패턴**: `## 🔖 원문 (English)` 섹션을 그대로 두면 *압축 번역의 정확성*을 원문과 즉시 대조 가능 — PKM의 *검증 대역폭* 패턴 그대로 구현됨

### Step 4. YouTube 영상 transcript를 Vault 노트로 만들기 (10분)

> **목표**: 영상을 *처음부터 끝까지 보지 않고도* 자막을 Vault에 텍스트로 가져와 정리한다. *보는 시간*을 *읽고 검증하는 시간*으로 바꾼다 — Step 1~3의 *복붙 사라짐*에 이은 *시청 시간 자체의 압축*.

#### 4-1. YouTube Summary 확장 설치 (2분)

Glasp가 만든 Chrome 확장. 무료, 가입 불필요, *transcript 추출만* 사용할 거라 API 키도 필요 없음.

설치 링크: <https://glasp.co/youtube-summary>

(또는 Chrome Web Store에서 *"YouTube Summary with ChatGPT & Claude"* 검색 — Glasp Inc. 개발자)

설치 후 YouTube 영상 페이지를 열면 우측 상단에 *YouTube Summary* 패널이 자동으로 나타납니다.

#### 4-2. 자막 복사 (1분)

영상 1개 열고 — 본인 도메인 관심사 영상이 좋습니다. 예시 후보:

- 짧은 인터뷰·강연 (10~20분) — 정리하기 쉬움
- 본인 업계 발표·콘퍼런스 톡 — 즉시 PKM 가치
- 한국어 또는 영어 영상 둘 다 가능 (한국어는 자동 생성 자막도 인식)

진행:

1. 영상 페이지에서 우측 상단 **YouTube Summary** 패널 클릭해 펼치기
2. 상단 탭 두 개 중 **Transcript** 탭 (좌측) 선택
3. 그 아래 *언어 선택 버튼* 확인 — 보통 `한국어 (자동 생성됨)` 또는 영상 원어가 기본
4. 우측 상단 *복사 아이콘* (📋, 또는 두 사각형이 겹친 아이콘) 클릭 — 클립보드에 전체 transcript 복사됨
5. 짧은 토스트 *"Copied"* 가 보이면 성공

> **💡 Transcript 패널 구조**: 좌측 `Transcript` 탭이 *자막 원본 텍스트*, 우측 `Summary` 탭은 ChatGPT/Claude API 키를 연결해야 동작하는 *요약 기능*입니다. **본 실습은 Transcript만 씁니다** — 요약은 다음 4-4 단계에서 *우리가 만든 omo*에게 위임하는 게 훨씬 유연.

#### 4-2.5. (대안 길) Web Clipper로 같은 영상을 통째 클리핑 (3분)

> **왜 두 길을 모두 다루나**: 4-2 → 4-3은 *transcript 텍스트*를 손에 쥐는 길입니다. 정확히 자막 본문을 raw로 보존하고 싶을 때 강합니다. 반면 *Web Clipper 길*은 *영상 메타데이터(title·author·description·date·url)가 properties로 자동 채워지는* 장점이 있습니다 — 강의 Step 2에서 이미 깐 도구를 *영상에도* 재사용. 두 길의 결과 노트는 4-4 omo 정리에 *똑같이* 들어갑니다. 본인 워크플로에 어느 쪽이 맞는지 직접 체감해보세요.

##### 4-2.5-1. YouTube 전용 Template 신규 작성 (2분)

Step 3-1에서 만든 `Translate to Korean` Template은 *번역 prompt*가 들어있어 영상에는 부적합합니다 (응답 시간 길고 자막 압축 번역은 의미 흐려짐). 영상 전용 Template을 1개 더 만듭니다 — LLM 호출 없는 *순수 메타데이터 캡처* Template이라 응답이 즉시 끝납니다.

Web Clipper **Options** → **Templates** → 좌측 하단 **+ Add new template**.

| 필드 | 값 |
|---|---|
| **Template Name** | `Capture YouTube` |
| **Behavior** | `Create new note` |
| **Vault** | `dangtong-note` (Step 2-2와 동일) |
| **Path / Folder** | `01-Inbox/` |
| **Note name** | `yt-{{date\|date:"YYYY-MM-DD"}}-{{title\|safe_name}}` |

**Properties**:

| Key | Value |
|---|---|
| `source` | `{{url}}` |
| `captured` | `{{date\|date:"YYYY-MM-DD HH:mm"}}` |
| `type` | `youtube-transcript` |
| `language` | `ko` |
| `author` | `{{author}}` |
| `published` | `{{published}}` |
| `tags` | `clip/youtube/raw` |

**Trigger** (선택 사항, 자동 매칭): `https://(www\.)?youtube\.com/watch.*` — 정규식 매칭. YouTube 페이지에서 단축키 누를 때 이 Template이 *자동 선택*됩니다.

**Note content**:

```
# {{title}}

> **채널**: {{author}}
> **공개일**: {{published}}
> **원본 URL**: <{{url}}>

---

## 📝 설명 (영상 description)

{{description}}

---

## 🎬 페이지 본문 (raw, Web Clipper 추출)

{{content}}
```

저장.

> **💡 왜 LLM prompt가 없나**: 영상 페이지의 `{{content}}`에는 *YouTube Summary 확장이 띄운 transcript 패널*까지 같이 잡혀 들어오는 경우가 많습니다. 게다가 영상 description은 *원작자가 직접 쓴 문장*이라 LLM 압축이 오히려 정보를 잃게 만듭니다. *raw 보존 → omo가 정리*가 4-4와도 일관됩니다.

##### 4-2.5-2. 영상 페이지에서 클리핑 (1분)

4-2와 *같은 영상 페이지*를 그대로 유지한 상태에서:

1. 브라우저 단축키 (`Cmd/Ctrl + Shift + O`) 또는 Obsidian 아이콘 클릭
2. Template 드롭다운 — Trigger 정규식을 넣었다면 `Capture YouTube`가 *자동 선택*됨. 아니면 수동으로 선택
3. *Properties* 미리보기 확인:
   - `title`은 영상 제목으로 채워짐
   - `author`는 채널명
   - `published`는 영상 공개일
   - `url`은 영상 URL (`watch?v=...` 형태)
   - `description`은 영상 설명문
4. **Interpret 버튼은 누르지 않음** — 이 Template에는 LLM prompt가 없어 누를 필요가 없습니다. 바로 우측 하단 **Add to Obsidian** 클릭
5. Obsidian이 포커스 받고 `01-Inbox/yt-2026-05-14-<영상제목>.md` 생성

생성된 노트 확인:

- [ ] frontmatter의 `source`/`author`/`published`/`tags` 자동 채워짐
- [ ] `## 📝 설명` 섹션에 영상 description (원작자가 쓴 문장 그대로)
- [ ] `## 🎬 페이지 본문` 섹션에 페이지에서 추출된 raw 텍스트 — 자막 패널 내용이 들어왔으면 그대로 두기

> **⚠️ 영상마다 `{{content}}` 품질이 다릅니다**: YouTube 페이지 구조·Glasp 확장 활성화 여부에 따라 추출되는 본문이 들쭉날쭉할 수 있습니다. 자막 부분이 비거나 깨졌으면 4-2 길(Glasp Transcript 탭 직접 복사)로 *추가 보강* 가능 — 같은 노트의 `## 🎬 페이지 본문` 아래에 *Glasp transcript*를 붙여넣어 두 raw를 함께 보존하면 4-4 omo 정리 단계의 정확도가 올라갑니다.

##### 4-2.5-3. 두 길 비교 (30초 회고)

같은 영상으로 두 노트가 생겼다면 잠깐 비교:

| 항목 | 4-2 → 4-3 (Glasp 복사 + 수동 노트) | 4-2.5 (Web Clipper) |
|---|---|---|
| **frontmatter 입력** | 수동 (직접 타이핑) | 자동 (Template properties) |
| **transcript 정확도** | 높음 (Glasp 추출 그대로) | 가변적 (페이지 구조에 의존) |
| **메타데이터 (author·published)** | 직접 채워야 함 | 자동 |
| **소요 시간** | 3~4분 | 30초 |
| **이미지·썸네일** | 없음 | 페이지 추출 시 함께 들어옴 |

본인 워크플로에 맞는 길을 채택하세요. *raw 자막 텍스트가 중요한 영상*은 4-2 길, *영상 메타데이터·간이 메모가 우선인 영상*은 4-2.5 길. 둘 다 INBOX의 *같은 layer*에 들어가므로 다음 챕터 LLM Wiki에서 동일하게 처리됩니다.

> **✨ Aha moment 보강**: 슬라이드 S3의 *"Proxy 하나로 채널 N개를 같은 규약으로 묶는다"* — Web Clipper *하나*가 이제 글(Step 3)·영상(4-2.5) *두 채널*을 같은 규약(`01-Inbox/` 자동 진입 + frontmatter 자동 채움)으로 묶고 있습니다. 캡처 도구의 수를 늘리지 않고 *Template만 1개 추가*해서 입력 채널을 확장한 패턴.

#### 4-3. Obsidian Vault 01-Inbox에 노트로 붙여넣기 (3분, 4-2 길의 마무리)

Obsidian 앱을 열고:

1. 좌측 폴더 트리에서 `01-Inbox/` 우클릭 → **New note**
2. 노트 제목을 영상 제목으로 — 예: `yt-2026-05-14-새마을운동-지도자대회`
3. Frontmatter를 직접 입력 (또는 Properties UI 사용):

   ```yaml
   ---
   source: https://www.youtube.com/watch?v=<VIDEO_ID>
   captured: 2026-05-14
   type: youtube-transcript
   language: ko
   tags:
     - clip/youtube/raw
   ---
   ```

4. 본문 영역에 클립보드 내용 붙여넣기 (`Cmd/Ctrl + V`)
5. 자동으로 다음과 같은 형식으로 들어옵니다:

   ```
   00:01  내가 이 새마을 주황에는 제가 성남 시장할 때도 가끔씩 와 봤는데...
   00:23  여러분을 보니까 참 익숙하네요...
   ...
   ```

저장 (`Cmd/Ctrl + S`, Obsidian은 자동 저장).

> **💡 왜 frontmatter를 굳이 추가하나**: `type: youtube-transcript`와 `tags: clip/youtube/raw`가 있으면 *다음 챕터 LLM Wiki*에서 Vault 내 raw 자료들을 *유형별로 검색·정제*할 때 결정적 단서가 됩니다. 지금 30초 투자가 미래의 1시간을 절약.

#### 4-4. omo에게 정리 위임 (4분)

> **두 길의 결과 노트 모두에 동일하게 적용됩니다.** 4-3에서 만든 *Glasp transcript 노트*든 4-2.5에서 만든 *Web Clipper 노트*든 — omo 입장에서는 *01-Inbox/에 있는 raw 자료*일 뿐. 아래 프롬프트의 파일 경로만 본인이 정리할 노트에 맞게 바꾸세요.

raw transcript는 *읽기 어렵습니다* — timestamp가 줄마다 박혀있고, 자동 자막이라 오타·구어체·반복이 많습니다. omo에게 PKM 친화 형식으로 정리하라고 시킵니다.

OpenCode TUI를 열고 다음 프롬프트:

```text
방금 저장한 ~/AI-Knowledge/01-Inbox/yt-2026-05-14-<영상이름>.md 의
YouTube 자막을 정리해줘.

규칙:
1. 원본 timestamp 자막은 노트 *맨 아래*로 옮기고 `## 🎬 원본 자막 (raw)` 섹션에 보관
2. 노트 상단부에 다음 3개 섹션 신규 작성:
   - `## 🎯 영상의 핵심 주장 (3~5 bullet)`
   - `## 📋 주요 인용구 (timestamp + 한 줄 + 출처)` — 인상적인 문장 3~5개를 [00:23] 본문 형식으로
   - `## 🔗 추가 조사 키워드 (5~10개)` — 영상에서 등장한 *깊이 파볼 만한* 개념·인명·도구
3. frontmatter의 `tags`에 `clip/youtube/processed` 추가
4. 원본은 *수정하지 마* — raw 섹션은 그대로 보존

자동 자막이라 오타·반복 있을 수 있으니 *의미 위주*로 읽어줘. ultrawork.
```

omo가 다음 흐름으로 처리하며 *승인 게이트*를 띄웁니다:

1. 노트 Read → **승인**
2. 분석·구조화 결과 미리보기 → 핵심 주장·인용구·키워드 보여줌
3. Edit 도구로 노트 갱신 시도 → **승인** (변경 사항 diff 확인)
4. 저장 완료

이제 노트를 다시 열면 *맨 위에 핵심 정리*, *맨 아래에 raw 자막*이 함께 있습니다. 영상을 *처음부터 끝까지 안 봐도* 핵심을 파악할 수 있고, 의심스러우면 timestamp 따라 *해당 부분만* 영상 재생.

> **✨ Aha moment**: 1시간짜리 영상을 *시청 0분 + 검증 10분*으로 정리했습니다. *영상을 본다*는 행위 자체가 검증 대역폭의 병목이었는데, transcript + omo 정리가 그 병목을 우회합니다. 슬라이드 S2의 *Last-mile Finisher*가 *동영상 컨텐츠*까지 확장됐습니다 — 90%는 자막+omo, 10%는 *당신이 어떤 timestamp를 깊이 볼지* 판단.

#### 4-5. (옵션) Summary 탭 활용 가이드

YouTube Summary의 우측 `Summary` 탭은 ChatGPT/Claude API 키를 확장 Settings에 등록하면 *확장 안에서 직접 요약*해줍니다. 단:

- *추가 API 비용 발생* (각자 키 사용량 따라)
- *omo 위임이 더 유연* — 본 강의 패턴(transcript → Vault → omo 정리)이 일관됨
- Summary 탭은 *영상만 빠르게 보고 넘기는* 시청 시나리오에 적합 — Vault에 남기지 않을 때

본 강의 시나리오(*Vault에 raw + 정리본 둘 다 보존*)에는 4-2~4-4 흐름이 더 적합합니다.

---

## ✅ 완료 체크

- [ ] **Step 1**: ChatMock 컨테이너가 healthy 상태로 기동 + `curl localhost:8000/v1/chat/completions`로 한국어 응답 확인 (omo 위임 또는 수동)
- [ ] **Step 2**: Web Clipper 확장 설치 + Vault 등록 + Interpreter에 ChatMock provider 등록 (`http://localhost:8000/v1/chat/completions`, model `gpt-5.4`) + Obsidian Files & Links에 *attachment folder = `99-Attachments/web-clipper-images`* 설정
- [ ] **Step 3**: 영문 글 1편 클립 → `01-Inbox/clip-YYYY-MM-DD-*.md` 생성, 압축 번역(최대 1500단어) + 요약 + 원문 보존 모두 포함 + *Download attachments* 명령으로 이미지가 `99-Attachments/web-clipper-images/`에 분리 저장됨
- [ ] **Step 4**: YouTube 영상 1개에 대해 두 길 중 하나 이상 수행 — (A) Glasp Transcript 복사 + 수동 노트 (4-2 → 4-3), (B) Web Clipper의 `Capture YouTube` Template로 클립 (4-2.5). 결과는 `01-Inbox/yt-YYYY-MM-DD-*.md` → omo가 핵심 주장·인용구·키워드 3섹션으로 정리 (4-4)

---

## 💡 Aha Moment

> **입력의 마찰이 0이 되면 PKM이 시작된다.**
>
> 어제까지 영어 글 1편 정리 = *직접 읽기 30분 + ChatGPT 복붙 5단계 + Vault에 다시 붙여넣기*. 오늘부터는 클립 한 번 → Vault Inbox에 *압축 번역된 노트* 자동 생성, 30~50초.
>
> 어제까지 1시간짜리 영상 정리 = *처음부터 끝까지 시청*. 오늘부터는 *두 길 중 본인 워크플로에 맞는 것을 선택* — Glasp Transcript 복사 또는 Web Clipper 클립 한 번 → omo에게 "정리해줘" → 핵심 주장·인용구·키워드 3섹션 + raw 보존. 시청 0분, 검증 10분.
>
> 슬라이드 S2의 *"수집되지 않으면 검증할 것도 없다"*가 손에 잡혔습니다. 이제 *수집* 부담이 사라졌으니 검증 대역폭을 *읽고 판단하는 데* 온전히 쓸 수 있습니다 — Last-mile Finisher의 10% 자리.
>
> **두 라인 모두 omo가 손을 잡고 있습니다.** Step 1의 docker-compose 셋업도, Step 4의 transcript 구조화도 *당신이 키보드로 친 명령은 거의 없습니다* — 검증과 승인이 작업의 본체가 됐습니다.

---

## 🔗 참고 자료

- ChatMock (ChatGPT OAuth → OpenAI compatible proxy): <https://github.com/RayBytes/ChatMock>
- ChatMock Docker 가이드: <https://github.com/RayBytes/ChatMock/blob/main/DOCKER.md>
- Obsidian Web Clipper: <https://obsidian.md/clipper>
- Obsidian Web Clipper docs: <https://help.obsidian.md/web-clipper>
- Obsidian Web Clipper *Capture* (이미지 동작 공식 설명): <https://help.obsidian.md/web-clipper/capture>
- Obsidian *Attachments* 설정 docs: <https://help.obsidian.md/attachments>
- Obsidian 1.8.0 changelog (*Download attachments* 명령 추가): <https://obsidian.md/changelog/2024-12-18-desktop-v1.8.0/>
- (옵션) auto-download 커뮤니티 플러그인: <https://github.com/chenxiccc/obsidian-auto-download-imgs-after-web-clipping>
- one-api (대안 proxy): <https://github.com/songquanpeng/one-api>
- **YouTube Summary with ChatGPT & Claude** (by Glasp Inc., 2M+ users, 4.3⭐): <https://chromewebstore.google.com/detail/youtube-summary-with-chat/nmmicjeknamkfloonkhhcjmomieiodli>
- Glasp 공식 사이트: <https://glasp.co/youtube-summary>

---

## ▶ 다음 단계

raw 자료가 들어오는 라인을 갖췄다면, 이제 그 자료를 **체계적으로 정제·운영**할 차례입니다.

**다음 챕터**: [07. LLM Wiki]({{< ref "07-llm-wiki" >}}) — Karpathy LLM Wiki 패턴 기반 지식 베이스 운영
