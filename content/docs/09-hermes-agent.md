---
title: "09. Hermes Agent 구축"
weight: 9
date: 2026-05-22
draft: false
---

> **⏰ 시간**: 약 70분 (이론 5분 + 실습 65분)
> **🎯 목표**: macOS·Windows WSL2 양쪽에서 Hermes Agent를 Docker 컨테이너로 띄우고, Slack Socket Mode로 워크스페이스와 연결해 *상주 동료*를 갖춥니다.

---

## 📄 이론 슬라이드

> 이 챕터의 이론은 아래 슬라이드로 다룹니다.
>
> *(슬라이드 모듈 `hermes-agent/v1.0` — 작성 예정)*

---

## 🧭 한 줄 요약

> **OpenCode가 "내 데스크탑의 검증 워크벤치"였다면, Hermes Agent는 "24시간 떠 있는 PKM 동료"입니다. Slack에서 말 걸면 답합니다.**

---

## 📚 학습 목표

이 챕터를 마치면 다음을 할 수 있습니다:

- [ ] OpenCode와 Hermes Agent의 *역할 분담* (검증 단계 vs 운영 단계) 을 설명할 수 있다
- [ ] macOS·Windows WSL2 두 환경 모두에서 Docker Desktop을 띄우고 검증할 수 있다
- [ ] Hermes Agent를 `docker compose`로 띄우고 CLI 채팅으로 첫 응답을 받을 수 있다
- [ ] Slack Socket Mode 앱을 만들고 Bot Token·App-Level Token을 발급해 wizard에 입력할 수 있다
- [ ] 자기 Slack 워크스페이스에서 봇과 DM·멘션으로 대화할 수 있다
- [ ] `hermes doctor`, `gateway status`, `docker compose logs` 진단 명령으로 장애를 격리할 수 있다

---

## 🛠️ 사전 요구사항

| 항목 | 요구 사항 |
|---|---|
| **OS** | macOS 13↑ / Windows 10 22H2↑ · Windows 11 + WSL2 Ubuntu |
| **OpenCode** | Chapter 01에서 설치 완료 (역할 비교용) |
| **`$VAULT_ROOT`** | Chapter 05 Step 6 도입부에서 설정 완료 |
| **OpenAI ChatGPT Pro 구독** | Hermes provider OAuth용 |
| **Slack 워크스페이스** | 앱 설치 권한 보유 (없으면 무료 워크스페이스 신규 생성) |
| **메모리** | 4 GB 이상 가용 |
| **디스크** | 5 GB 이상 여유 |

> [!info] **📍 5장과 동일한 약속**
>
> 이 챕터의 모든 터미널 명령은 **macOS / Windows WSL2 Ubuntu** 두 환경에서 *동일하게* 작동합니다. 프롬프트는 `$`로 표기하며, PowerShell은 사용하지 않습니다.
>
> Windows 학생: **Windows Terminal → Ubuntu 탭**에서 작업합니다.

---

## §0. 왜 또 다른 Agent? — OpenCode와의 역할 분담 (이론, 5분)

지금까지의 챕터에서 우리는 **OpenCode**라는 도구를 깊이 다뤘습니다. MCP, Command, Skill, AGENTS.md — 모두 OpenCode 위에서 동작했습니다.

그런데 이번 챕터에서 우리는 *또 다른 AI Agent*인 **Hermes Agent**를 추가합니다. 왜일까요?

### 두 도구는 *다른 시간대*를 책임집니다

| 항목 | OpenCode | Hermes Agent |
|---|---|---|
| **언제 동작?** | 내가 *부를 때* | 내가 *없어도* (24시간 상주) |
| **어디서?** | 내 데스크탑 터미널 | 내 PC·VPS의 백그라운드 |
| **어떻게 말 걸지?** | TUI (Terminal UI) | Slack·Telegram·이메일 |
| **주된 역할** | *검증 동반자* — 같이 코드 읽고, 분석하고, 정제 | *상주 동료* — 알림 받고, 스케줄 돌리고, 메모리 누적 |
| **검증 대역폭의 자리** | 깊이·정확성 | 지속성·접근성 |

### 충돌하지 않습니다 — *시간대가 다르니까*

> [!info] **두 번째 도구가 아니라, *다른 시간대의 도구***
>
> OpenCode와 Hermes가 *같은 일*을 두 번 하는 도구라면 골라야 합니다. 그런데 둘은 *다른 시간대를 책임집니다*:
>
> - 책상에 앉아 vault를 정제할 땐 → **OpenCode**
> - 출퇴근길에 Slack으로 한 줄 던질 땐 → **Hermes**
> - 매일 새벽 4시에 어제 Inbox 요약을 받고 싶을 땐 → **Hermes의 cron**
> - 받은 raw 자료를 LLM Wiki로 정제할 땐 → **OpenCode**
>
> *검증 대역폭은 시간대마다 다른 도구가 맡을 수 있다* — 이게 이번 챕터의 메타 메시지입니다.

### 그래서 이번 챕터에서 무엇을 완성하나

이 챕터를 마치면 다음 세 가지가 *학생 환경에 완성*됩니다:

1. **Hermes Agent가 Docker 컨테이너로 띄워진 상태** — 호스트 보호 + 재현성 + cleanup 한 방
2. **Slack Socket Mode로 자기 워크스페이스와 양방향 연결** — inbound 포트 없이, 모바일·웹 어디서나 봇과 대화
3. **첫 PKM 시나리오 동작 확인** — Slack에서 "Inbox에 저장해" → vault에 raw 노트 적재

이 세 가지가 모두 작동하면 — **책상에서만 가능했던 PKM 작업이 출퇴근길·점심시간에도 가능**해집니다.

다음 §1에서 *전체 구조*를 한 장 그림으로 봅니다.

---

## §1. 아키텍처 한 장 그림 (3분)

```text
[내 PC — macOS / Windows]
│
├─ 터미널 환경  ← 5장에서 약속한 그대로
│   ├─ (Mac) iTerm2 / Warp / 기본 터미널
│   └─ (Win) Windows Terminal → Ubuntu 탭 (WSL2)
│       └─ opencode  ← Chapter 01의 그것 (변경 없음)
│
└─ Docker Desktop  ← 이번 챕터에서 신규
    └─ hermes-gateway 컨테이너
        ├─ /data ← bind mount ←→ 호스트의 ~/.hermes/
        │   ├─ .env (provider OAuth + Slack 토큰)
        │   ├─ slack-manifest.json (wizard 자동 생성)
        │   ├─ config.yaml
        │   ├─ skills/ · memories/ · sessions/
        │   └─ logs/
        │
        ├─ /vault ← bind mount ←→ $VAULT_ROOT (§7에서 추가)
        │
        ├─ outbound WebSocket → Slack (Socket Mode)
        └─ outbound HTTPS    → OpenAI Codex (OAuth)
```

### 왜 이렇게 짰는가 — 세 가지 결정

#### 1. 왜 Docker 컨테이너?

- **호스트 보호**: Hermes는 *AI에게 shell 권한*을 주는 도구입니다. 컨테이너 격리가 1차 방어선
- **재현성**: macOS·WSL2 *어디서 실행해도 컨테이너 안은 동일 Linux*
- **Cleanup 한 방**: 실습 후 `docker compose down -v` 한 줄로 깔끔
- **자동 재기동**: `restart: unless-stopped` → 재부팅 후 자동 살아남

#### 2. 왜 Socket Mode?

```text
[Slack 서버] ←──── WebSocket (outbound, TLS 443) ────  [Hermes 컨테이너]
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
              Hermes가 *밖으로* 연결을 엽니다
```

- 내 PC에 **inbound 포트 노출 불필요** (방화벽·NAT 설정 X)
- 사내 PC·집 공유기 뒤·VPS 어디서나 같은 패턴
- Hermes의 outbound WebSocket이 *살아있는 한* Slack 메시지가 흘러옵니다

#### 3. 왜 bind mount 2개?

| 마운트 | 역할 | 잃으면? |
|---|---|---|
| `~/.hermes:/data` | 영속 데이터 (skills·memory·sessions·logs·.env) | 컨테이너 재생성 시 *모든 학습 데이터 손실* |
| `$VAULT_ROOT:/vault` | Hermes가 vault 직접 읽기·쓰기 | Slack에서 "Inbox에 저장해" 안 됨 |

→ **§4까지는 `/data` 마운트만**, §7에서 vault 마운트를 추가합니다 (시나리오에 따라).

### 다음 단계

§2에서 Docker Desktop을 깔고, §3에서 Slack 워크스페이스를 미리 준비한 뒤, §4에서 Hermes를 띄웁니다.

---

## §2. Docker Desktop 설치 (10분, 실습)

Hermes를 *컨테이너*로 띄우려면 먼저 Docker 데몬이 필요합니다. **Docker Desktop**이 가장 단순한 경로 — Windows의 WSL Integration도 자동, macOS의 가상 머신 관리도 자동.

> [!info] **🪟 Windows + 🍎 macOS 두 갈래 — 그러나 §2-수렴부터 동일**
>
> §2-A·§2-B에서 설치 GUI 단계만 다릅니다. 그 후 *모든 `docker` 명령은 §2-수렴부터 동일*하게 작동합니다.

### §2-A. Windows 학생 — Docker Desktop + WSL Integration

#### 1. Docker Desktop 설치 파일 다운로드

브라우저에서 <https://www.docker.com/products/docker-desktop> → **Download for Windows (AMD64)**.

> 💡 **ARM64 Windows 사용자** (Surface Pro X 등): 같은 페이지에서 ARM64 버전 선택.

#### 2. 설치 실행

`Docker Desktop Installer.exe` 더블클릭. 설치 옵션에서:

- ✅ **Use WSL 2 instead of Hyper-V (recommended)** — 기본값, 그대로
- ✅ **Add shortcut to desktop** — 취향대로
- 클릭: **Ok** → 설치 진행 → **Close and restart**

재시작 후 자동 로그인 시 *Docker Desktop이 트레이에 자동 실행*됩니다.

#### 3. WSL Integration 켜기 (핵심)

Docker Desktop 창 열고:

1. 우측 상단 톱니바퀴(⚙️) → **Settings**
2. 왼쪽 메뉴 **Resources → WSL Integration**
3. **Enable integration with my default WSL distro** ✅
4. 아래 *Enable integration with additional distros* 목록에서 **Ubuntu-24.04** (또는 본인이 쓰는 배포판) ✅
5. **Apply & restart**

→ 이 순간부터 **WSL Ubuntu 안에서 `docker` 명령이 자동 동작**합니다.

#### 4. WSL Ubuntu 탭으로 이동 + 검증

Windows Terminal → Ubuntu 탭에서:

```bash
$ docker --version
Docker version 27.x.x, build ...
```

✅ 버전 출력되면 §2-수렴으로.

> [!warning] **Docker Desktop 라이선스 — 회사 노트북 사용자**
>
> Docker Desktop은 *개인·교육용은 무료*입니다. 그러나 **250명↑ 직원** 또는 **연 매출 1천만 달러↑** 회사의 *업무용* 사용은 유료 (Docker Pro/Team/Business).
>
> 회사 노트북으로 듣는 학생은 정책 확인 필수. 무료 대안:
>
> - **Rancher Desktop** (<https://rancherdesktop.io>) — Apache 2.0, Docker CLI 호환. 본 챕터 명령 그대로 동작
> - **Podman Desktop** (<https://podman-desktop.io>) — Apache 2.0. `docker` 대신 `podman` 명령

### §2-B. macOS 학생 — Docker Desktop

#### 1. Docker Desktop 다운로드

<https://www.docker.com/products/docker-desktop>에서:

- **Apple Silicon (M1/M2/M3/M4)** → `Docker.dmg` (Apple Silicon)
- **Intel Mac** → `Docker.dmg` (Intel chip)

#### 2. 설치 실행

`.dmg` 더블클릭 → **Docker** 아이콘을 **Applications** 폴더로 드래그.

처음 실행 시:

- 보안 경고: "Docker.app cannot be opened because the developer cannot be verified" → 시스템 설정 → 개인정보·보안 → **Open Anyway**
- 권한 요청 (privileged helper 설치) → 사용자 암호 입력

#### 3. 첫 실행 + 검증

Docker Desktop이 트레이(메뉴바)에 뜨면 *고래 아이콘*이 *정지 상태(흰색)* → *동작 상태(파란색)* 으로 바뀜 (1~2분).

터미널(iTerm2 / Warp / 기본 터미널)에서:

```bash
$ docker --version
Docker version 27.x.x, build ...
```

✅ 버전 출력되면 §2-수렴으로.

> [!note] **Colima / OrbStack을 선호한다면?**
>
> macOS에서 Docker Desktop의 *가벼운 대안*들:
>
> - **Colima** (<https://github.com/abiosoft/colima>) — 무료, CLI 기반
> - **OrbStack** (<https://orbstack.dev>) — 개인 무료·상업 유료, 가장 빠름
> - **Rancher Desktop** — 무료, GUI
>
> 셋 다 *Docker CLI 호환* → 본 챕터의 모든 `docker` 명령이 그대로 작동합니다. Docker Desktop과 동시 사용은 권장하지 않음 (소켓 충돌).

### §2-수렴. 공통 검증 (양쪽 학생 모두)

#### 1. Docker Compose 플러그인 확인

```bash
$ docker compose version
Docker Compose version v2.x.x
```

> 💡 `docker-compose` (하이픈) v1은 *deprecated*. 본 챕터는 *공백 있는* `docker compose` v2를 사용합니다. Docker Desktop은 v2를 기본 포함.

#### 2. Hello World 컨테이너 실행

```bash
$ docker run --rm hello-world
```

성공 메시지:

```text
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

#### 3. (Windows 학생만 추가 확인) WSL에서 Docker 컨텍스트 확인

```bash
$ docker context ls
NAME            DESCRIPTION                               DOCKER ENDPOINT
default *       Current DOCKER_HOST based configuration   npipe:////./pipe/docker_engine
desktop-linux   Docker Desktop                            unix:///var/run/docker.sock
```

`default` 또는 `desktop-linux`에 `*` (현재 활성) 표시 — 둘 중 어느 쪽이든 *WSL Integration이 켜져 있으면* 동작.

#### ✅ 검증 체크리스트

- [ ] `docker --version` 출력 (27.x 이상)
- [ ] `docker compose version` 출력 (v2.x 이상)
- [ ] `docker run --rm hello-world` 성공 메시지
- [ ] (Windows) Windows Terminal Ubuntu 탭에서 위 3가지 모두 동작
- [ ] (macOS) 메뉴바에 Docker 고래 아이콘 *파란색* 상태

> [!warning] **위 3개 명령이 모두 통과해야 §3으로 넘어갑니다.**
>
> 막힐 때:
>
> - "Cannot connect to the Docker daemon" → Docker Desktop이 *기동 중*. 1~2분 대기 후 재시도
> - "permission denied" (WSL) → Docker Desktop Settings → WSL Integration 재확인, WSL 셸 새로 열기
> - 가상화 비활성화 에러 (Windows) → BIOS에서 Intel VT-x / AMD-V 활성화

---

## §3. Slack 계정·워크스페이스 미리 준비 (5분)

§4에서 Hermes setup wizard가 *Bot Token·App-Level Token*을 요구하는 시점에 *wizard를 멈추고* Slack 앱을 만들러 갈 것입니다. 그때 *워크스페이스·홈 채널·자기 멤버 ID*가 *이미 준비되어 있어야* 흐름이 끊기지 않습니다.

### 3-A. Slack 워크스페이스 확보 (3분)

#### 이미 워크스페이스가 있다면

자기가 **앱 설치 권한**이 있는 워크스페이스인지 확인:

1. Slack 데스크탑 앱 또는 웹에서 워크스페이스 열기
2. 좌측 상단 워크스페이스 이름 → **Settings & administration → Workspace settings**
3. **Permissions** → **Apps** 섹션 → "Anyone can install apps" 또는 "Admin approval required" 확인
4. *Admin approval required*이면 — 관리자에게 요청하거나, 3-B로 새 워크스페이스 생성

#### 새 워크스페이스가 필요하다면 (학습 전용 권장)

본 챕터의 실습 안전성을 위해 **개인 학습용 워크스페이스 신규 생성**을 권장합니다.

1. <https://slack.com/get-started#/createnew> 접속
2. 이메일 입력 → 인증 코드 → 워크스페이스 이름 (예: `dangtong-pkm`)
3. 첫 채널 이름 입력 → Skip team members → 워크스페이스 생성 완료

> 💡 **개인 학습용 워크스페이스의 이점**:
>
> - 모든 앱 설치 권한이 자기에게 있음
> - 실수해도 회사 워크플로우에 영향 없음
> - 무료 플랜으로 무제한 사용 (메시지 90일 보관 한도만 있음)

### 3-B. 자기 멤버 ID 복사 (1분)

Hermes의 `SLACK_ALLOWED_USERS` 설정에 들어갈 *자기 고유 ID*입니다.

1. Slack 좌측 상단 자기 *프로필 사진* 클릭
2. **Profile** 클릭 → 우측 패널에 자기 프로필 표시
3. 프로필 패널 우측 상단의 **⋯ (더보기) 버튼** → **Copy member ID**
4. 클립보드에 `U01ABCD2EFG` 같은 *알파벳·숫자 문자열*이 복사됨

> 💡 **메모장에 임시 저장**: §4-K에서 입력할 때까지 안전한 곳에 보관. 보통 `U`로 시작하는 11자 안팎.

### 3-C. 홈 채널 결정 + 생성 (1분)

Hermes가 *기본 채널*로 사용할 채널 — 알림·스케줄 결과가 이리로 옵니다.

#### 채널 생성

1. 좌측 사이드바 **Channels** 옆 **+** → **Create a new channel**
2. 채널 이름: `hermes-office` (또는 원하는 이름)
3. **Public** 권장 (Private이면 봇을 *명시적으로* 초대해야 함)
4. **Create** → **Skip for now** (멤버 추가 건너뛰기)

#### 채널 ID는 §4-K에서 복사

채널 ID(`C04XYZ123AB` 형태)는 *§4 wizard가 채널 ID를 요구하는 시점*에 복사해도 늦지 않습니다. 미리 해두려면:

1. 만든 채널 이름 클릭 → 상단 채널명 → 우측에 채널 정보 패널
2. 하단 *About* 섹션 끝의 **Channel ID** 복사 (또는 채널명 우클릭 → **View channel details** → 맨 아래)

### ✅ 검증

이 시점에 메모장 또는 안전한 곳에 다음 4가지가 준비:

- [ ] Slack 워크스페이스 (접속 가능)
- [ ] 자기 멤버 ID (`U01...`)
- [ ] 홈 채널 이름 (`#hermes-office` 등)
- [ ] 홈 채널 ID (`C04...`) — 또는 §4-K에서 복사

> 📌 **봇 자체는 §4에서 만듭니다.** 지금은 *워크스페이스만 준비*. Hermes setup wizard가 *manifest JSON*을 자동 생성하므로, 우리가 봇 scope·event subscription을 *수동으로* 입력할 필요가 없습니다. 흐름이 훨씬 단순합니다.

---

## §4. Hermes 컨테이너 + setup wizard (25분, 실습)

이 섹션이 이번 챕터의 핵심입니다. *컨테이너를 띄우고 → wizard로 설정 → Slack 연동까지* 한 흐름으로 진행합니다.

### 4-A. 작업 디렉토리 + docker-compose.yaml 작성 (3분)

#### 1. 데이터 디렉토리 생성

```bash
$ mkdir -p ~/.hermes
$ ls -la ~/.hermes
total 0
drwxr-xr-x  ...  .
drwxr-xr-x  ...  ..
```

빈 디렉토리. wizard가 이 안에 `.env`·`config.yaml`·`slack-manifest.json` 등을 채울 것입니다.

#### 2. compose 작업 디렉토리

```bash
$ mkdir -p ~/hermes-stack
$ cd ~/hermes-stack
```

> 💡 **왜 `~/hermes-stack`?** `~/.hermes`(데이터)와 *분리*하면 `docker-compose.yaml`을 git으로 관리할 때 *.env·logs는 자동 제외*되어 안전합니다.

#### 3. `docker-compose.yaml` 작성

```bash
$ cat > ~/hermes-stack/docker-compose.yaml << 'EOF'
services:
  gateway:
    image: ghcr.io/nousresearch/hermes-agent:latest
    container_name: hermes-gateway
    restart: unless-stopped
    command: gateway
    env_file: /Users/dangtongbyun/.hermes/.env   # ⚠️ 자기 경로로 수정
    volumes:
      - ~/.hermes:/data
    environment:
      - HERMES_HOME=/data
    stdin_open: true
    tty: true
EOF
```

> [!warning] **`env_file` 경로 — 자기 환경에 맞게 수정**
>
> Docker Compose는 `env_file`에서 `~`를 *해석하지 않습니다*. 절대경로로 적어야 합니다:
>
> - macOS: `env_file: /Users/사용자이름/.hermes/.env`
> - WSL2: `env_file: /home/사용자이름/.hermes/.env`
>
> 자기 홈 경로 확인: `$ echo $HOME`

> 💡 **`.env`가 아직 없어서 첫 실행 시 경고 나옴** — 정상. wizard가 채울 예정. 미리 빈 파일 만들어 두려면:
>
> ```bash
> $ touch ~/.hermes/.env
> ```

#### 4. compose 문법 검증

```bash
$ cd ~/hermes-stack
$ docker compose config
```

`yaml validation OK` 류 출력이면 통과. 에러 나면 들여쓰기·경로 재확인.

### 4-B. 📌 컨테이너 실행 패턴 — `run --rm -it` vs `up -d`

> [!info] **이 박스를 *반드시* 먼저 이해하고 넘어가세요**
>
> 본 챕터는 `docker compose`의 *두 가지 실행 모드*를 번갈아 씁니다:
>
> | 패턴 | 언제? | 동작 |
> |---|---|---|
> | `docker compose run --rm -it gateway <cmd>` | **wizard·일회성** 명령 | 전용 컨테이너 생성 → 명령 실행 → 종료 + 자동 삭제 (`--rm`) |
> | `docker compose up -d gateway` | **백그라운드 상시 운영** | 백그라운드 데몬 모드, 재부팅 시 자동 살아남 (`restart: unless-stopped`) |
> | `docker compose exec -it gateway <cmd>` | **이미 떠 있는** 컨테이너에 명령 보내기 | 백그라운드 컨테이너 안에서 명령 실행 (TUI·진단) |
>
> §4-C ~ §4-N 동안은 **`run --rm -it`** 모드를 씁니다 — wizard가 interactive하니까. §4-N 끝에서 `up -d`로 전환.

### 4-C. Wizard 진입 + Provider OAuth (5분)

#### 1. Wizard 실행

```bash
$ cd ~/hermes-stack
$ docker compose run --rm -it gateway hermes setup
```

처음 이미지 다운로드 1~3분 대기 (`Pulling fs layer...`). 끝나면:

```text
╔══════════════════════════════════════════════════╗
║         Welcome to Hermes Agent Setup            ║
╚══════════════════════════════════════════════════╝

How would you like to set up Hermes?
  → (●) Quick setup — provider, model & messaging (recommended)
    (○) Full setup — configure everything
```

→ **Quick setup** 선택 후 Enter.

#### 2. Provider 선택 — OpenAI Codex

```text
Select your LLM provider:
  → (●) OpenAI Codex (ChatGPT Pro/Plus subscription)
    (○) OpenAI (sk-... API key)
    (○) Anthropic Claude
    (○) OpenRouter
    (○) Nous Portal
    (○) Custom (OpenAI-compatible endpoint)
```

→ **OpenAI Codex** 선택 후 Enter.

> 💡 **왜 OpenAI Codex?** ChatGPT Pro 구독에 *포함된 Codex 모델*을 사용 — 별도 API key 발급·과금 없이 OAuth로 인증. 사전 요구사항의 *ChatGPT Pro 구독*이 이 항목 때문.

#### 3. OAuth 디바이스 코드 흐름

```text
[Not logged into OpenAI Codex. Starting login...

Signing in to OpenAI Codex...
(Hermes creates its own session — won't affect Codex CLI or VS Code)

To continue, follow these steps:
  1. Open this URL in your browser:
     https://auth.openai.com/codex/device
  2. Enter this code:
     XXXX-YYYY

Waiting for sign-in... (press Ctrl+C to cancel)
```

수강생 액션:

1. **자기 PC 브라우저**에서 <https://auth.openai.com/codex/device> 열기
2. 표시된 *코드 8자리* 입력
3. ChatGPT Pro 계정 로그인 (이미 로그인 상태면 자동 통과)
4. **Authorize** 클릭

→ wizard 터미널이 **자동 감지** → 다음 단계로 이동:

```text
Login successful!
  Auth state: /data/auth.json
  Config updated: /data/config.yaml (model.provider=openai-codex)
```

> [!warning] **디바이스 코드는 일회용 + 짧은 만료**
>
> 보통 5~10분 안에 입력해야 합니다. 시간 지나면 wizard에서 Ctrl+C로 중단 후 `hermes setup` 재실행.

### 4-D. 모델 선택 — `gpt-5.5` (1분)

```text
Select default model:
  →  gpt-5.5
     gpt-5.4
     gpt-5.4-mini
     gpt-5.3-codex
     gpt-5.3-codex-spark
     gpt-5.2
     Enter custom model name
     Skip (keep current)
```

→ **gpt-5.5** 선택 후 Enter.

> 💡 **왜 `gpt-5.5`?** Hermes는 *상주 동료* — 한국어 자연어 대화·요약·정제 비중이 높습니다. `gpt-5.5`가 일반 대화 품질에서 가장 균형 잡혀 있습니다.
>
> Chapter 05에서 OpenCode용으로는 `gpt-5.3-codex`(도구 호출·코드 강세)를 권장했지만, *Hermes의 사용 맥락이 다르기 때문에* 모델 선택도 다릅니다. 언제든 `hermes model` 명령으로 전환 가능.

### 4-E. 세션 운영 옵션 (3분)

> 💡 **Quick setup에선 일부 항목만 묻습니다.** 묻지 않는 항목은 *기본값* 적용 — 나중에 `hermes config set`으로 조정 가능.

다음 4가지가 차례로 등장:

#### 1. Max Iterations

```text
Max iterations per request:
  Default: 60
  Recommended for complex tasks: 90
  For exploratory workflows: 150+
> [60]:
```

→ **90** 입력 후 Enter.

복잡한 PKM 정제 요청(*"vault audit 후 결과로 회의록 정리 액션 만들어"*)에서 60으로는 부족할 수 있습니다.

#### 2. Tool Progress Mode

```text
Tool progress display:
  (1) Silent — no tool output
  → (2) Compact — show every call with a short preview  ← 권장
    (3) Verbose — show full tool input/output
```

→ **2 (Compact)** Enter.

Verbose는 정보량 폭주, Silent는 *AI가 뭐 하는지* 안 보임. Compact가 학습·운영 양쪽 균형.

#### 3. Compact Threshold

```text
Compact context at what fill ratio?
  Default: 0.5 (compact when half full)
  Use more context: 0.75
> [0.5]:
```

→ **0.75** 입력 후 Enter.

대화 컨텍스트의 75%가 찰 때까지 그대로 활용 → 더 긴 대화 유지.

#### 4. Reset Policy

```text
Session reset triggers:
  (1) Manual only
  (2) Inactivity
  (3) Daily
  → (4) Inactivity + Daily  ← 권장
```

→ **4** Enter → 후속 질문:

```text
Inactivity timeout (minutes): [1440]   → Enter (24시간)
Daily reset time (HH:MM):    [04:00]   → Enter (새벽 4시)
```

### 4-F. Terminal Backend — Local (1분)

```text
Where should Hermes execute shell commands?
  → (●) Local — run directly on this machine
    (○) Docker — isolated container with configurable resources
    (○) Modal · SSH · Daytona · Vercel Sandbox · Singularity
    (○) Keep current
```

→ **Local** 선택 후 Enter.

> [!info] **"우리는 이미 컨테이너 안인데 또 Docker?"**
>
> wizard의 `Docker` 옵션은 *호스트에서 또 다른 컨테이너*를 띄우는 옵션 — Hermes가 *기본 환경 밖*에서 살 때 격리하려는 용도입니다.
>
> 우리는 *이미* `hermes-gateway` 컨테이너 안에서 wizard를 돌리고 있습니다. 여기서 `Local`은 *Hermes가 자기 컨테이너 내부에서 직접 명령 실행*을 뜻하며, **호스트에는 영향 없음**. 컨테이너 격리는 그대로.

### 4-G. Messaging — Slack 선택 → wizard 일시 멈춤 (1분)

```text
Connect a messaging platform?
  → (●) Set up messaging now (recommended)
    (○) Skip — set up later with 'hermes setup gateway'
```

→ **Set up messaging now** Enter → 플랫폼 선택:

```text
Which messenger?
  (1) Telegram
  → (2) Slack  ← 권장
    (3) Discord
    (4) WhatsApp
    (5) Signal
    (6) Email
    (7) Mattermost
    ...
```

→ **2 (Slack)** Enter.

다음 화면:

```text
Slack setup
─────────────
A manifest file has been generated at:
  /data/slack-manifest.json

Next steps:
  1. Go to https://api.slack.com/apps
  2. Click "Create New App" → "From an app manifest"
  3. Select your workspace → paste the manifest

When done, paste your Bot Token (xoxb-...) here:
> _
```

> 🔀 **여기서 wizard가 멈춥니다 — 입력 대기 상태**
>
> 터미널을 *그대로 두고* 다음 §4-H로 넘어가 Slack App을 만듭니다. wizard는 죽지 않고 *Bot Token 입력만 기다립니다*.

### 4-H. Slack App 생성 — Hermes 자동 생성 manifest 활용 (5분) ⭐

#### 1. manifest JSON 파일 위치 확인

**별도 터미널 창** (또는 같은 창의 새 탭) 열고:

```bash
$ cat ~/.hermes/slack-manifest.json
```

JSON 전체가 출력됩니다. 이런 모양:

```json
{
  "display_information": {
    "name": "Hermes Agent"
  },
  "features": {
    "bot_user": {
      "display_name": "hermes",
      "always_online": true
    }
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "im:history",
        "im:read",
        ...
      ]
    }
  },
  "settings": {
    "event_subscriptions": { ... },
    "interactivity": { "is_enabled": true },
    "socket_mode_enabled": true
  }
}
```

→ 전체 출력을 **전체 선택 + 복사**.

> 💡 **왜 Hermes가 manifest를 자동 생성?** Slack 앱 등록에 필요한 *scope·event subscription·Socket Mode 활성화* 등 ~30개 항목이 정확히 채워진 JSON입니다. 우리가 수동으로 입력하면 *오타 한 번에 봇이 메시지를 못 받음*. wizard가 책임집니다.

#### 2. Slack App 생성

자기 PC 브라우저에서 <https://api.slack.com/apps>:

1. 우측 상단 **Create New App** 클릭
2. **From an app manifest** 선택
3. Workspace 선택: §3에서 준비한 워크스페이스 → **Next**
4. **YAML / JSON 선택 탭** → **JSON** 클릭
5. 미리 표시된 기본 manifest *전체 삭제* → 위에서 복사한 JSON *전체 붙여넣기*
6. **Next** → 미리보기 화면에서 확인 → **Create**

→ Slack이 *앱 페이지*로 이동합니다.

#### 3. (선택) 앱 이름·아이콘 커스터마이즈

좌측 메뉴 **Basic Information** → **Display Information**:

- **App name** — 기본 "Hermes Agent" 또는 자기 이름 (예: `pkm-helper`)
- **Short description** — 한 줄 설명
- **Background color** — 자기 취향
- **App icon** — 봇 아이콘 (선택)

> 💡 봇에 *사람 이름·페르소나*를 주면 *동료 느낌*이 살아납니다. 학습 효과에도 좋음.

**Save Changes** 클릭.

### 4-I. Bot Token (xoxb) 발급 → wizard로 복귀 (3분)

#### 1. 워크스페이스에 앱 설치

좌측 메뉴 **OAuth & Permissions** → 상단 **OAuth Tokens**:

1. **Install to <워크스페이스명>** 버튼 클릭
2. 권한 검토 화면 → **Allow**
3. 페이지가 새로고침되면 **Bot User OAuth Token** 표시:

   ```text
   Bot User OAuth Token
   xoxb-<your-bot-token>  [Copy]
   ```

4. **Copy** 버튼 클릭 → 클립보드에 `xoxb-...` 토큰 복사

#### 2. wizard 터미널로 돌아가 붙여넣기

§4-G에서 멈춰뒀던 터미널 (`Paste your Bot Token (xoxb-...):` 상태):

1. **마우스 우클릭 → 붙여넣기** (또는 `Cmd/Ctrl+V`)
2. **Enter**

> [!warning] **붙여넣을 때 *커서가 안 움직여도 정상*입니다**
>
> 보안상 *토큰 입력을 마스킹*하는 터미널 동작입니다. 입력은 *되어 있습니다* — 그냥 Enter 누르세요.

wizard가 다음 단계로:

```text
✓ Bot Token saved
Now I need an App-Level Token (xapp-...) for Socket Mode.

Steps:
  1. In your Slack app page, go to "Basic Information"
  2. Scroll to "App-Level Tokens" → "Generate Token and Scopes"
  3. Name: "hermes-socket", Scope: "connections:write"
  4. Click "Generate" → copy the xapp-... token

Paste your App-Level Token (xapp-...) here:
> _
```

→ wizard가 또 멈춤. §4-J로.

### 4-J. App-Level Token (xapp) 발급 → wizard 입력 (2분)

#### 1. Slack 앱 페이지에서 App-Level Token 생성

좌측 메뉴 **Basic Information** → 페이지 *맨 아래* **App-Level Tokens** 섹션:

1. **Generate Token and Scopes** 클릭
2. **Token Name**: `hermes-socket` (또는 임의)
3. **Add Scope** 클릭 → `connections:write` 선택 → **Add**
4. **Generate** 클릭
5. 표시된 `xapp-...` 토큰 → **Copy** 버튼 클릭

#### 2. wizard 터미널에 붙여넣기

§4-I와 동일한 절차 — 우클릭 붙여넣기 → Enter.

```text
✓ App-Level Token saved
✓ Socket Mode handshake test... OK
```

→ wizard가 다음 단계로 진행.

> [!info] **두 토큰의 역할 차이 — 헷갈리지 마세요**
>
> | 토큰 | 시작 | 역할 | 어디서 발급? |
> |---|---|---|---|
> | **Bot Token** | `xoxb-...` | 봇이 메시지·이모지·파일을 *보내고 받기* | OAuth & Permissions → Install to Workspace |
> | **App-Level Token** | `xapp-...` | Hermes ↔ Slack 사이 *Socket Mode 연결* 자체 | Basic Information → App-Level Tokens |
>
> `xoxb`는 *말하기*, `xapp`은 *연결하기*. 둘 다 있어야 동작합니다.

### 4-K. Allowed Users + Home Channel ID (2분)

#### 1. Allowed User IDs

```text
Slack Allowed User IDs (comma-separated):
  Members who can talk to Hermes. Bot ignores everyone else.
> _
```

§3-B에서 복사한 *자기 멤버 ID* 붙여넣기:

```text
> U01ABCD2EFG
```

여러 명 허용하려면 쉼표로:

```text
> U01ABCD2EFG,U02HIJK3LMN
```

Enter.

> [!warning] **이 목록에 없는 사람은 봇이 *무시*합니다**
>
> 봇이 들어간 채널의 *다른 멤버가 멘션해도 응답 안 함*. 보안상 중요한 안전망 — 정확히 입력하세요.

#### 2. Home Channel ID

```text
Slack Home Channel ID (optional, press Enter to skip):
  Used for proactive messages, scheduled summaries, etc.
> _
```

§3-C에서 만든 채널의 ID 입력 (아직 ID 안 복사했으면 지금 복사):

채널 ID 복사 방법 (재확인):

1. Slack에서 채널 이름 클릭 (사이드바)
2. 채널 상단 채널명 클릭 → 채널 정보 패널 열림
3. 패널 맨 아래 **Channel ID** 표시 → 복사

```text
> C04XYZ123AB
```

Enter.

### 4-L. systemd 서비스 질문 → No (1분)

```text
Install gateway as a system service?
  This makes Hermes auto-start at boot.
  → (●) Yes — user service (recommended for personal use)
    (○) Yes — system service (requires sudo)
    (○) No — I'll start it manually
```

→ **No** 선택 후 Enter.

> [!info] **왜 No?** 우리는 *Docker 컨테이너의 `restart: unless-stopped`*가 같은 역할을 합니다 — 시스템 재부팅 후 자동 시작, 컨테이너 크래시 시 자동 재기동.
>
> systemd 서비스를 *컨테이너 내부에서* 등록하면 *컨테이너 종료 시 의미 없음* + 호스트 systemd와 충돌 위험. **컨테이너 환경에선 항상 No**.

### 4-M. 부가 Provider — 빠르게 통과 (2분)

여러 항목이 차례로 나옵니다. 기본값 또는 권장값으로 빠르게:

#### CLI Tools

```text
Allowed CLI tools: [defaults]
```

→ **Enter** (기본값 유지)

#### Browser

```text
Browser provider:
  → (●) Local browser (headless Chromium via Playwright)
    (○) Firecrawl
    (○) Browser Use
    (○) Browserbase
```

→ **Local browser** Enter — 추가 비용 없음, 자기 PC 자원 사용.

#### Image Model

```text
Image model provider:
  → (●) OpenAI Codex (gpt-image-2)  ← Pro 구독에 포함
    (○) fal.ai
    (○) Skip
```

→ **OpenAI Codex** Enter → 품질:

```text
Image quality: low / [medium] / high
```

→ **medium** Enter.

> 💡 ChatGPT Pro 구독에 *gpt-image-2가 포함* — 별도 과금 없습니다.

#### TTS (음성)

```text
TTS provider:
  → (●) Microsoft TTS (free)
    (○) ElevenLabs
    (○) OpenAI TTS
    (○) Skip
```

→ **Microsoft TTS** Enter (무료 무료, 한국어 지원).

#### Search Provider

```text
Search provider:
  (1) Firecrawl Cloud (recommended)
  (2) Tavily
  (3) Brave Search
  → (4) Skip — configure later
```

→ **4 (Skip)** Enter.

> 💡 Firecrawl·Tavily는 외부 가입·API key 필요 → *나중에* `hermes config set`으로 추가 가능. 본 챕터는 핵심에 집중.

#### Slack Slash Commands

```text
Auto-register Slack slash commands? [Y/n]
```

→ **Enter** (기본값 Y) — `/hermes`, `/status` 등 자동 등록.

### 4-N. Setup 완료 → 컨테이너 백그라운드 기동 (1분)

wizard가 완료 메시지:

```text
╔══════════════════════════════════════════════════╗
║              Setup Complete!                     ║
╚══════════════════════════════════════════════════╝

Configuration saved to /data/config.yaml
Slack credentials saved to /data/.env

Next steps:
  - Start chatting: hermes
  - Run the gateway: hermes gateway
  - Run diagnostics: hermes doctor

Documentation: https://hermes-agent.nousresearch.com/docs/
```

→ wizard 종료 (`run --rm`이므로 컨테이너도 자동 삭제).

#### 1. `.env` 파일 확인

호스트 터미널에서:

```bash
$ ls -la ~/.hermes/
total ...
-rw-------  .env
-rw-r--r--  config.yaml
-rw-r--r--  auth.json
-rw-r--r--  slack-manifest.json
drwxr-xr-x  skills/
drwxr-xr-x  memories/
drwxr-xr-x  sessions/
drwxr-xr-x  logs/
```

`.env` 권한이 `-rw-------` (600) 인지 확인 — *자기만 읽을 수 있는* 상태.

#### 2. 백그라운드 gateway 기동

```bash
$ cd ~/hermes-stack
$ docker compose up -d gateway
[+] Running 2/2
 ✔ Network hermes-stack_default  Created
 ✔ Container hermes-gateway      Started
```

#### 3. 로그 확인 — Slack 연결 성공 메시지

```bash
$ docker compose logs -f gateway
```

다음 줄들이 차례로 출력되어야 합니다:

```text
hermes-gateway | starting gateway...
hermes-gateway | provider: openai-codex (gpt-5.5) — auth OK
hermes-gateway | slack: connecting to Socket Mode
hermes-gateway | slack: connected as @hermes (workspace: dangtong-pkm)
hermes-gateway | gateway ready — listening for messages
```

> ✅ **`slack: connected as @...`** 줄이 보이면 성공.

`Ctrl+C`로 로그 follow 종료 (컨테이너는 계속 실행).

> [!warning] **`slack: connecting...` 만 보이고 멈춤 — 디버깅**
>
> 1. **App-Level Token 잘못 입력** (가장 흔함): `xapp-...` 토큰이 누락·오타·다른 토큰
> 2. **`connections:write` scope 누락**: §4-J에서 토큰 생성 시 scope 안 추가
> 3. **워크스페이스에 앱 미설치**: §4-I의 "Install to Workspace" 안 함
>
> → 해결: `.env` 파일을 직접 열어 토큰 재확인 후 `docker compose restart gateway`. 부록 A 참조.

#### ✅ §4 검증 체크리스트

- [ ] `~/.hermes/.env` 파일 존재, 권한 `-rw-------`
- [ ] `~/.hermes/.env`에 `SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`, `SLACK_ALLOWED_USERS`, `SLACK_HOME_CHANNEL` 4개 키 포함
- [ ] `~/.hermes/slack-manifest.json` 존재 (참고용)
- [ ] `docker compose ps`에서 `hermes-gateway` 상태 `Up`
- [ ] `docker compose logs gateway`에 `slack: connected as @...` 줄 확인

§5에서 *첫 대화*를 나눕니다 — TUI와 Slack 양쪽으로.

---

## §5. 첫 대화 — TUI + Slack 양쪽 (10분, 실습)

설정이 다 끝났으니 이제 *실제로 말 걸어봅니다*. 두 가지 경로 — 터미널과 Slack — 둘 다 *같은 Hermes*에 연결됩니다.

### 5-A. TUI에서 첫 대화 (3분)

컨테이너 안의 Hermes TUI(Terminal UI)에 직접 접속:

```bash
$ docker compose exec -it gateway hermes
```

> 💡 **`run --rm`이 아니라 `exec`**: 컨테이너는 이미 `up -d`로 떠 있으니, *그 안에* 들어가서 TUI만 띄우는 명령. 종료해도 컨테이너는 살아있음.

TUI 화면:

```text
╔══════════════════════════════════════════════════╗
║  Hermes Agent — gpt-5.5 — session: 2026-05-23   ║
╚══════════════════════════════════════════════════╝

Hello! I'm Hermes. How can I help today?

> _
```

첫 인사:

```text
> 안녕, 너는 누구야?
```

→ Hermes가 한국어로 자기소개 + 능력 안내.

다음 시도:

```text
> 지금 날짜와 시간 알려줘
> 내 vault 경로는 어디야?
> 짧은 한 줄 시를 써줘
```

> ✅ **3가지가 모두 작동하면 TUI는 OK.**

종료:

```text
> /exit
```

또는 `Ctrl+D` — 컨테이너는 계속 백그라운드 실행.

### 5-B. Slack에서 첫 대화 (5분) ⭐

이게 이번 챕터의 *클라이맥스 모먼트*입니다.

#### 1. Slack 워크스페이스 열기

데스크탑 앱 또는 웹 — §3에서 준비한 워크스페이스로 이동.

#### 2. 봇이 자기 DM 또는 홈 채널에 보이는지 확인

좌측 사이드바 **Apps** 또는 **Direct messages** 아래에 봇 이름 (`Hermes Agent` 또는 §4-H에서 바꾼 이름)이 나타나야 합니다.

> 💡 **안 보이면?** 새로고침 (`Cmd/Ctrl+R`) → 그래도 없으면 좌측 사이드바 **+** → **Browse Slack** → **Apps** → 봇 검색.

#### 3. 봇에게 DM 보내기

봇 이름 클릭 → DM 창 열림 → 입력:

```text
안녕 처음 만나
```

→ Hermes가 *몇 초 안에* 응답.

> [!info] **첫 응답이 살짝 느립니다 (5~15초)**
>
> 컨테이너가 *콜드 스타트* 상태에서 OAuth 갱신·세션 초기화를 하는 첫 사이클. 두 번째 메시지부터는 빠릅니다.

#### 4. 홈 채널에서 멘션 시도

§3에서 만든 채널 (`#hermes-office`)로 이동:

봇이 *아직 채널에 없으면*: 채널 상단 채널명 클릭 → **Integrations** 탭 → **Add an App** → 봇 선택 → Add.

또는 채널에서 직접:

```text
/invite @<봇이름>
```

채널 멘션:

```text
@hermes 오늘 내가 이 챕터를 끝낸 걸 축하해줘
```

→ 봇이 **쓰레드로 답변**:

```text
🎉 축하해! Hermes Agent를 자기 PC에 직접 띄우고,
Slack 워크스페이스와도 연결한 거 — 결코 가벼운 일이 아니야.
검증 대역폭의 *시간대*가 확장됐어. 이제는 ...
```

> 💡 **Slack 쓰레드 = 대화 컨텍스트 단위**
>
> Hermes는 *각 쓰레드를 별개 세션*으로 관리합니다. 새 주제는 *새 메시지* (쓰레드 밖)로 시작하고, 같은 주제 후속 질문은 *기존 쓰레드 안*에서 답글로. 이게 메모리·컨텍스트 누수를 막는 *Slack 활용의 핵심*.

#### 5-C. (선택) 봇 이름·아이콘 사후 변경

처음에 *일반적인* 이름으로 만들었다면, *페르소나*를 부여하면 동료 느낌이 살아납니다.

1. <https://api.slack.com/apps> → 자기 앱 → **App Manifest**
2. JSON 안에서:

   ```json
   "bot_user": {
     "display_name": "민지",
     ...
   }
   ```

3. **Save Changes**
4. 좌측 **Install App** → **Reinstall to Workspace** ← *재설치 필수*
5. Slack에서 봇 이름이 자동 갱신됨 (몇 초 후)

> 💡 봇에 사람 이름을 부여하면 "Hermes에게 시킨다"가 아니라 "민지와 상의한다"로 *프레임이 바뀝니다*. PKM 운영자에게 *동료 감각*이 생기는 미세한 효과.

### ✅ §5 검증 체크리스트

- [ ] `docker compose exec -it gateway hermes`로 TUI 진입 성공
- [ ] TUI에서 3가지 질문 (한국어 자기소개·시간·짧은 시) 모두 응답
- [ ] Slack 사이드바 **Apps**에 봇 표시
- [ ] 봇 DM에서 첫 인사 응답 수신
- [ ] 홈 채널에서 멘션 → 쓰레드 답변 수신
- [ ] (선택) 봇 이름 커스터마이즈 + Reinstall

---

## §6. 운영 체크리스트 (5분)

이 시점부터 Hermes는 *상주 동료*입니다. 그렇다면 *관리*가 필요합니다. *일상 명령 6가지*만 외워두면 99% 운영 가능.

### 6-A. 매일 쓰는 6개 명령

#### 1. 컨테이너 상태 확인

```bash
$ docker compose ps
NAME             IMAGE                                          STATUS         PORTS
hermes-gateway   ghcr.io/nousresearch/hermes-agent:latest       Up 2 hours     ...
```

`Up`이면 정상. `Exited`·`Restarting`이면 §부록 A.

#### 2. 실시간 로그 따라가기

```bash
$ docker compose logs -f gateway
```

`Ctrl+C`로 빠져나옴 (컨테이너는 계속 실행).

특정 줄 수만:

```bash
$ docker compose logs --tail=50 gateway
```

#### 3. Hermes 자체 진단

```bash
$ docker compose exec gateway hermes doctor
```

체크 항목:

```text
✓ Hermes home: /data
✓ Config: /data/config.yaml (valid)
✓ Provider: openai-codex (auth OK)
✓ Model: gpt-5.5 (available)
✓ Gateway: running (pid 12)
✓ Slack: connected as @hermes (workspace: dangtong-pkm)
✓ Skills: 87 bundled, 0 missing
```

전부 `✓`이면 OK. `✗`가 있으면 그 줄이 *문제의 출발점*.

#### 4. Gateway 상태

```bash
$ docker compose exec gateway hermes gateway status
```

```text
Gateway: running
Connected platforms:
  - slack: @hermes (last message: 3 minutes ago)
Active sessions: 2
```

#### 5. 업그레이드 (월 1회 권장)

```bash
$ cd ~/hermes-stack
$ docker compose pull        # 최신 이미지 받기
$ docker compose up -d       # 새 이미지로 재기동
```

`~/.hermes/`의 데이터·메모리·skills 모두 *그대로 유지*되며 컨테이너만 새 버전으로 교체됩니다.

#### 6. 잠시 멈춤·재개

```bash
$ docker compose stop gateway     # 종료 (데이터 유지)
$ docker compose start gateway    # 재시작
$ docker compose restart gateway  # 재기동 (설정 변경 후)
```

### 6-B. 백업 — `~/.hermes/` 통째로

`~/.hermes/` 디렉토리 *하나*가 모든 학습·기억·설정의 *유일한 상태*입니다.

#### 백업 (월 1회 권장)

```bash
$ tar czf ~/hermes-backup-$(date +%Y%m%d).tar.gz -C ~ .hermes
$ ls -lh ~/hermes-backup-*.tar.gz
```

`.hermes/`에는 *세션 히스토리·메모리*가 누적되어 시간이 지날수록 커집니다. 백업 파일도 그만큼 증가 — 외장 드라이브·클라우드로 주기적 이관 권장.

#### 복원

```bash
$ docker compose down
$ rm -rf ~/.hermes  # ⚠️ 기존 데이터 폐기
$ tar xzf ~/hermes-backup-20260501.tar.gz -C ~
$ docker compose up -d
```

### 6-C. 보안 체크리스트

- [ ] **`.env` 절대 commit 금지** — `~/hermes-stack/.gitignore`에 `.env` 추가 (compose 디렉토리에 git 쓸 거면)
- [ ] **`SLACK_ALLOWED_USERS` 정확히** — 다른 사람이 봇에게 말 걸어도 무시되도록
- [ ] **`.env` 권한 600** — `chmod 600 ~/.hermes/.env`
- [ ] **컨테이너 포트 노출 X** — 본 챕터의 `docker-compose.yaml`은 포트 매핑 없음. *유지*

### 6-D. 컨테이너 완전 제거 (필요 시)

학습이 끝나거나 다시 시작하려면:

```bash
$ docker compose down -v         # 컨테이너 + volume 제거
$ docker image rm ghcr.io/nousresearch/hermes-agent:latest  # 이미지 제거
$ rm -rf ~/.hermes               # ⚠️ 모든 데이터 삭제
$ rm -rf ~/hermes-stack          # compose 파일 제거
```

> [!warning] **`docker compose down -v`의 `-v`는 *볼륨 제거* 옵션**
>
> 본 챕터는 *named volume*을 안 쓰고 bind mount만 사용하므로 `-v`로 `~/.hermes/`가 *자동 삭제되지는 않습니다*. 하지만 *습관적으로 `-v`를 쓴다면* 다른 환경에서 named volume을 *날릴 수 있음*. 의도 확인 후 사용.

---

## §7. PKM 워크플로우에 붙이기 (5분, 회고)

Hermes 자체는 떴습니다. 그런데 *Vault에 직접 쓰지는 못합니다* — 컨테이너 안에 갇혀있으니까. **vault 마운트**를 추가해야 시나리오가 살아납니다.

### 7-A. Vault Bind Mount 추가

`~/hermes-stack/docker-compose.yaml`을 수정:

```yaml
services:
  gateway:
    image: ghcr.io/nousresearch/hermes-agent:latest
    container_name: hermes-gateway
    restart: unless-stopped
    command: gateway
    env_file: /Users/dangtongbyun/.hermes/.env
    volumes:
      - ~/.hermes:/data
      - ${VAULT_ROOT}:/vault     # ⬅️ 추가
    environment:
      - HERMES_HOME=/data
      - VAULT_ROOT=/vault        # ⬅️ 컨테이너 안에서도 변수 사용
    stdin_open: true
    tty: true
```

#### 1. `$VAULT_ROOT` 환경변수 확인

호스트 셸에서:

```bash
$ echo $VAULT_ROOT
/Users/dangtongbyun/Obsidian/dangtong-lecture
```

비어있으면 Chapter 05 Step 6 도입부 박스의 `export` 명령 다시 실행.

#### 2. 컨테이너 재기동

```bash
$ cd ~/hermes-stack
$ docker compose down
$ docker compose up -d gateway
```

#### 3. 마운트 확인

```bash
$ docker compose exec gateway ls /vault
00_시스템   02_프로젝트   04_지식창고   06_첨부파일
01_수신함   03_관리영역   05_보관       AGENTS.md
```

→ 컨테이너 안에서 vault가 *직접 보입니다*.

### 7-B. 시나리오 — Slack 한 줄 → Inbox 자동 적재

Slack에서:

```text
@hermes 다음 한 줄을 vault의 01_수신함에 새 노트로 저장해줘:
"검증 대역폭은 도구가 아니라 시간대로 늘릴 수 있다."
파일명은 오늘 날짜 + 한 줄 핵심으로.
```

→ Hermes가:

1. `/vault/01_수신함/` 경로 인식
2. 파일명 제안 (예: `2026-05-23-검증대역폭-시간대.md`)
3. **승인 게이트** — "이 파일을 생성할까요?"
4. 승인 → frontmatter 포함 노트 생성

호스트에서 확인:

```bash
$ ls $VAULT_ROOT/01_수신함/ | head -3
2026-05-23-검증대역폭-시간대.md
...
```

→ Obsidian이 *자동으로 감지* (vault watch). 책상에 돌아왔을 때 *이미 거기 있음*.

### 7-C. 두 도구의 합주 — Slack에서 시작, OpenCode에서 정제

```text
[출퇴근 길]
  Slack DM → @hermes "오늘 떠오른 PKM 인사이트 1줄"
  → Hermes가 01_수신함/에 raw 노트 적재

[책상 앞]
  $ cd $VAULT_ROOT && opencode
  /todo-add raw 노트 정제 (Chapter-05 Step 6 Command)
  /meeting-clean (회의록이라면)
  vault-audit (주간 점검)
  → 04_지식창고/로 승격
```

**Hermes = 입력 채널**, **OpenCode = 정제 워크벤치**. 두 도구가 *서로 보완*합니다.

### 7-D. 1줄 회고 — Daily Note에 추가

오늘 Daily Note의 `## 💡 배운 것` 섹션에:

> 데스크탑 도구만 있을 때와 *상주 동료*가 있을 때의 차이는?  
> 검증 대역폭은 *시간대*로도 확장된다 — OpenCode(책상)·Hermes(이동 중)·cron(새벽).  
> 다음 주에 *출퇴근길 1회* 이상 Hermes에게 PKM raw 던져보기.

---

## 부록 A. 트러블슈팅

### 공통

#### Docker Desktop이 안 뜸 (Windows)

증상: 트레이에 고래 아이콘 *빨간색*, 또는 "Docker Desktop is starting..." 무한 대기.

해결 순서:

1. PowerShell 관리자:

   ```powershell
   wsl --shutdown
   wsl --update
   ```

2. Docker Desktop 우클릭 → **Restart**
3. BIOS에서 Intel VT-x / AMD-V 활성화 확인
4. Windows 기능: **Virtual Machine Platform** + **Windows Subsystem for Linux** 둘 다 체크

#### Docker Desktop이 안 뜸 (macOS)

증상: 메뉴바 고래가 회색·"Docker Desktop is starting..." 무한.

해결:

1. **시스템 설정 → 개인정보·보안 → 전체 디스크 접근** → Docker 추가
2. Docker Desktop 종료 후 재실행
3. 그래도 안 되면: `~/Library/Group Containers/group.com.docker` 삭제 후 재설치

#### OAuth 디바이스 코드가 만료됨

증상: 브라우저에 코드 입력 후 "Code expired" 표시.

해결:

```bash
$ docker compose run --rm -it gateway hermes setup
```

→ Quick setup → 모든 답변이 *이전 값으로 표시됨* → Enter만 누르며 통과 → OAuth 단계에서 *새 코드* 발급.

#### `slack: connecting...` 무한 (가장 흔한 장애)

증상: 로그에 connecting만 나오고 `connected as` 안 뜸.

원인 + 해결:

```bash
# 1) Token 직접 확인
$ cat ~/.hermes/.env
SLACK_BOT_TOKEN=xoxb-...
SLACK_APP_TOKEN=xapp-...   ← 둘 다 있는지
SLACK_ALLOWED_USERS=U01...

# 2) App-Level Token scope 확인
# 브라우저: api.slack.com/apps → 자기앱 → Basic Information
# → App-Level Tokens → "hermes-socket" 토큰의 Scopes
# → connections:write 있어야 함

# 3) 워크스페이스에 앱 설치 확인
# 브라우저: 자기앱 → Install App
# → "Reinstall to Workspace" 버튼이 있으면 설치 완료 상태

# 4) 재기동
$ docker compose restart gateway
$ docker compose logs -f gateway
```

#### 봇이 채널 메시지를 못 받음

증상: DM은 되는데 채널 멘션은 무응답.

원인:

- 봇이 *채널에 없음*: 채널에서 `/invite @봇이름`
- 채널이 *Private*인데 봇이 멤버 아님: Private 채널 설정 → Members → Add
- `SLACK_ALLOWED_USERS`에 자기 ID 누락: `.env` 확인 후 `restart`

#### 컨테이너에서 vault 못 봄

증상: `docker compose exec gateway ls /vault` → "No such file or directory".

원인 + 해결:

```bash
# 1) $VAULT_ROOT 호스트에서 정의됐는지
$ echo $VAULT_ROOT

# 2) docker compose.yaml의 volumes에 ${VAULT_ROOT}:/vault 있는지
$ grep -A2 volumes ~/hermes-stack/docker-compose.yaml

# 3) compose가 환경변수를 읽도록 — .env 또는 export 후 재기동
$ export VAULT_ROOT=~/Obsidian/dangtong-lecture
$ cd ~/hermes-stack
$ docker compose down && docker compose up -d
```

#### WSL2에서 `/mnt/c/...` vault 접근이 매우 느림

증상: vault가 Windows 드라이브에 있을 때 Hermes가 vault 작업에서 수초~수십 초 멈춤.

원인: WSL2의 9P 프로토콜 *교차 파일시스템* 성능 저하 (Chapter 05 §9-G와 같은 이슈).

해결:

- vault를 *WSL 홈*(`~/Obsidian/`)으로 복사
- 또는 빈도가 낮은 작업이면 그대로 두기 (느릴 뿐 동작은 함)

---

## 부록 B. OpenClaw 사용자에게

이 챕터는 **clean-start 기준**으로 작성되었습니다. 기존에 OpenClaw를 사용 중이라면:

```bash
$ docker compose exec gateway hermes claw migrate
```

→ Hermes가 `~/.openclaw/`를 감지해 *임포트 가능한 항목 미리보기* 제시. 동의 시 마이그레이션 진행.

⚠️ **마이그레이션 주의**:

- *기존 OpenClaw의 Slack 워크스페이스*가 그대로 연결될 수 있음
- *민감 설정·시크릿*이 함께 옮겨짐 → 의도하지 않은 채널에 알림 갈 위험
- 본 챕터의 *학습용 워크스페이스*를 깨끗하게 쓰고 싶다면 **migrate 안 하는 것을 권장**

`hermes claw migrate`는 *언제든* 다시 실행 가능 — 이번 챕터를 *깨끗하게* 마친 뒤 별도 결정.

---

## 📦 산출물 (학생 환경에 남는 것)

이 챕터를 완료하면 다음이 *학생 PC·Slack 워크스페이스에 남습니다*:

### 시스템 설정
- [ ] Docker Desktop (macOS / Windows WSL Integration)
- [ ] `~/.hermes/` 디렉토리 (config·skills·memory·sessions·logs·`.env`)
- [ ] `~/hermes-stack/docker-compose.yaml`
- [ ] `~/.hermes/slack-manifest.json` (참고용)

### Slack 워크스페이스
- [ ] Hermes Agent 앱 등록 + Socket Mode 활성
- [ ] Bot Token (`xoxb-...`) · App-Level Token (`xapp-...`)
- [ ] 홈 채널 (`#hermes-office`) + 봇 멤버십
- [ ] 첫 대화 기록 (DM + 멘션 쓰레드)

### Vault
- [ ] `docker-compose.yaml`에 `$VAULT_ROOT:/vault` 마운트 추가
- [ ] `01_수신함/`에 Slack 발 raw 노트 1건 이상
- [ ] Daily Note `## 💡 배운 것`에 1줄 회고

### 인식 변화
- [ ] **OpenCode = 책상 워크벤치, Hermes = 이동 중 동료** — *시간대 분리*
- [ ] **컨테이너 = 호스트 보호 + cleanup + 재현성**
- [ ] **Socket Mode = inbound 포트 없는 통신** — 사내·집·VPS 모두 동일
- [ ] **Slack 쓰레드 = 세션 컨텍스트 단위**

---

## 🧭 챕터 마무리

```text
검증 대역폭의 *깊이*    →  지금까지의 챕터들 (OpenCode·MCP·Skill·LLM Wiki)
검증 대역폭의 *시간대*  →  이번 챕터 (Hermes Agent)
```

지금까지 책상에서만 가능했던 PKM 작업이 *출퇴근길·점심시간·새벽*에도 가능해졌습니다. **두 번째 도구가 아니라, *다른 시간대의 도구*** — 이 챕터의 메타 메시지입니다.

> 🎯 **다음 주 자기 도전**:
>
> - 이번 주 *3회 이상* Slack에서 Hermes에게 raw 자료 던지기
> - 그중 1건을 OpenCode에서 정제해 `04_지식창고/`로 승격
> - 두 도구가 *합주*한 흔적을 Daily Note `## 💡 배운 것`에 기록

**다음 챕터**: 작성 예정 — Hermes의 *Skill 자동 학습·cron 스케줄·다중 페르소나 (프로필)* 활용으로 *상주 동료를 더 똑똑하게* 만드는 방법.

