---
title: "10. Hermes 에이전트 팀 — OpenCode로 멀티에이전트 셋업"
weight: 10
date: 2026-05-29
draft: false
---

> **⏰ 시간**: 약 60분 (이론 10분 + 실습 50분)
> **🎯 목표**: gpt-5.5 기반 OpenCode로 **5명의 AI 임원 팀(COO·CTO·CDO·CCO·CMO)** 을 Hermes에 셋업하고, kanban으로 협업시켜 *제품 스펙 → Instagram 홍보 콘텐츠*를 자동 산출합니다. SOUL을 손으로 쓰지 않고 **OpenCode가 셋업을 대행**하는 워크플로우를 익힙니다.

---

## 🧭 한 줄 요약

> **"혼자 다 하는 AI"가 아니라, 회사 컨텍스트를 공유하는 *전문가 에이전트 팀*을 OpenCode로 셋업한다. 봇끼리 떠들게 하지 않고, kanban으로 병렬 작업 → 한 명이 종합한다.**

---

## 📚 학습 목표

이 챕터를 마치면 다음을 할 수 있습니다:

- [ ] `hermes profile`로 역할별 에이전트(프로파일)를 만들고 **`--clone`의 중요성**을 설명할 수 있다
- [ ] **회사 페르소나(Company Brief)** 를 공유 프레임으로 잡고 OpenCode로 SOUL을 일괄 생성할 수 있다
- [ ] `hermes kanban`으로 **병렬 워커 → 종합** 태스크 그래프(DAG)를 구성할 수 있다
- [ ] 디스패처가 에이전트를 자동 실행하는 흐름을 이해하고, **크래시 루프를 진단·복구**할 수 있다
- [ ] 워커 워크스페이스의 **휘발성**을 이해하고 산출물을 영속화할 수 있다
- [ ] **PC 재시작 후 Hermes를 복구**할 수 있다 (WSL2)

---

## 🛠️ 사전 요구사항

| 항목 | 요구 사항 |
|---|---|
| **OS** | Windows + WSL2 (Ubuntu) — **Windows Terminal → Ubuntu 탭**에서 작업 |
| **Hermes** | Chapter 09에서 로컬 설치 + 게이트웨이 기동 완료 |
| **모델** | gpt-5.5 (OpenAI Codex / ChatGPT 구독) |
| **OpenCode** | gpt-5.5로 설정 (셋업 대행용) |
| **Slack** | Chapter 09에서 연동 완료 (선택, 결과 전송용) |

> [!info] **이 챕터의 모든 명령은 WSL2 Ubuntu 탭에서 로컬로 실행합니다**
>
> Chapter 09에서 띄운 *로컬 Hermes 게이트웨이* 위에서 진행합니다. 원격 서버·SSH 없이, `hermes ...` 명령을 바로 칩니다. 대시보드는 `http://localhost:9119`로 접속합니다.

---

## §0. 왜 "팀"인가 — 1명 vs 5명 (이론, 10분)

지금까지 Hermes는 *한 명의 만능 비서*였습니다. 하지만 실제 의사결정은 **서로 다른 관점이 충돌·보완**하며 나옵니다. 그래서 이번엔 5명의 **AI 임원 팀**을 만듭니다.

### 이번 챕터의 가상 회사 — BeanPilot

| 항목 | 내용 |
|---|---|
| 정체성 | "AI가 입맛을 학습해 스페셜티 원두를 추천하는 구독 D2C" |
| 파일럿 제품 | **Taste Match** — 입맛 진단 퀴즈 → 첫 박스 추천 온보딩 |
| 산출 목표 | 제품 스펙 1장 + Instagram 홍보 콘텐츠(캡션·해시태그·이미지) |

### 5인 로스터

| 역할 | 관점 / 임무 |
|---|---|
| 💼 **COO** | 시장·타깃·포지셔닝·구독가격·물류 |
| 🔧 **CTO** | 취향 진단/추천 엔진·기술 실현성·리스크 |
| 🎨 **CDO** | 퀴즈 UX·언박싱 경험·디자인 컨셉 |
| 🫘 **CCO** | 산지·로스팅·커핑 노트·취향 프로파일 정의 (커피 전문) |
| 📣 **CMO** | (마케팅) 마케팅·IG 관점 — 5인 워커 중 하나 |
| ⚕ **Hermes** | (**default Agent**) Slack 창구 · 오케스트레이터 · **종합** (5인 결과를 모아 스펙+IG) |

### 협업의 3원칙 (안전 설계)

> [!warning] **봇끼리 Slack에서 자유롭게 대화시키지 마세요**
>
> Hermes 봇은 (1) 서로의 메시지를 이벤트로 받지 못하고(알려진 제약), (2) 봇 핑퐁은 *무한 루프·비용 폭주* 위험이 큽니다. 게다가 **턴 상한(기본 90)은 에이전트별**이라 5봇 = 최대 450턴.
>
> → 정답은 **kanban 오케스트레이션**입니다: 5 워커 병렬 → default Agent 종합. *사람은 **Hermes(default Agent)**(@Hermes)와 대화*하고, Hermes가 kanban으로 팀을 돌리고 결과를 **종합**해 중계합니다. (CMO는 *마케팅 관점 워커* — 종합 담당이 아니며, 워커는 Slack 연결 없음)

```
🏢 BeanPilot (회사 페르소나, 공유)
      │  (모든 임원이 이 위에서 사고)
   ┌──┴───────────────────────┐
  💼COO 🔧CTO 🎨CDO 🫘CCO 📣CMO   ← 5 병렬 워커
      │  ⚕ Hermes(default Agent)이 분배·종합
      ▼
   📄 제품 스펙  →  📣 IG 콘텐츠(캡션·해시태그·이미지)
      ▼
   ✋ 사람 승인 (게시는 직접)
```

> [!info] **이 챕터의 메타 메시지 — "AI로 AI 팀을 셋업한다"**
>
> 핵심은 *SOUL을 손으로 쓰는 것*이 아니라, **OpenCode(gpt-5.5)에게 시켜서** 프로파일·페르소나·kanban을 구성하는 것입니다. 사람은 *무엇을 만들지*만 정하고, OpenCode가 *어떻게 만들지*를 대행합니다.

---

## §1. 환경 준비 — gpt-5.5 정렬 (5분)

팀의 두뇌를 통일합니다.

```bash
# 현재 모델 확인
$ hermes --version

# 게이트웨이 모델을 gpt-5.5로 (OpenCode에게 시켜도 됨)
$ hermes model        # 대화형 → gpt-5.5 선택
# 또는 비대화형:  ~/.hermes/config.yaml 의 model.default 를 gpt-5.5 로

# 게이트웨이 재기동 + 검증
$ systemctl --user restart hermes-gateway
$ hermes gateway status        # active (running) + slack/email connected
```

> [!warning] **`gpt-5.3-codex`를 쓰고 있었다면 반드시 `gpt-5.5`로 전환**
>
> 일부 codex 모델은 ChatGPT 계정 경로에서 *조기 종료(sunset)* 됩니다. 일반 대화·요약·종합 비중이 높은 팀 작업엔 `gpt-5.5`가 균형이 좋습니다.

> 💡 **OpenCode 활용**: *"Hermes 게이트웨이 모델을 gpt-5.5로 바꾸고 재기동해줘"* 라고 OpenCode에 시키면 config 수정·재기동·검증까지 대행합니다. 학생이 yaml을 손대지 않습니다.

#### ✅ 검증

- [ ] `hermes --version` 출력
- [ ] `hermes gateway status` → `active (running)`
- [ ] `hermes doctor` → `Provider: openai-codex (auth OK)`, `Model: gpt-5.5`

---

## §2. 프로파일 5개 생성 — ⚠️ `--clone` 함정 (10분)

각 임원 = **프로파일**(자기 SOUL·home·skills·sessions 격리). 한 줄 역할 설명(`describe`)은 *kanban이 작업을 라우팅*할 때 씁니다.

```bash
$ for r in coo cto cdo cco cmo; do
    hermes profile create $r --clone        # ⚠️ --clone 필수! (아래 박스)
  done

$ hermes profile describe coo --text "COO: 시장, 타깃, 포지셔닝, 구독가격, 물류"
$ hermes profile describe cto --text "CTO: 취향진단/추천엔진 실현성, 리스크"
$ hermes profile describe cdo --text "CDO: 퀴즈 UX, 언박싱 경험, 디자인 컨셉"
$ hermes profile describe cco --text "CCO: 산지, 로스팅, 커핑노트, 취향 프로파일 정의"
$ hermes profile describe cmo --text "CMO: 종합(synthesizer), 제품 스펙, IG 콘텐츠"

$ hermes profile list
```

> [!danger] **가장 흔한 함정 — `--clone` 없이 만들면 워커가 24번 크래시한다**
>
> `hermes profile create <name>` 을 **그냥** 실행하면 *빈 프로파일*(config·인증 없음)이 만들어집니다. 나중에 kanban 워커가 그 프로파일로 실행될 때:
>
> ```
> It looks like Hermes isn't configured yet -- no API keys or providers found.
> Agent crashed 24x: pid ... exited with code 1
> ```
>
> 디스패처가 계속 재시도하며 크래시 루프에 빠집니다.
>
> **해결**: 프로파일 생성 시 **`--clone`** 을 붙입니다 — active(default) 프로파일의 `config.yaml`·`.env`·`SOUL.md`를 복사해 *provider·인증을 상속*시킵니다.
>
> | 방법 | 효과 |
> |---|---|
> | `hermes profile create <name> --clone` | config.yaml/.env/SOUL.md 복사 (CLI) |
> | `hermes profile create <name> --clone-all` | 전체 상태 복사 |
> | 대시보드 NEW PROFILE → **"Clone config from default profile"** 체크 | 위와 동일 (UI) |

> [!warning] **이미 `--clone` 없이 만들어 크래시 중이라면 (복구)**
>
> ```bash
> # 1) 크래시 루프 정지
> $ hermes kanban --board <board> block <task-id>
> # 2) default의 config·auth를 각 프로파일에 복사
> $ for r in coo cto cdo cco cmo; do
>     cp ~/.hermes/config.yaml ~/.hermes/profiles/$r/config.yaml
>     cp ~/.hermes/auth.json   ~/.hermes/profiles/$r/auth.json
>   done
> # 3) 검증
> $ hermes --profile cco doctor        # "OpenAI Codex auth (logged in)" 확인
> ```

> 💡 **OpenCode 활용**: *"BeanPilot 임원 5명(coo/cto/cdo/cco/cmo) 프로파일을 default에서 clone해 만들고 각 역할 설명을 등록해줘"* → OpenCode가 `create --clone` + `describe`를 일괄 실행합니다.

#### ✅ 검증

- [ ] `hermes profile list`에 5개 프로파일 표시
- [ ] 각 프로파일 `hermes --profile <r> doctor` → **Codex auth (logged in)**

---

## §3. 회사 페르소나 + SOUL 5개 — OpenCode 생성 (10분)

5명이 *같은 회사*라는 공유 프레임이 없으면 협업이 공중에 뜹니다. **회사 브리프**를 만들고, 각 SOUL에 주입합니다.

### 3-A. 공유 회사 브리프

`~/.hermes/profiles/_shared/BeanPilot.md` (또는 vault) 에 회사 정의를 둡니다:

```markdown
# BeanPilot — Company Brief (공유)
- 정체성: AI가 입맛을 학습해 스페셜티 원두를 추천하는 구독 D2C
- 미션: 커피 초보도 헤매지 않고 자기 취향의 원두를 만나게 한다
- 타깃: 25-39세, 홈카페 입문~중급
- 브랜드 보이스: 친근하지만 전문성 있는 바리스타 친구. 과장 광고 금지, 감각적·따뜻함
- 비주얼 무드: 따뜻한 크림/브라운, 자연광, 원두·드립, 미니멀
- 파일럿 제품: Taste Match — 입맛진단 퀴즈 → 첫 박스 추천 온보딩
```

### 3-B. SOUL은 OpenCode가 생성 (수기 X)

각 `~/.hermes/profiles/<role>/SOUL.md` = **[회사 컨텍스트] + [역할 관점] + [협업 규칙]**. 이걸 직접 타이핑하지 말고 OpenCode에 시킵니다:

> 💬 **OpenCode 프롬프트 예시**
>
> *"위 BeanPilot 회사 브리프를 공유 컨텍스트로, coo/cto/cdo/cco/cmo 5개 프로파일의 SOUL.md를 생성해줘. 각 SOUL은 (1) 회사 한 단락 (2) 역할별 관점·무엇을 끝까지 따지는지 (3) '봇끼리 직접 대화 금지, 합의는 kanban 보드로' 협업 규칙을 포함. CMO엔 IG 산출 규칙(캡션 후킹+본문+CTA, 해시태그 5개 이내, 이미지는 따뜻한 무드·텍스트 없음, 게시 자동 금지)도 넣어줘. Slack 창구는 Hermes 게이트웨이임을 명시 — 워커는 Slack 연결 없음."*

생성된 CMO SOUL.md 예시:

```markdown
# SOUL — BeanPilot CMO (종합 워커)

## 회사
너는 BeanPilot의 CMO다. (AI 취향 추천 원두 구독 D2C / 바리스타 친구 톤 / 과장 금지)

## 역할
COO/CTO/CDO/CCO의 (kanban 선행) 결과를 종합해 제품 스펙과 IG 콘텐츠를 만든다.
(Slack 창구는 Hermes 게이트웨이 — 너는 종합·산출 담당, Slack 연결 없음)

## 협업
선행 작업 결과를 kanban에서 읽어 종합한다.
봇끼리 직접 대화 금지 · 합의는 kanban 보드로.

## 산출 규칙 (IG)
- 캡션: 후킹 1줄 + 본문 3~4줄 + CTA (과장 금지)
- 해시태그: 5개 이내 / 이미지: 따뜻한 크림·브라운, 자연광, 이미지에 텍스트 X
- 게시 자동 금지 — 반드시 사용자 승인 후 직접 게시
```

> 💡 **왜 OpenCode 생성인가**: ① 손으로 안 써서 빠르고 ② 회사 컨텍스트가 5개에 *일관*되게 들어가며 ③ 명령 한 줄로 *재현* 가능. 이게 "AI로 AI 팀 셋업"의 핵심.

#### ✅ 검증

- [ ] `~/.hermes/profiles/<r>/SOUL.md` 5개 모두 회사+역할 반영
- [ ] CMO SOUL에 "봇끼리 대화 금지 / 게시 자동 금지" 포함

---

## §4. kanban 보드 + 태스크 DAG (10분)

협업 엔진은 **kanban**입니다: SQLite 보드, 태스크를 *named 프로파일*이 격리 워크스페이스에서 실행, 의존관계 지원.

```bash
$ hermes kanban init                          # kanban.db 생성
$ hermes kanban boards create beanpilot       # 프로젝트 보드

# 워커 4개 (병렬) — ⚠️ --board 는 'kanban' 바로 뒤 (서브커맨드 앞!)
$ hermes kanban --board beanpilot create "COO 관점: Taste Match 시장/타깃/가격/물류" --assignee coo
$ hermes kanban --board beanpilot create "CTO 관점: 추천엔진 실현성/리스크"           --assignee cto
$ hermes kanban --board beanpilot create "CDO 관점: 퀴즈 UX/언박싱/디자인"            --assignee cdo
$ hermes kanban --board beanpilot create "CCO 관점: 산지/로스팅/커핑/취향 프로파일"     --assignee cco

# 종합 1개 (워커 4개에 의존) — 출력된 task id 를 --parent 로
$ hermes kanban --board beanpilot create "CMO 종합: 제품 스펙 + IG 콘텐츠" \
    --assignee cmo --parent <coo-id> --parent <cto-id> --parent <cdo-id> --parent <cco-id>

$ hermes kanban --board beanpilot list
```

```
▶ t_xxxx ready  coo   ...        ┐
▶ t_xxxx ready  cto   ...        │ 병렬 (디스패처가 자동 실행)
▶ t_xxxx ready  cdo   ...        │
▶ t_xxxx ready  cco   ...        ┘
◻ t_xxxx todo   cmo   ...        ← 4개 의존, 자동 blocked → 4개 done 시 ready
```

> [!info] **함정 — `--board`의 위치 + `--text` 류 플래그**
>
> - `--board <slug>`는 **`hermes kanban` 바로 뒤**에 옵니다 (`create`/`list` *뒤*가 아님).
> - `boards switch beanpilot`로 현재 보드를 바꿔두면 `--board` 생략 가능.

> [!note] **`hermes kanban swarm`은 verifier가 필수**
>
> `kanban swarm "<goal>" --worker ... --verifier ... --synthesizer ...`는 *병렬→검증→종합*을 한 명령으로 만들지만 **검증자 프로파일이 별도로 필요**합니다. 5인 구성엔 위처럼 **수동 DAG**(`create --parent`)가 더 깔끔합니다.

---

## §5. 실행 & 트러블슈팅 (10분)

게이트웨이는 **내장 디스패처**(약 60초 tick)를 호스팅합니다. **`ready` 태스크는 게이트웨이가 떠 있으면 자동 실행**됩니다.

```bash
$ hermes kanban --board beanpilot list     # ready → running → done 추적
$ hermes kanban --board beanpilot show <task-id>   # 진단·런 이력·코멘트
$ hermes kanban --board beanpilot log <task-id>    # 실행 로그
```

> [!warning] **`ready`로 만든 순간 "장전"된다 — 자동 실행 주의**
>
> Step 4에서 태스크를 `ready`로 만들면 디스패처가 *알아서 실행*합니다(비용 발생). 의도적으로 막으려면 생성 시 `--initial-status blocked`, 실행할 때 `unblock`.

### 크래시 루프 진단·복구 (실전)

`show`에서 `Agent crashed Nx: pid ... exited with code 1`이 보이면 — 거의 **§2의 `--clone` 누락**입니다:

```bash
# 1) 멈추기
$ hermes kanban --board beanpilot block <task-id>
# 2) 로그로 원인 확인
$ hermes kanban --board beanpilot log <task-id>
#    → "Hermes isn't configured yet -- no API keys or providers found"
# 3) config·auth 복사 (§2 복구 박스)
$ cp ~/.hermes/config.yaml ~/.hermes/profiles/<r>/config.yaml
$ cp ~/.hermes/auth.json   ~/.hermes/profiles/<r>/auth.json
# 4) 1개만 먼저 재실행해 검증 → OK면 나머지 unblock
$ hermes kanban --board beanpilot unblock <task-id>
```

> [!tip] **비용 절약 — 1개 먼저 시범 실행**
>
> 전체를 한 번에 돌리기 전에 **워커 1명만 unblock**해서 *이번엔 크래시 없이 done 되는지* 확인하세요. crash는 API 호출 *전* setup 단계라 codex 소비는 적지만, 정상 실행은 실제 토큰을 씁니다.

정상 흐름:

```
워커 4개 done → CMO 종합 자동 ready → CMO 실행
  └ 부모 결과 종합 → 제품 스펙 + IG 캡션/해시태그
  └ image_gen 으로 비주얼 생성 → vision 으로 QA (텍스트/로고 없음 확인)
  └ done
```

> 💡 4 프로파일이 **같은 codex 토큰 공유** → 동시 실행 시 ChatGPT 사용량 한도에 걸려 느려질 수 있습니다. 급하면 1명씩 순차 실행.

---

## §6. 산출물 — ⚠️ 워크스페이스는 휘발성 (5분)

> [!danger] **워커 워크스페이스는 작업 완료 후 정리된다 — 산출물이 사라진다**
>
> 워커가 만든 파일은 `~/.hermes/kanban/boards/<board>/workspaces/<task-id>/` 의 *스크래치 공간*에 저장됩니다. 이 공간은 **태스크 완료 후 휘발**됩니다. CMO가 `beanpilot_cmo_final.md`(캡션·스펙)를 거기에만 저장하면 — *나중에 찾으면 없습니다*.
>
> | 산출물 | 생존? |
> |---|---|
> | image_gen 이미지 | ✅ `~/.hermes/profiles/cmo/cache/images/`에 캐시되어 생존 |
> | 텍스트(스펙/캡션) | ❌ 워크스페이스에만 있으면 소실 |

**영속화 3가지 방법** (SOUL에 규칙으로 박아두세요):

1. **vault에 저장**: `$VAULT_ROOT/03-Projects/.../` 등 영속 경로에 직접 write
2. **kanban 코멘트에 본문 포함**: `hermes kanban comment <task-id> --text "<전체 캡션·스펙>"`
3. **Slack로 전송**: 게이트웨이를 통해 (§7)

> [!warning] **워커는 Slack에 직접 못 보낸다**
>
> 워커는 격리 컨텍스트라 *메시징 플랫폼 연결이 없습니다*. CMO SOUL에 "Slack 전송" 규칙을 넣어도 워커 단계에선 *"연결된 채널 없음"* 으로 실패합니다. → Slack 전송은 **게이트웨이 쪽 `hermes send`**(§7)로 합니다.

---

## §7. 운영 — 대시보드 · Slack 전송 (5분)

### 7-A. 대시보드 (localhost)

```bash
$ hermes dashboard            # http://localhost:9119
```

WSL2에서는 Windows 브라우저로 `http://localhost:9119`가 바로 열립니다(로컬 포워딩). 여기서 프로파일·skill·sessions·**Jobs(kanban) 보드**를 볼 수 있습니다.

> [!info] **`--insecure`의 정확한 정의**
>
> `--insecure`는 ① **비-loopback 바인딩 허용**(0.0.0.0 → LAN/원격) ② **OAuth 로그인 게이트 OFF**, 딱 두 가지입니다. **인증을 완전히 끄는 게 아닙니다** — 관리 API는 여전히 *세션 토큰*(`Authorization: Bearer ...`)을 요구하고, 웹 UI는 페이지에 주입된 토큰으로 자동 인증합니다.
>
> - **로컬(localhost) 접속이면 `--insecure` 불필요** — 그냥 `hermes dashboard`.
> - 원격 노출은 *API 키가 네트워크에 드러나는* 위험 → WSL2 로컬에선 권장하지 않음.
> - UI에서 `401 UNAUTHORIZED`가 보이면 → 토큰 만료/캐시 → **하드 새로고침**(Ctrl+Shift+R). 대시보드 재시작 시 토큰이 갱신되어 열린 탭은 401이 됩니다.

### 7-B. Slack로 결과 전송

게이트웨이는 Slack에 연결돼 있으므로 `hermes send`로 보냅니다(LLM·에이전트 불필요):

```bash
$ hermes send --list                          # 사용 가능한 타깃 확인
$ hermes send -t slack:#hermes-office -s "BeanPilot Taste Match" "<캡션·해시태그 텍스트>"
```

> [!note] **이미지 첨부는 별도**
>
> `hermes send`는 *텍스트 전용*입니다. IG 이미지(`~/.hermes/profiles/cmo/cache/images/...png`)를 Slack에 올리려면 Slack 봇 토큰으로 `files.upload`(Slack API)를 쓰거나, 사람이 직접 첨부합니다. **IG 게시 자체는 자동화하지 않고 사람이 승인 후** 올립니다.

---

## 🧯 부록 A. PC 재시작 후 Hermes 복구 (WSL2)

> [!tip] **컴퓨터를 끄고 켜면 WSL2가 종료되어 Hermes도 멈춥니다 — 복구 순서**
>
> WSL2는 Windows 재부팅 시 종료됩니다. 게이트웨이를 systemd user 서비스로 등록했어도, WSL2는 *터미널을 열어야* 시작되는 특성이 있습니다.

**1) WSL2부터 깨우기**
```powershell
# (Windows PowerShell) WSL 상태 확인 / 깨우기
wsl -l -v
```
보통은 **Windows Terminal → Ubuntu 탭을 열면** WSL2가 자동 시작됩니다.

**2) 게이트웨이 상태 확인 → 없으면 시작**
```bash
$ hermes gateway status
# active 면 OK. stopped/없으면:
$ systemctl --user start hermes-gateway
$ hermes gateway status          # active (running) + slack connected 확인
```

**3) 자동 시작이 안 될 때 (systemd·linger 설정)**

WSL2에서 user 서비스가 부팅 시 안 뜨면:
```bash
# (a) WSL2에 systemd 활성화 — /etc/wsl.conf
$ sudo tee -a /etc/wsl.conf > /dev/null <<'EOF'
[boot]
systemd=true
EOF
# PowerShell에서:  wsl --shutdown   후 Ubuntu 탭 재오픈

# (b) 로그아웃해도 서비스 유지 (linger)
$ loginctl enable-linger $USER

# (c) 서비스 enable
$ systemctl --user enable hermes-gateway
```

**4) 게이트웨이가 `deactivating`에 멈춰 있으면 (drain hang)**
```bash
$ systemctl --user kill --signal=SIGKILL hermes-gateway
$ systemctl --user reset-failed hermes-gateway
$ systemctl --user start hermes-gateway
```

**5) 대시보드/워크스페이스는 수동 재시작**

`hermes dashboard`를 `nohup`/포그라운드로 띄웠다면 **재부팅 후 사라집니다**. 다시 실행:
```bash
$ hermes dashboard            # localhost:9119
```

**6) 최종 점검**
```bash
$ hermes doctor               # provider auth OK / gpt-5.5 / slack connected
$ hermes kanban boards list   # 보드·태스크 보존 확인 (kanban.db는 영속)
```

> [!info] **무엇이 살아남고 무엇이 사라지나**
>
> | 항목 | 재부팅 후 |
> |---|---|
> | `~/.hermes/` (프로파일·SOUL·메모리·kanban.db·skills) | ✅ 영속 |
> | 게이트웨이 서비스 | ⚠️ WSL2 시작 + (linger/enable 시) 자동, 아니면 수동 `start` |
> | `hermes dashboard` (nohup) | ❌ 수동 재시작 |
> | 워커 워크스페이스 스크래치 | ❌ 원래 휘발성 (§6) |

---

## 부록 B. 트러블슈팅 요약

| 증상 | 원인 | 해결 |
|---|---|---|
| 워커 `Agent crashed Nx exited code 1` | `--clone` 누락 (provider/auth 없음) | config.yaml+auth.json 복사 (§2) |
| 태스크가 `ready`에서 안 움직임 | 게이트웨이/디스패처 미동작 | `hermes gateway status` → start |
| 산출물 파일이 안 보임 | 워크스페이스 휘발 | 영속화 (vault/코멘트/Slack) §6 |
| 대시보드 `401` | 세션 토큰 만료/캐시 | 하드 새로고침 (§7) |
| Slack 중계 실패 (워커) | 워커는 메신저 연결 없음 | `hermes send`(게이트웨이) §7 |
| 게이트웨이 `deactivating` 멈춤 | drain hang | SIGKILL + reset-failed + start (부록 A) |

---

## ✅ 최종 체크리스트

- [ ] 게이트웨이 gpt-5.5 + active
- [ ] 프로파일 5개 (`--clone`) + 역할 describe + 각 doctor auth OK
- [ ] 회사 브리프 + SOUL 5개 (OpenCode 생성)
- [ ] kanban 보드 + 워커4→종합 DAG
- [ ] 파일럿 실행 → 제품 스펙 + IG 콘텐츠(캡션·해시태그·이미지) 산출
- [ ] 산출물 영속화 (vault/코멘트/Slack 중 1)
- [ ] PC 재시작 복구 절차 숙지 (부록 A)

---

## 🎓 이 챕터에서 배운 것

- **AI로 AI 팀을 셋업**: SOUL을 손으로 쓰지 않고 OpenCode가 회사 브리프·페르소나·kanban을 대행
- **`--clone`의 중요성**: 프로파일은 기본이 빈 상태 → provider/auth 상속을 명시해야 워커가 산다
- **kanban 오케스트레이션**: 봇끼리 대화 X, 병렬 워커 → 종합 (안전·재현·감사)
- **휘발성 워크스페이스**: 산출물은 반드시 영속화
- **WSL2 운영**: 재부팅 복구 + systemd/linger + 대시보드 재시작

> 다음 단계(선택): CMO가 만든 IG 콘텐츠를 vault에 영속 저장하고, `hermes send`로 Slack 팀 채널에 자동 공유하는 cron을 붙여보세요.
