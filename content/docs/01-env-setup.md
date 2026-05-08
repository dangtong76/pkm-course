---
title: "01. 환경 설치"
weight: 1
date: 2026-05-04
draft: false
---

> **⏰ 시간**: 1시간 (이론 슬라이드 30분 + 실습 30분)
> **🎯 목표**: OpenCode + Oh-My-OpenCode + Obsidian 환경 완벽 구축

---

## 📄 이론 슬라이드

> 이 챕터의 이론(환경 구성 전체 그림·CLI 에이전트 비교·왜 이 조합인가 등)은 아래 슬라이드로 다룹니다. 슬라이드를 먼저 보거나 강의를 들은 뒤 아래 실습으로 넘어가세요.

<div style="border: 2px dashed #cbd5e1; border-radius: 8px; padding: 48px 24px; text-align: center; background: #f8fafc; margin: 16px 0;">
  <div style="font-size: 48px; margin-bottom: 12px;">📊</div>
  <div style="font-size: 18px; font-weight: 600; color: #475569; margin-bottom: 8px;">
    슬라이드 PDF 준비 중
  </div>
  <div style="font-size: 14px; color: #64748b; margin-bottom: 16px;">
    이론 슬라이드는 별도 PDF로 제공될 예정입니다.<br>
    PDF 파일을 받으셨다면 <code style="background:#e2e8f0;padding:2px 6px;border-radius:4px;">static/pdf/01-env-setup.pdf</code> 위치에 저장해 주세요.
  </div>
  <div style="font-size: 12px; color: #94a3b8;">
    PDF가 추가되면 이 자리에 자동으로 슬라이드가 표시됩니다.
  </div>
</div>

<!-- PDF가 준비되면 위 placeholder를 아래 iframe으로 교체:
<iframe src="/pdf/01-env-setup.pdf"
        width="100%"
        height="600px"
        style="border: 1px solid #ddd; border-radius: 4px;">
  PDF를 표시할 수 없습니다. <a href="/pdf/01-env-setup.pdf">다운로드</a>하세요.
</iframe>

> 📥 [PDF 다운로드](/pdf/01-env-setup.pdf) · [전체 화면 보기](/pdf/01-env-setup.pdf)
-->


---

## 📚 학습 목표

이 실습을 마치면 다음을 할 수 있습니다:

- [ ] OpenCode가 정상 실행되고 API 키가 설정되어 있다
- [ ] Oh-My-OpenCode 플러그인이 설치되어 Sisyphus 에이전트가 활성화되어 있다
- [ ] Obsidian이 설치되고 첫 Vault가 생성되어 있다
- [ ] OpenCode가 Vault에 파일을 직접 만들 수 있는 것을 확인했다

---

## 🛠️ 사전 요구사항

| 항목 | 요구 사항 |
|------|---------|
| **터미널** | iTerm2 / Warp / Windows Terminal + WSL / Alacritty 등 |
| **LLM API 키** | Anthropic Claude / OpenAI / OpenCode Zen 중 하나 이상 |
| **OS** | macOS, Linux, Windows (WSL 권장) |
| **인터넷** | 패키지 다운로드 + API 호출 |

---

## 🪜 실습 단계

### Step 1. Oh My Zsh 설치 (10분, 선택)

> 💡 **이미 설치되어 있다면 Step 2로 건너뛰세요.** Oh My Zsh는 터미널 생산성을 크게 높여주지만 필수는 아닙니다.

#### 1-1. Zsh 확인

```bash
echo $SHELL
```

`zsh`가 출력되면 OK. 다른 셸이라면:

```bash
# macOS: 기본 셸이 Zsh (별도 설치 불필요)
# Ubuntu/Debian:
sudo apt install zsh
# RHEL/Fedora:
sudo dnf install zsh

# 기본 셸을 Zsh로 변경
chsh -s $(which zsh)
```

#### 1-2. Oh My Zsh 설치

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

설치 완료 시 `~/.zshrc` 파일이 자동 생성됩니다.

#### 1-3. 추천 플러그인 설정

```bash
# 자동 완성 플러그인
git clone https://github.com/zsh-users/zsh-autosuggestions \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# 구문 강조 플러그인
git clone https://github.com/zsh-users/zsh-syntax-highlighting \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

`~/.zshrc`에서 활성화:

```bash
plugins=(
  git
  zsh-autosuggestions
  zsh-syntax-highlighting
)
```

```bash
source ~/.zshrc
```

> 💡 **팁**: `git` 플러그인은 `gst`(git status), `gco`(git checkout) 등 편리한 alias를 제공합니다.

---

### Step 2. OpenCode 설치 (5분)

#### 2-1. macOS 설치

**방법 1: 설치 스크립트 (권장)**
```bash
curl -fsSL https://opencode.ai/install | bash
```

**방법 2: Homebrew**
```bash
brew install anomalyco/tap/opencode
```

**방법 3: npm**
```bash
npm install -g opencode-ai
```

#### 2-2. Windows 설치

**방법 1: PowerShell 스크립트 (권장)**
```powershell
irm https://opencode.ai/install.ps1 | iex
```

**방법 2: Chocolatey**
```powershell
choco install opencode
```

**방법 3: Scoop**
```powershell
scoop bucket add extras
scoop install opencode
```

> 💡 **Windows 팁**: WSL(Windows Subsystem for Linux)을 사용하면 macOS/Linux와 동일한 명령어를 사용할 수 있습니다.

#### 2-3. 설치 확인

```bash
opencode --version
```

예상 출력:
```
opencode version x.x.x
```

✅ 버전 번호가 출력되면 설치 완료.

---

### Step 3. Oh-My-OpenAgent (omo) 설치 (5분)

Oh-My-OpenAgent는 **Sisyphus, Hephaestus, Prometheus, Oracle, Librarian, Explore** 등 멀티 에이전트 시스템과 LSP·AST-Grep·Hash-Anchored Edit 같은 도구를 한 번에 묶어주는 핵심 플러그인입니다.

> 💡 **이름 안내**
> - 프로젝트 이름은 **`oh-my-openagent`** (줄여서 **`omo`**)로 리브랜딩되었습니다.
> - 다만 npm 패키지·CLI 바이너리·플러그인 설정 파일명은 **호환성 유지**를 위해 여전히 `oh-my-opencode`를 사용합니다.
> - `opencode.json` 안에서는 `oh-my-openagent` 항목이 우선이며, 옛 `oh-my-opencode` 항목도 경고와 함께 그대로 동작합니다.

#### 3-1. 방법 1 — 에이전트에게 설치 시키기 (공식 권장)

공식 README가 가장 강하게 권장하는 방법입니다. **OpenCode TUI(또는 Claude Code, Cursor 등 다른 LLM 에이전트)** 에 아래 프롬프트를 그대로 붙여넣으세요.

```
Install and configure oh-my-openagent by following the instructions here:
https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/refs/heads/dev/docs/guide/installation.md
```

에이전트가 위 URL의 설치 가이드를 직접 읽고 — `opencode.json` 수정, 플러그인 등록, 모델 설정까지 — 알아서 처리합니다.

> 💡 **왜 이 방법인가?**
> 공식 README의 표현을 빌리면 *"진심으로, 에이전트한테 시키세요. 사람은 설정 파일을 오타로 망칩니다."* `opencode.json`은 JSONC 형식 + 모델 카테고리 매핑 + 플러그인 배열 등 손댈 곳이 많아 직접 편집하면 실수하기 쉽습니다.

#### 3-2. 방법 2 — 설치 가이드를 직접 따라 하기

에이전트에게 맡기는 게 꺼려진다면, 같은 가이드를 사람이 직접 읽고 따라 할 수 있습니다.

- 공식 설치 가이드: <https://github.com/code-yeongyu/oh-my-openagent/blob/dev/docs/guide/installation.md>
- 한국어 README (전체 맥락): <https://github.com/code-yeongyu/oh-my-openagent/blob/dev/README.ko.md>

핵심은 `~/.config/opencode/opencode.json`(또는 `opencode.jsonc`)의 `plugin` 배열에 `"oh-my-openagent"` 항목을 추가하고, 카테고리별 모델 매핑을 설정하는 것입니다. 자세한 단계는 위 가이드를 따르세요.

#### 3-3. 설치 검증 — `doctor` 명령어

omo는 빌트인 진단 도구를 제공합니다. 플러그인 등록·설정 파일·모델 매핑·환경 변수까지 한 번에 점검합니다.

```bash
bunx oh-my-opencode doctor
```

> ⚠️ CLI 바이너리 이름은 호환성 때문에 여전히 `oh-my-opencode`입니다. 정상입니다.

모든 항목에 ✅가 뜨면 설치 완료.

#### 3-4. OpenCode 재시작 후 에이전트 확인

```bash
opencode
# TUI에서 입력
/agents
```

목록에 다음 에이전트들이 보이면 OK:

| 에이전트 | 역할 |
|---------|------|
| **Sisyphus** | 메인 오케스트레이터 (계획·위임·병렬 실행) |
| **Hephaestus** | 자율 실행 작업자 (목표만 주면 엔드투엔드 처리) |
| **Prometheus** | 전략 플래너 (인터뷰 모드로 스코프 확정) |
| **Oracle** | 아키텍처·디버깅 컨설턴트 (read-only) |
| **Librarian** | 외부 문서·OSS·웹 검색 |
| **Explore** | 코드베이스 grep·탐색 |

> 💡 PKM 강의에서는 이 중 **Sisyphus**(주로 대화)와 **Librarian/Explore**(자료 조사)를 자주 활용합니다.

---

### Step 4. LLM 모델 연결 (5분)

OpenCode는 여러 LLM 제공자를 지원합니다. 본인 상황에 맞는 옵션 하나만 선택해 진행하세요.

> 🎯 **수업 표준은 옵션 1 (ChatGPT 구독 OAuth 로그인)** 입니다. ChatGPT Plus/Pro/Max 구독이 있다면 이 방식만 따라오시면 됩니다.

#### 4-1. OpenCode TUI에서 `/connect` 실행

모든 옵션은 동일하게 `/connect` 명령어로 시작합니다.

```bash
opencode
# TUI에서 입력
/connect
```

#### 4-2. 옵션 1 — OpenAI ChatGPT 구독 (수업 권장) ⭐

ChatGPT Plus / Pro / **Max** 구독이 있다면 별도 API 키 발급·결제 없이 OAuth 한 번으로 로그인됩니다. 공식 Codex CLI와 동일한 인증 흐름입니다.

**1. `/connect` 실행 후 `OpenAI` 선택**

```text
┌ Select provider
│ ...
│ ● OpenAI
└
```

**2. `ChatGPT Plus/Pro` 선택** — 브라우저가 자동으로 열립니다

```text
┌ Select auth method
│
│ ● ChatGPT Plus/Pro
│   Manually enter API Key
└
```

> 💡 **Max 구독자도 이 옵션을 선택하세요.** 공식 문서 라벨은 "ChatGPT Plus/Pro"이지만, Max는 상위 등급이라 동일한 OAuth 흐름으로 인증됩니다.

**3. 브라우저에서 ChatGPT 계정으로 로그인** → 권한 승인 → 터미널로 자동 복귀

**4. 사용 가능한 모델 확인**

```bash
# TUI에서 입력
/models
```

ChatGPT 구독 OAuth로 로그인하면 OpenAI 모델군(예: `gpt-5.4`, `gpt-5.4-mini`, `gpt-5.5`, `gpt-5-pro`)이 한꺼번에 사용 가능해집니다. 어느 모델을 어느 작업에 매핑할지는 다음 단계(4-2-A)에서 한 번에 설정합니다.

> 💡 omo의 `ultrawork`나 카테고리 라우팅을 쓰면 작업 성격에 따라 적합한 모델이 자동 선택되므로, 학생이 매번 `/models`로 모델을 바꿀 일은 거의 없습니다.

#### 4-2-A. 에이전트별 모델 라우팅 설정 (필수)

ChatGPT 구독으로 로그인했다면 **omo의 어떤 에이전트가 어떤 OpenAI 모델을 쓸지** 를 직접 매핑해줘야 합니다. 이 매핑이 없으면 omo가 기본 모델(주로 Anthropic 계열)을 호출하려다 실패합니다.

**1. 설정 파일 위치**

omo는 다음 경로의 `oh-my-opencode.jsonc` 또는 `oh-my-opencode.json` 파일을 자동으로 인식합니다.

```
~/.config/opencode/oh-my-opencode.jsonc
```

> ⚠️ 파일명은 호환성 때문에 여전히 `oh-my-opencode`(기존 이름)를 사용합니다. 이게 정상입니다.

파일이 없다면 새로 만드세요:

```bash
mkdir -p ~/.config/opencode
touch ~/.config/opencode/oh-my-opencode.jsonc
```

**2. 에이전트별 모델 매핑 표**

수업에서 사용하는 매핑입니다. **OpenAI 모델은 작업 성격에 따라 4단계로 나눠 배정**합니다 — 추론 강도가 높을수록 비싸고 느린 모델을 씁니다.

| 에이전트 | 모델 | reasoningEffort | 역할 한 줄 |
|---------|------|----------------|-----------|
| **sisyphus** | `openai/gpt-5.4` | `high` | 메인 오케스트레이터 |
| **sisyphus-junior** | `openai/gpt-5.4` | `high` | Sisyphus 위임 작업자 |
| **prometheus** | `openai/gpt-5.5` | `high` | 전략 플래너 (인터뷰 모드) |
| **oracle** | `openai/gpt-5-pro` | `xhigh` | 아키텍처·디버깅 컨설턴트 (가장 비싼 모델) |
| **momus** | `openai/gpt-5.5` | `xhigh` | 플랜 리뷰어 |
| **metis** | `openai/gpt-5.5` | `high` | 플랜 사전 컨설턴트 |
| **librarian** | `openai/gpt-5.4-mini` | `medium` | 외부 문서·OSS 검색 (경량) |
| **explore** | `openai/gpt-5.4-mini` | `low` | 코드베이스 grep (경량·빠름) |
| **multimodal-looker** | `openai/gpt-5.4-mini` | `high` | PDF·이미지 분석 |
| **frontend-ui-ux-engineer** | `openai/gpt-5.4` | `high` | UI/UX 작업 (PKM에선 거의 미사용) |
| **document-writer** | `openai/gpt-5.4` | `medium` | 문서 작성 (PKM 핵심 활용) |
| **atlas** | `openai/gpt-5.4-mini` | `medium` | 보조 작업자 |

> 💡 **설계 원리**
> - **`gpt-5-pro` (xhigh)** — Oracle 단 하나. 어려운 아키텍처·디버깅에만. 토큰 비싸므로 절제.
> - **`gpt-5.5` (high/xhigh)** — Prometheus·Momus·Metis 등 "플래닝/검토" 라인. 의도 파악과 누락 잡기.
> - **`gpt-5.4` (high)** — Sisyphus 메인 라인. 일상 실행의 기본 엔진.
> - **`gpt-5.4-mini` (low/medium)** — Librarian·Explore 등 "검색·요약" 라인. 빠르고 저렴.

**3. 전체 설정 파일 (그대로 복사)**

위 매핑을 JSON으로 옮긴 완성본입니다. `~/.config/opencode/oh-my-opencode.jsonc`에 그대로 붙여넣으세요.

```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",
  "agents": {
    "atlas": {
      "model": "openai/gpt-5.4-mini",
      "reasoningEffort": "medium",
      "variant": "medium"
    },
    "sisyphus": {
      "model": "openai/gpt-5.4",
      "reasoningEffort": "high",
      "variant": "high"
    },
    "sisyphus-junior": {
      "model": "openai/gpt-5.4",
      "reasoningEffort": "high",
      "variant": "high"
    },
    "oracle": {
      "model": "openai/gpt-5-pro",
      "reasoningEffort": "xhigh",
      "variant": "xhigh"
    },
    "prometheus": {
      "model": "openai/gpt-5.5",
      "reasoningEffort": "high",
      "variant": "high"
    },
    "momus": {
      "model": "openai/gpt-5.5",
      "reasoningEffort": "xhigh",
      "variant": "xhigh"
    },
    "metis": {
      "model": "openai/gpt-5.5",
      "reasoningEffort": "high",
      "variant": "high"
    },
    "librarian": {
      "model": "openai/gpt-5.4-mini",
      "reasoningEffort": "medium",
      "variant": "medium"
    },
    "explore": {
      "model": "openai/gpt-5.4-mini",
      "reasoningEffort": "low",
      "variant": "low"
    },
    "frontend-ui-ux-engineer": {
      "model": "openai/gpt-5.4",
      "reasoningEffort": "high",
      "variant": "high"
    },
    "document-writer": {
      "model": "openai/gpt-5.4",
      "reasoningEffort": "medium",
      "variant": "medium"
    },
    "multimodal-looker": {
      "model": "openai/gpt-5.4-mini",
      "reasoningEffort": "high",
      "variant": "high"
    }
  }
}
```

**4. 적용 확인**

OpenCode를 재시작하고 진단을 다시 돌립니다.

```bash
bunx oh-my-opencode doctor
```

`agents` 섹션의 각 에이전트가 위 매핑대로 OpenAI 모델로 잡혀 있으면 OK.

또한 OpenCode TUI에서 `/models`를 입력하고 `openai`로 필터하면 ChatGPT Max 구독으로 접근 가능한 모델 전체가 보입니다.

```text
Select model
openai│
  GPT-5.3 Codex Spark        OpenAI
  GPT-5.4 Fast               OpenAI
  GPT-5.4 mini               OpenAI
  GPT-5.4 mini Fast          OpenAI
  GPT-5.5 Fast               OpenAI
  GPT-5.5 Pro                OpenAI
  GPT-5.5                    OpenAI
  GPT-5.4                    OpenAI
  GPT-5.3 Codex              OpenAI
  GPT-5.2                    OpenAI
```

위 JSON에서 사용하는 `openai/gpt-5.4`, `openai/gpt-5.5`, `openai/gpt-5-pro`(= GPT-5.5 Pro), `openai/gpt-5.4-mini`는 모두 **이 OAuth 경로로 직접 호출**되며 별도 API 키나 추가 결제가 필요 없습니다 (ChatGPT 구독에 포함).

#### 4-3. 옵션 2 — OpenCode Zen (별도 구독 시)

OpenAI 모델뿐 아니라 **Anthropic Claude·Google Gemini 등 여러 벤더 모델을 하나의 키로 통합**해서 쓰고 싶을 때 선택합니다 (Pay-as-you-go).

1. [opencode.ai/zen](https://opencode.ai/zen) 접속 → 회원가입 → API 키 생성
2. `/connect` 실행 후 `OpenCode Zen` 선택
3. API 키 붙여넣기
4. `/models`에서 `claude-opus-4-7`, `gemini-3.1-pro` 등 비-OpenAI 모델까지 함께 선택 가능

> 💡 **언제 옵션 2를 쓰나?** OpenAI 모델만 쓸 거면 **옵션 1로 충분**합니다 (ChatGPT 구독에 포함). Claude·Gemini를 섞어 쓰는 멀티벤더 워크플로우가 필요할 때만 옵션 2를 추가로 고려하세요.

#### 4-4. 옵션 3 — Anthropic Claude (Pro/Max 구독 또는 API 키)

Claude를 OpenCode에서 쓰려면 **두 가지 경로** 중 하나를 선택해야 합니다. 그런데 한 가지 중요한 사실을 먼저 알아야 합니다.

> ⚠️ **Anthropic의 정책 — 공식 OpenCode `/connect`는 Claude 구독을 지원하지 않습니다**
>
> Anthropic은 **자사 공식 클라이언트(claude.ai, Claude Code) 외의 3rd-party 앱에서 Claude Pro/Max 구독을 사용하는 것을 공식적으로는 허용하지 않습니다.** OpenAI(ChatGPT)가 OpenCode `/connect`에 OAuth 옵션을 제공하는 것과는 대조적입니다.
>
> 따라서 OpenCode TUI의 `/connect` → `Anthropic`을 고르면 **API 키 입력 화면만** 나옵니다. 구독으로 바로 로그인하는 옵션은 제공되지 않습니다.

##### 4-4-A. 경로 1 — 정식 API 키 (권장, 합법)

[console.anthropic.com](https://console.anthropic.com)에서 종량제 API 키를 발급받아 사용. Anthropic이 공식 지원하는 정통 경로입니다.

1. [console.anthropic.com](https://console.anthropic.com) → **API Keys**에서 새 키 생성 → 결제 수단 등록
2. OpenCode TUI에서 `/connect` 실행 → `Anthropic` 선택
3. API 키 입력

**비용**: Pay-per-token (Claude Pro/Max 구독과 **별도** 청구). 구독료를 이미 내고 있어도 API 호출은 따로 결제됩니다.

##### 4-4-B. 경로 2 — Pro/Max 구독 OAuth (커뮤니티 플러그인, 회색지대)

[ex-machina-co/opencode-anthropic-auth](https://github.com/ex-machina-co/opencode-anthropic-auth) 플러그인을 사용하면 **이미 결제 중인 Claude Pro/Max 구독을 OpenCode에서도 사용**할 수 있습니다 (추가 API 비용 0).

> 🚨 **반드시 읽고 결정하세요 — 사용자 책임**
>
> 이 플러그인은 Anthropic 공식 제품이 아닙니다. 메인테이너 본인이 README에 다음과 같이 경고합니다:
>
> - *"This plugin comes with no guarantees. **You might be banned for breaking the TOS, you might not be.**"*
> - *"Plugins like **oh-my-openagent are known to trigger bans**."*
> - Ralph Loop·`ultrawork` 같은 **과도한 자동화 사용 패턴은 자제** 권고
>
> 즉 — Anthropic이 비정상 트래픽 패턴을 감지하면 **본인의 Claude 구독 계정이 정지될 수 있습니다.** 자기 책임 하에 사용 여부를 결정하세요.
>
> **수업에서는 안전을 위해 4-4-A(정식 API 키) 또는 옵션 1(ChatGPT OAuth)을 권장합니다.** 이 경로 2는 *"이런 방법도 있다"* 는 정보 제공 목적입니다.

**사용 절차** (위 위험을 감수하기로 결정한 경우):

**1. `~/.config/opencode/opencode.json`(또는 `opencode.jsonc`)의 `plugin` 배열에 추가**

```json
{
  "plugin": [
    "@ex-machina/opencode-anthropic-auth@1.8.0",
    "oh-my-openagent"
  ]
}
```

> ⚠️ **버전 핀 필수.** 메인테이너가 명시적으로 강조: *"플러그인 버전을 핀하지 않으면 OpenCode가 시작 시마다 자동 업데이트해서 악성 업데이트에 노출될 수 있습니다. 모든 OpenCode 플러그인에 동일하게 적용되는 보안 권고입니다."* 위 예시의 `@1.8.0`은 작성 시점 최신. [Releases](https://github.com/ex-machina-co/opencode-anthropic-auth/releases)에서 최신 버전 확인 후 적용.

**2. OpenCode 재시작 후 `/connect` → `Anthropic` 선택**

플러그인이 활성화되면 인증 옵션이 3가지로 늘어납니다:

```text
┌ Select auth method
│
│ ● Claude Pro/Max               ← 구독 OAuth (이 옵션이 추가됨)
│   Create an API Key            ← console.anthropic.com에서 자동 발급
│   Manually enter API Key
└
```

**3. `Claude Pro/Max` 선택** → 브라우저가 `claude.ai`로 열리며 PKCE OAuth 흐름 진행 → 권한 승인

**4. 동작 원리** (저장소 README 발췌)

플러그인은 내부적으로 다음을 수행합니다:
- PKCE OAuth 토큰 발급·자동 갱신
- 요청 헤더에 OAuth 토큰과 beta 플래그 주입
- **시스템 프롬프트를 Claude Code 호환 형식으로 자동 재작성** (Anthropic 서버가 클라이언트 식별을 위해 요구)
- 모델 비용을 0으로 처리 (구독에 포함이므로)

**5. 문제 발생 시**

저장소 README 권고:

```bash
rm -rf ~/.cache/opencode
# 그리고 opencode.json의 플러그인 버전이 최신인지 확인
```

이 단계 후에도 안 되면 [GitHub Issues](https://github.com/ex-machina-co/opencode-anthropic-auth/issues)에 보고.

> 💡 **omo와 함께 쓸 때의 권고**
>
> 플러그인 메인테이너가 *"oh-my-openagent are known to trigger bans"* 라고 명시한 이유는 omo의 `ultrawork`·Ralph Loop 같은 **장시간·고빈도 자동 실행** 패턴 때문입니다. 만약 두 플러그인을 함께 쓴다면:
> - `ultrawork` 같은 무한 루프 명령 자제
> - 한 작업당 호출 횟수 자체 제한
> - **언제든 계정 잃을 수 있다는 전제로** 백업 계정·결제 수단 준비
>
> 안전하게 omo 풀스택 기능을 쓰려면 옵션 1(ChatGPT) 또는 4-4-A(Anthropic 정식 API 키)를 사용하세요.

#### 4-5. 옵션 4 — 환경 변수 직접 설정 (고급)

GUI 흐름 없이 셸 환경 변수로 직접 인증하고 싶다면 `~/.zshrc` 또는 `~/.bashrc`에 추가:

```bash
# Anthropic
export ANTHROPIC_API_KEY="sk-ant-..."

# OpenAI (API 키 방식, ChatGPT 구독 OAuth와는 별개)
export OPENAI_API_KEY="sk-..."
```

```bash
source ~/.zshrc
```

> ⚠️ ChatGPT 구독자는 이 방식이 **아닙니다.** 옵션 1을 사용하세요. `OPENAI_API_KEY`는 platform.openai.com에서 별도 발급받은 종량제 API 키입니다.
> 마찬가지로 `ANTHROPIC_API_KEY`도 console.anthropic.com에서 발급한 **정식 API 키**(옵션 3 경로 1)입니다. Pro/Max 구독 OAuth(경로 2)와는 다릅니다.

---

### Step 5. Obsidian 설치 + 첫 Vault 생성 (5분)

#### 5-1. 다운로드

[obsidian.md/download](https://obsidian.md/download) → 본인 OS에 맞는 버전 설치.

#### 5-2. 새 Vault 만들기

1. Obsidian 실행
2. **"Create new vault"** 클릭
3. 이름: `AI-Knowledge` (또는 원하는 이름)
4. 위치 권장:
   - **macOS**: `~/Documents/AI-Knowledge`
   - **Windows**: `C:\Users\사용자명\Documents\AI-Knowledge`
5. **"Create"** 클릭

> 💡 **위치 선택 팁**: iCloud, Google Drive 같은 클라우드 동기화 폴더는 **피하세요** — 동기화 충돌로 파일이 손상될 수 있습니다. 로컬 폴더를 권장합니다.

---

### Step 6. ⭐ 통합 테스트 — OpenCode가 Vault에 파일 만들기 (5분)

이제 모든 도구가 연결됐는지 **실제 작업으로** 검증합니다.

#### 6-1. Vault 폴더로 이동

```bash
cd ~/Documents/AI-Knowledge  # 본인 Vault 경로
opencode
```

#### 6-2. Sisyphus에게 첫 명령

OpenCode TUI에서:

```
이 폴더에 "Hello AI Knowledge Management"라는 내용의
welcome.md 파일을 만들어줘.

frontmatter에 created 날짜와 tags: [welcome] 도 넣어줘.
```

#### 6-3. 결과 확인

1. Obsidian으로 돌아가기
2. 좌측 사이드바에서 Vault 새로고침 (Cmd/Ctrl + R)
3. `welcome.md` 파일이 생성됐는지 확인
4. 파일 내용 확인:

기대 결과 예시:
```markdown
---
created: 2026-05-04
tags: [welcome]
---

# Hello AI Knowledge Management
```

✅ 파일이 생성되고 Obsidian에서 열린다면 — **모든 환경 구축 완료!**

---

## 🪄 보너스 — `ultrawork` 한 단어의 위력

omo의 진짜 강력함은 다음 한 단어에 있습니다.

> **`ultrawork`** (또는 줄여서 **`ulw`**)

OpenCode TUI에서 작업 지시 끝에 이 단어 하나만 붙이면, omo가 **모든 에이전트를 동원해 끝날 때까지 멈추지 않고** 작업을 수행합니다. 우리가 다음 챕터부터 만들 PKM 워크플로우에서도 핵심 명령어로 등장합니다.

### 어떤 일이 벌어지나?

| 단계 | 동작 |
|------|------|
| 1. **IntentGate** | 사용자의 진짜 의도를 먼저 분석 (분류 → 행동 순서) |
| 2. **Sisyphus 오케스트레이션** | 작업을 잘게 쪼개 적절한 카테고리·서브에이전트에 위임 |
| 3. **Background Agents** | Explore·Librarian·Oracle 등을 **병렬로** 실행 |
| 4. **Todo Enforcer** | 에이전트가 멈추거나 딴짓하면 시스템이 다시 끌어옴 |
| 5. **Hash-Anchored Edit** | 모든 파일 편집을 콘텐츠 해시로 검증 — 망가진 편집 0건 |

### 카테고리 자동 라우팅

`ultrawork` 안에서 Sisyphus는 모델을 직접 고르지 않고 **카테고리**만 고릅니다. 카테고리는 자동으로 적합한 모델로 매핑됩니다:

| 카테고리 | 용도 | PKM에서의 예 |
|---------|------|------------|
| `quick` | 단일 파일 변경, 오타 수정 | 노트 한 곳 제목 변경 |
| `visual-engineering` | 프론트엔드·UI·디자인 | (이 강의에선 거의 안 씀) |
| `deep` | 자율 리서치 + 실행 | Inbox 자동 분류, 태그 추천 |
| `ultrabrain` | 어려운 로직·아키텍처 결정 | Vault 폴더 구조 설계 |
| `writing` | 문서·산문·기술 글 | 노트 요약, 컨셉 노트 작성 |

> 💡 **다음 챕터 맛보기**
> 03챕터에서 *"AI로 Vault 구조 만들기"* 를 진행할 때 우리가 실제로 입력할 명령은 다음과 같은 형태가 됩니다:
>
> ```
> 내 관심사가 [Kubernetes, AI, DevOps]일 때
> PARA 방법론 기반으로 Obsidian Vault 폴더 구조를
> 설계하고 템플릿 파일까지 만들어줘.
> ultrawork
> ```
>
> 끝의 `ultrawork` 하나가 단순 답변과 **"끝날 때까지 자율 실행"** 을 가르는 스위치입니다.

지금은 단어를 기억만 해두세요. 다음 챕터부터 실제로 써봅니다.

---

## ✅ 완료 체크

- [ ] `opencode --version` 정상 출력
- [ ] `bunx oh-my-opencode doctor` 모든 항목 ✅
- [ ] `/agents`에 Sisyphus·Hephaestus·Prometheus·Oracle·Librarian·Explore 표시됨
- [ ] API 키로 첫 대화 성공
- [ ] Obsidian Vault 생성됨
- [ ] OpenCode가 Vault에 `welcome.md` 직접 생성함

**6개 모두 ✅** 면 다음 챕터로 진행 가능.

---

## 💡 Aha Moment

> **"AI가 진짜로 내 컴퓨터에서 파일을 만든다."**

ChatGPT는 *"이 파일을 만드세요"* 라고 안내만 했습니다.
**OpenCode는 직접 파일을 만듭니다.** 이게 CLI 에이전트의 차별점입니다 — *대화*가 아니라 *실행*.

이제 여러분의 PKM은 **"AI에게 부탁하면 알아서 정리되는"** 시스템으로 진화할 준비가 됐습니다.

---

## 🚧 트러블슈팅

### 문제 1. `opencode` 명령어를 찾을 수 없음

```bash
# PATH 확인
echo $PATH

# npm global 경로 확인
npm config get prefix

# PATH에 추가 필요시 (~/.zshrc)
export PATH="$(npm config get prefix)/bin:$PATH"
source ~/.zshrc
```

### 문제 2. 인증 실패 (옵션 1 — ChatGPT OAuth)

브라우저가 안 열리거나, 로그인 후 터미널로 안 돌아오는 경우:

```bash
# OpenCode 재시작 후 재시도
opencode
/connect
# → OpenAI → ChatGPT Plus/Pro
```

여전히 안 되면:
- 브라우저에서 [chatgpt.com](https://chatgpt.com)에 먼저 로그인된 상태인지 확인
- 시스템 기본 브라우저에서 팝업/리디렉트 차단을 풀기
- 회사·학교 네트워크의 경우 OAuth 콜백(`localhost`)이 막혀 있을 수 있음 → 개인 네트워크에서 재시도
- 재시도 후에도 실패 시 옵션 4(`OPENAI_API_KEY` 환경변수)로 우회 가능 (ChatGPT 구독과 별개의 API 키 필요)

### 문제 2-1. API 키 인증 실패 (옵션 2~4)

```bash
# 환경 변수 확인
echo $ANTHROPIC_API_KEY

# 또는 OpenCode에서 재설정
opencode
/connect
```

다시 안 되면:
- API 키 콘솔에서 키 만료 여부 확인
- 키 앞뒤 공백 제거 후 재입력

### 문제 3. Obsidian에서 파일이 안 보임

- Vault 경로가 OpenCode가 작업한 경로와 같은지 확인
- Obsidian 새로고침 (Cmd/Ctrl + R)
- 숨김 파일 표시 설정 확인 (Settings → Files)

### 문제 4. Sisyphus 에이전트가 안 보임

먼저 진단 도구로 어디가 막혔는지 확인하세요.

```bash
bunx oh-my-opencode doctor
```

흔한 원인:

1. **`opencode.json`의 `plugin` 배열에 항목이 없음**
   - `~/.config/opencode/opencode.json`(또는 `opencode.jsonc`)을 열고 `plugin` 배열에 `"oh-my-openagent"`(또는 호환 이름 `"oh-my-opencode"`)가 있는지 확인.
2. **OpenCode 재시작 안 함**
   - 플러그인은 OpenCode 시작 시점에 로드됩니다. TUI 종료 후 다시 `opencode` 실행.
3. **설치 자체가 실패**
   - Step 3-1로 돌아가 설치 가이드 URL을 에이전트에게 다시 던지세요. 가이드: <https://github.com/code-yeongyu/oh-my-openagent/blob/dev/docs/guide/installation.md>

### 문제 5. 옛 이름 / 새 이름 헷갈림

- 프로젝트명: **`oh-my-openagent`** (새, GitHub repo·문서)
- npm 패키지·CLI·플러그인 설정 파일: **`oh-my-opencode`** (옛, 호환성 유지)
- `opencode.json` 안의 plugin 항목: 둘 다 인식 (`oh-my-openagent` 우선, `oh-my-opencode`는 경고와 함께 동작)

전환기라 두 이름이 혼재하는 게 정상입니다. 명령어를 입력할 때는 위 매핑을 그대로 따르면 됩니다.

---

## ▶ 다음 단계

환경 구축이 완료되었습니다. 이제 **어떤 방법론으로** 정리할지 결정할 차례입니다.

**다음 챕터**: [02. 지식관리 방법론 소개]({{< ref "02-methodology" >}}) — PARA vs 제텔카스텐, 본인에게 맞는 방법론 선택
