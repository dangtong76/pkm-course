# ChatMock Proxy — Quick Start

ChatGPT Plus/Pro/Max 구독을 OpenAI 호환 `/v1` endpoint로 노출하는 LLM proxy. Obsidian Web Clipper의 *Custom Provider*용. 전체 가이드는 [PKM 강의 06장]({{< ref "06-web-clipper" >}}) 참조.

## 사전 요구사항

| 항목 | 요구사항 |
|---|---|
| Docker | Desktop / Colima / OrbStack / Docker Engine + WSL2 (compose v2) |
| ChatGPT 구독 | Plus / Pro / Max (1장에서 OAuth 로그인 완료한 계정) |
| 포트 | 8000 (변경 시 `.env`의 `PORT` 수정) |

## Quick Start

```bash
cp .env.example .env                                          # 1. 환경 변수 (선택)
docker compose run --rm --service-ports chatmock-login        # 2. chatmock login (OAuth, 브라우저 1회)
docker compose up -d                                          # 3. 서버 기동
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-5.4","messages":[{"role":"user","content":"hello"}]}'  # 4. 동작 확인
```

## 검증 환경

| 항목 | 값 |
|---|---|
| 동작 확인 일자 | 2026-05-11 |
| ChatMock image | `storagetime/chatmock:latest` (sha256:757c65c8…) |
| 권장 모델 | `gpt-5.4` |

## ⚠️ 주의

OpenAI ToS 회색지대. 위험·대안(Anthropic native, Ollama)은 [06장 Step 1]({{< ref "06-web-clipper#step-1" >}}) 참조.
