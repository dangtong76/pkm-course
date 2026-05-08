# Opencode + Obsidian AI 지식관리 강의

> CLI 기반 AI 에이전트(Opencode + Oh-My-OpenCode)와 Obsidian을 결합하여 **AI 기반 지식관리 시스템**을 구축하는 실습 중심 강의입니다.

## 소스코드 안내

이 저장소는 [Hugo](https://gohugo.io/) 정적 사이트 생성기와 [Lotus Docs](https://github.com/colinwilson/lotusdocs) 테마를 사용한 강의 교안 사이트입니다.

```
opencode-course/
├── config.toml            # Hugo 사이트 설정
├── content/docs/          # 강의 콘텐츠 (마크다운)
├── assets/                # SCSS, JS, 이미지 등 정적 자산
├── i18n/                  # 다국어 지원 (ko, en, de, fr, pt)
├── theme.toml             # Lotus Docs 테마 설정
├── go.sum                 # Hugo 모듈 의존성
└── README.md              # 이 파일
```

## 강의 개요

| 세션 | 제목 | 시간 | 핵심 내용 |
|------|------|------|-----------|
| **01** | 오리엔테이션 & 환경 설정 | 1시간 | CLI AI 에이전트 소개, Opencode + Oh-My-OpenCode + Obsidian 설치 및 연동 |
| **02** | 지식관리 방법론 소개 | 1시간 | PARA vs 제텔카스텐 비교, 본인에게 맞는 방법론 선택 |
| **03** | AI로 Vault 구조 만들기 | 1시간 15분 | AI를 활용한 폴더 구조 자동 생성, 방법론별 템플릿 작성 |
| **04** | MCP로 Obsidian 연동 | 1시간 | MCP(Model Context Protocol) 개념, Obsidian MCP 서버 설정 |
| **05** | 지식 캡처 자동화 | 1시간 | Inbox 자동 분류, 태그/링크 추천, 웹/PDF 자료 자동 정리 |

## 주요 도구

- **[Opencode](https://github.com/anomalyco/opencode)** — 오픈소스 AI 코딩 에이전트 (TUI, 다양한 LLM 지원, MCP 연동)
- **[Oh-My-OpenCode](https://github.com/code-yeongyu/oh-my-opencode)** — Opencode 플러그인. Sisyphus 멀티 에이전트 시스템 (Oracle, Explore, Librarian 등)
- **[Obsidian](https://obsidian.md)** — 마크다운 기반 로컬 지식관리 도구 (양방향 링크, 그래프 뷰)

## 사전 요구사항

- 모던 터미널 (iTerm2, Warp, Windows Terminal + WSL 등)
- LLM API 키 (Anthropic, OpenAI, 또는 OpenCode Zen)
- Obsidian 설치

## 로컬 실행

```bash
# Hugo extended 0.140.0+ 필요
hugo server
```

## 라이선스

강의 교안 콘텐츠입니다. 사이트 테마는 Lotus Docs (MIT License)를 사용합니다.
