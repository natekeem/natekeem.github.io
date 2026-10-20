---
title: "Claude Code 토큰 절약 총정리, 기본기부터 오픈소스까지"
categories: [AI & Tech, Claude]
tags:
  - AI
  - Dev Log
  - Web Dev
description: "Claude Code의 토큰 폭증 원인부터 내장 명령어(/compact, /clear, Prompt Caching), 그리고 RTK, Context-Mode, Headroom, Caveman 등 실전 오픈소스 도구들의 팩트와 한계를 총정리했다."
date: 2026-10-20 09:00:00 +0900
published: true
image:
  path: /assets/img/posts/2026-10-20-claude-code-token-optimization-tools/cover.jpg
  alt: "RTK 공식 웹사이트와 CLI 출력 가로채기 기술 인터페이스"
---

Claude Code나 터미널 기반 에이전트 CLI를 켜두고 대규모 리팩토링이나 디버깅을 몇 턴 돌리다 보면, 터미널 구석의 토큰 카운터가 무섭게 올라간다. 

터미널 기반 코딩 에이전트는 파일 읽기, 테스트 실행, git diff 조회 등 도구(Tool)를 루프로 연쇄 실행한다. 한 턴이 돌 때마다 전체 파일 내용과 이전 실행 로그가 누적 컨텍스트로 다시 입력되기 때문에, 복잡한 태스크 하나를 끝내면 순식간에 수십만 토큰이 증발하고 월 청구서나 사용량 한도에 빨간불이 켜진다.

이 문제를 풀겠다며 깃허브에는 **RTK(Rust Token Killer)**, **Context-Mode**, **Headroom**, **Caveman**, **Ponytail**, **codebase-memory-mcp** 등 수많은 오픈소스 도구가 쏟아져 나왔다. 프로젝트 README마다 "토큰 70~90% 절감", "터미널 출력 98% 압축" 같은 자극적인 숫자를 내걸고 있다.

과연 이 도구들은 정말로 전체 작업 비용을 줄여줄까? 아니면 Anthropic이 기본 제공하는 내장 기능만 잘 써도 충분할까? 공식 기술 문서와 실제 오픈소스 아키텍처를 바탕으로 팩트체크와 추천 구성을 정리했다.

---

## 1. 왜 Claude Code는 토큰을 무섭게 먹는가?

토큰 다이어트를 하려면 에이전트가 토큰을 어디서 태우는지부터 알아야 한다. Claude Code의 토큰 소모는 일반 챗봇 대화와 완전히 다른 구조를 가진다.

```text
[초기 베이스라인 오버헤드]
  ├─ 시스템 프롬프트 (에이전트 행동 지침)
  ├─ 도구 정의 (Bash, Read, Edit 등 JSON 스키마)
  ├─ CLAUDE.md (프로젝트 규칙 및 빌드 명령어)
  └─ MCP 도구 정의 (연결된 서버당 수천 토큰)
          │
          ▼ (매 턴마다 누적 재전송: Re-read Tax)
[작업 턴 1] ──> [파일 읽기 40KB] ──> [npm test 출력 80KB]
          │
          ▼
[작업 턴 2] ──> [이전 턴의 120KB 로그를 통째로 다시 입력받아 추론]
```

핵심은 **'재읽기 세금(Re-read Tax)'**이다. 
에이전트가 턴 1에서 `npm test`를 돌려 5,000토큰짜리 로그를 출력했다면, 턴 2에서는 그 5,000토큰이 고스란히 입력 컨텍스트에 포함된다. 턴 10까지 가면 과거의 모든 터미널 출력과 파일 내용이 매 턴마다 중복 과금된다. 

결과적으로 전체 작업에서 **입력 토큰(Input Tokens)이 85~90% 이상을 차지**하며, 출력 토큰(Output Tokens)은 10~15%에 불과하다. 즉, 모델의 답변 길이를 줄이는 것보다 **컨텍스트에 들어가는 잡음과 반복 입력을 차단하는 것**이 토큰 절약의 본체다.

---

## 2. 설치 없이 당장 쓰는 공식 기본기 5가지

외부 프로그램을 하나도 깔지 않아도, Claude Code에 내장된 기본 기능만 정확히 이해하면 토큰 낭비의 70% 이상을 잡을 수 있다.

### 1) `/context`로 컨텍스트 윈도우 감사
현재 세션의 컨텍스트가 무엇으로 채워져 있는지 시각적으로 확인하는 명령이다. 시스템 프롬프트, 도구 스키마, 대화 히스토리의 비중을 확인하여 언제 비워야 할지 객관적인 판단을 내릴 수 있다.

### 2) `/compact [지시문]`의 현명한 활용
컨텍스트가 차올랐을 때 이전 대화를 요약 압축해 공간을 확보한다. 
단, 주의할 점이 있다. **`/compact` 자체도 LLM 추론을 거치므로 공짜가 아니다.** 수만 토큰의 과거 히스토리를 읽고 요약문을 작성하는 입력 비용이 발생한다. 따라서 무작정 자주 치는 것보다, `/compact 프론트엔드 작업 로그는 버리고 백엔드 API 변경점과 미해결 버그만 남겨줘`처럼 구체적인 지시문을 붙여 한 번에 확실한 요약을 얻는 것이 효율적이다.

### 3) `/clear` (또는 `/reset`): 비용 0원의 완전 초기화
태스크가 끝났거나 디버깅 방향이 꼬였을 때는 주저 없이 `/clear`를 쳐야 한다. 대화 히스토리를 완전히 날리므로 **토큰 비용이 0원**이다. 디스크에 저장된 `CLAUDE.md`와 `MEMORY.md`는 그대로 살아있으므로, 새로운 작업은 항상 깨끗한 백지 상태에서 시작하는 것이 비용 누수를 막는 가장 강력한 습관이다.

### 4) `/cost` 실시간 지출 모니터링
현재 세션에서 소모된 누적 토큰 수와 달러 환산 비용을 즉시 보여준다. 무거운 작업을 돌리기 전후에 호출해 비용 누수 지점을 파악할 수 있다.

### 5) 셸 명령어의 자체 필터링 (`--quiet`, 파이프)
에이전트에게 `git status`나 테스트를 시킬 때, 프롬프트로 침묵 옵션을 권장하는 것만으로 수천 토큰을 아낀다:
```bash
# 나쁜 예: 수백 줄의 정상 통과 로그가 컨텍스트로 직행
npm test

# 좋은 예: 실패 로그만 남기고 정상 로그는 침묵
npm test -- --silent
git status --short
```

---

## 3. 프롬프트 캐싱(Prompt Caching)과 CLAUDE.md의 상관관계

Anthropic의 **프롬프트 캐싱(Prompt Caching)**은 Claude Code의 경제성을 지탱하는 핵심 기술이다. 캐시 읽기(Cache Read)가 발생하면 **표준 입력 토큰 가격의 90%를 할인(0.1x 과금)**받는다.

하지만 프롬프트 캐싱에는 치명적인 제약이 있다. 바로 **엄격한 접두사 일치(Strict Prefix-matching)**다.

```text
[프롬프트 캐시 체인]
  [시스템 프롬프트] ➔ [도구 스키마] ➔ [CLAUDE.md] ➔ [대화 히스토리...]
         ▲
         └─ 이 중간 어디든 1글자라도 바뀌면 이후의 캐시가 전면 무효화됨!
```

### 세션 중간에 `CLAUDE.md`를 고치면 생기는 참사
`CLAUDE.md`는 시스템 프롬프트 바로 뒤, 컨텍스트의 최전방에 위치한다. 세션 진행 도중 `CLAUDE.md`를 수정하면 이전까지 쌓여있던 프롬프트 캐시가 전부 깨진다(Cache Invalidation). 다음 턴부터는 90% 할인 대신 정가로 캐시 쓰기(Cache Write) 비용을 다시 물어야 한다. `CLAUDE.md`는 세션 시작 전에 확정하고, 길이는 150~200줄 이내로 핵심만 담백하게 유지해야 한다.

### MCP 서버 스키마 세금
연결된 MCP(Model Context Protocol) 서버가 많을수록, 각 서버의 도구 정의와 JSON Schema가 최전방 컨텍스트에 추가된다. 10개 이상의 MCP 서버를 연결하면 아무 작업도 하지 않고 `안녕` 한마디만 보내도 15,000~30,000토큰이 기본 소모된다. 쓰지 않는 MCP 서버는 과감히 비활성화해야 한다.

---

## 4. 오픈소스 도구 선별 분석: 어디를 어떻게 깎는가?

공식 기능 외에 깃허브 오픈소스 생태계에서 검증된 도구들은 에이전트 파이프라인의 서로 다른 계층에 개입한다.

![Claude Code 토큰 최적화 4단계 개입 아키텍처](/assets/img/posts/2026-10-20-claude-code-token-optimization-tools/01-token-optimization-architecture.png)

### 1) CLI 출력 프록시: RTK (Rust Token Killer)
- **저장소**: [`rtk-ai/rtk`](https://github.com/rtk-ai/rtk)
- **개입 위치**: CLI 명령어의 `stdout/stderr` (도구 실행 결과 계층)
- **작동 원리**: 러스트로 작성된 단일 바이너리 프록시다. Claude Code의 `PreToolUse` 훅에 등록되어, 에이전트가 실행하는 `git status`, `npm test`, `cargo build` 등의 출력을 가로챈다. 성공한 수백 개의 테스트 로그와 쓸데없는 힌트 문구를 걷어내고 실패한 스택트레이스만 압축해 LLM 컨텍스트로 넘긴다.

```bash
# OS별 공식 설치 방법
brew install rtk                  # macOS / Linux (Homebrew)
winget install rtk-ai.rtk         # Windows (Winget)

# Cargo로 직접 빌드할 경우 (주의: crates.io에 동명의 타 패키지가 있으므로 반드시 Git 지정)
cargo install --git https://github.com/rtk-ai/rtk

# Claude Code 전역 훅 등록 (현재 프로젝트 디렉토리만 적용할 때는 'rtk init')
rtk init -g                       # rtk init --global과 동일

# 세션별 절감 통계 확인
rtk gain
```

- **수치와 실질 효과의 구분**: RTK 공식 대시보드가 보고하는 60~85% 절감은 **터미널 stdout/stderr 출력 버퍼(바이트/4 환산치)의 압축률**이다. 빌드와 단위 테스트를 빈번하게 반복하는 리팩토링 세션에서는 터미널 재읽기 세금을 크게 줄여주지만, 파일 읽기나 텍스트 작성이 주를 이루는 일반 세션에서는 **프로젝트 및 커뮤니티 추산치 기준** 전체 청구서 절감 폭이 15~35% 수준에 머문다(공인 독립 실측치가 아니며 명령어 구성과 작업 패턴에 따라 편차가 큼).

### 2) 샌드박스 연산 & 세션 연속성: Context-Mode
- **저장소**: [`mksglu/context-mode`](https://github.com/mksglu/context-mode)
- **개입 위치**: 파일 시스템 탐색 및 장기 기억 계층 (MCP 서버)
- **작동 원리**: "코드로 생각하라(Thinking in Code)"는 철학을 구현했다. 에이전트가 40개의 대형 소스 파일을 일일이 읽어 컨텍스트를 채우는 대신, 샌드박스 툴(`ctx_execute`)을 통해 하위 격리 프로세스에서 미니 파싱 스크립트를 실행한다. 대용량 파일은 샌드박스 내부에서만 돌고, 에이전트에게는 최종 요약 결과 2줄만 반환된다.

```bash
# Claude Code 공식 플러그인 마켓플레이스 설치 (Claude Code v1.0.33+)
/plugin marketplace add mksglu/context-mode
/plugin install context-mode@context-mode

# 환경 진단 및 FTS5/훅 정상 등록 확인
/context-mode:ctx-doctor
```

```json
// Claude Code 모델이 내부적으로 호출하는 MCP Tool Call 구조 (CLI 터미널 명령이 아님)
{
  "tool": "ctx_execute",
  "arguments": {
    "command": "python scripts/analyze_ast.py src/",
    "timeout": 30
  }
}
```

- **수치와 실질 효과의 구분**: Context-Mode 데모에서 언급되는 "98% 컨텍스트 차단"은 **대용량 파일 덤프를 2줄 스크립트 출력으로 대체한 프로젝트 자체 시연 케이스**의 수치다. 거대 로그나 다중 모듈 탐색 시에는 컨텍스트 폭증을 막는 탁월한 방어막이 되지만, 파싱 스크립트 실행이 실패하거나 환경 종속성으로 인해 추가 디버깅 턴이 발생하면 역효과가 날 수 있으므로 독립적인 검증과 작업 성격 구분이 필요하다. 또한 세션 히스토리를 SQLite FTS5(BM25)에 색인하여, `/compact` 이후에도 과거 수정 맥락을 정확히 찾아온다.

### 3) 가역적 다중 모달 압축: Headroom
- **저장소**: [`headroomlabs-ai/headroom`](https://github.com/headroomlabs-ai/headroom)
- **개입 위치**: LLM API 호출 프록시 계층
- **작동 원리**: `headroom wrap claude` 형태로 CLI를 감싸거나 로컬 프록시로 동작한다. JSON 데이터는 `SmartCrusher`, 소스코드는 AST 기반 `CodeCompressor`, 로그/텍스트는 로컬 소형 ONNX 모델(`Kompress-base`)로 분기 처리한다. 중요한 점은 **가역적 압축(CCR)**을 지원해, 모델이 원본 스택트레이스를 요구하면 로컬 저장소에서 원문을 복원해준다.

### 4) 발화 스타일 & 코드 최소화: Caveman & Ponytail
- **Caveman ([`JuliusBrussee/caveman`](https://github.com/JuliusBrussee/caveman))**: "원시인처럼 말하라." 인사말, 변명조, 장황한 부연 설명을 시스템 프롬프트 차원에서 차단하고 단답형 기술 팩트만 출력하게 만든다. 출력 텍스트를 65~75% 깎아낸다.
- **Ponytail ([`DietrichGebert/ponytail`](https://github.com/DietrichGebert/ponytail))**: "게으른 시니어 개발자처럼 코딩하라." YAGNI 원칙을 강제해 불필요한 래퍼 함수나 과도한 디자인 패턴 생성을 막아 생성 코드량을 약 15% 줄인다.

### 5) 대규모 코드베이스 탐색: codebase-memory-mcp & ast-grep
- **codebase-memory-mcp ([`DeusData/codebase-memory-mcp`](https://github.com/DeusData/codebase-memory-mcp))**: Tree-sitter를 이용해 160여 개 언어의 코드베이스를 로컬 SQLite 지식 그래프로 색인한다. 함수 호출 관계, 라우트, 클래스 상속을 서브밀리초 그래프 쿼리로 찾아내므로, 맹목적인 `cat`이나 `grep`으로 인한 컨텍스트 낭비를 원천 차단한다.
- **ast-grep ([`ast-grep/ast-grep`](https://github.com/ast-grep/ast-grep))**: 텍스트 정규식 대신 AST 구문 구조로 코드를 검색/치환하여 필요한 노드만 정밀 추출한다.

---

## 5. 광고된 90% 절감과 실제 작업 비용의 괴리

도구 홍보 문구의 "90% 토큰 절약"을 그대로 믿으면 안 된다. 측정 대상 영역과 전체 세션 비용 사이에는 큰 간극이 존재한다.

| 도구 / 기법 | 측정 대상 영역 | 보고된 절감 수치 (출처/기준) | 전체 세션 비용 관점의 영향 | 주요 트레이드오프 및 주의점 |
|---|---|---|---|---|
| **Caveman** | 에이전트의 대화형 답변 텍스트 | 출력 토큰 65~75% 축약 (자체 측정) | **전체 비용 영향 미미 (~10% 미만)**: 긴 세션에서 출력 토큰은 전체 비용의 10% 미만 차지 | 말투가 극도로 축약되어 맥락 설명 부족, 협업 가독성 저하 |
| **Ponytail** | 생성되는 코드 라인 수 | 생성 코드 라인 ~15% 감소 (자체 측정) | **제한적**: 코드 생성 분량 절감 효과는 있으나 전체 세션 비용 지배 요인은 아님 | 과도한 축약으로 에러 핸들링 및 유효성 검증 누락 위험 |
| **RTK** | 터미널 stdout/stderr 출력 볼륨 | CLI 출력 바이트 60~85% 압축 (자체 측정 / bytes/4 추정) | **빌드·테스트 위주 세션에서 유의미**: 터미널 재읽기 세금 차단 | 과도한 필터링 시 핵심 에러 원인이 누락되어 추가 디버깅 턴 유발 가능 |
| **Context-Mode** | 대용량 파일 덤프 및 코드베이스 검색 | 데모 시나리오 기준 컨텍스트 최대 98% 차단 (프로젝트 자체 데모) | **대형 파일 분석 작업에서 강력**: 원문 전체를 프롬프트에 넣지 않고 결과만 반환 | 서브프로세스 샌드박스 툴 실행 실패 시 재시도 오버헤드, 환경 종속성 |
| **Prompt Caching** | 반복 입력 접두사 (CLAUDE.md, 시스템 프롬프트) | 캐시 적중 시 입력 토큰 90% 할인 (Anthropic 공식 기준) | **가장 확실하고 일관된 비용 절감원**: 전체 청구서의 대다수를 차지하는 입력 비용 직접 절감 | 세션 도중 `CLAUDE.md` 수정 또는 MCP 서버 변경 시 접두사 불일치로 캐시 전면 파괴 |

### 왜 특정 출력 90% 절감이 전체 90% 비용 절감이 되지 못하는가?
1. **입력 토큰 지배 구조**: 아무리 Caveman으로 출력을 줄여도, 매 턴마다 수만 토큰의 누적 대화 기록이 다시 입력된다면 전체 세션 비용 절감은 한 자릿수에 그친다.
2. **과도한 압축으로 인한 재작업(Rework)**: RTK나 필터가 컴파일러 경고나 미묘한 스택트레이스를 너무 공격적으로 잘라내면, 에이전트가 원인을 파악하지 못해 3~5번 헛돌게 된다. 추가로 발생한 3턴의 비용이 처음에 아낀 토큰보다 훨씬 크다.
3. **캐시 미스 유발**: 외부 프록시가 프롬프트 구조를 동적으로 뒤흔들면 Anthropic의 프롬프트 캐시가 깨져, 90% 할인받던 캐시 읽기가 정가 캐시 쓰기로 둔갑할 수 있다.

---

## 6. 사용자 환경별 최적 추천 조합

모든 도구를 다 깔 필요는 없다. 내 작업 환경과 결제 방식에 맞는 조합 하나만 고르면 된다.

### 👤 1. Claude Pro / Max 구독 플랜 사용자
- **고민**: 토큰당 요금보다는 5시간당 메시지 한도(Rate Limit) 도달이 문제.
- **추천**: **내장 `/clear` + `/compact` + Caveman**
- **이유**: 작업 단위가 바뀔 때마다 `/clear`로 히스토리를 비우고, 긴 작업에서는 Caveman으로 불필요한 잡담 턴을 줄여 한도 도달 시간을 늦춘다.

### 👤 2. API 종량제 (Pay-as-you-go) 개발자
- **고민**: 월말 신용카드 청구서 금액을 직접적으로 줄여야 함.
- **추천**: **Prompt Cache 사수 + RTK 프록시**
- **이유**: `CLAUDE.md`를 150줄 이내로 고정하고 미사용 MCP를 제거해 90% 캐시 할인을 유지한다. 여기에 빌드/테스트가 잦은 프로젝트라면 RTK를 걸어 터미널 재읽기 세금을 방어한다.

### 👤 3. 대규모 코드베이스 (Monorepo) 개발자
- **고민**: 코드베이스가 너무 커서 에이전트가 파일 몇 개만 찾아도 컨텍스트가 폭발함.
- **추천**: **Context-Mode (`ctx_execute`) + codebase-memory-mcp**
- **이유**: 파일을 텍스트로 읽지 않고 Tree-sitter 지식 그래프로 함수와 라우트를 쿼리하며, 무거운 로그는 샌드박스에서 파싱해 요약만 컨텍스트로 받는다.

### 👤 4. 외부 설치가 귀찮은 미니멀리스트
- **추천**: **오직 공식 기능만 사용**
- **이유**: 외부 도구를 하나도 깔지 않아도, `/context` 확인, `/clear` 습관화, 셸 명령어에 `-q` / `--silent` 붙이기만 지켜도 낭비의 75%는 차단된다.

---

## 7. 굳이 설치할 필요가 없는 경우와 주의점

1. **소규모 토이 프로젝트 / 단일 파일 스크립트**:
   컨텍스트 윈도우가 찰 일도 없는 500줄 미만 프로젝트에서는 Context-Mode나 codebase-memory-mcp를 붙이는 설정 작업 자체가 오버헤드다.
2. **호환성이 검증되지 않은 복수 프록시 중복 설치**:
   RTK와 Headroom을 동시에 걸면 훅과 프록시가 서로의 입출력을 이중 가공하면서 파서 에러가 나거나 캐시 접두사가 깨질 위험이 높다.
3. **비공식 래퍼의 보안 검토**:
   모든 터미널 입출력을 가로채는 CLI 도구는 반드시 오픈소스 코드를 직접 확인하고 설치해야 한다. 인증 토큰이나 환경변수가 노출될 위험이 있는 비공개 바이너리는 절대 쓰지 말자.

---

## 8. 핵심 공식 문서 및 출처 링크

- **Anthropic Claude Code 공식 문서**: [Claude Code Overview & Commands](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview)
- **Anthropic Prompt Caching 명세**: [Prompt Caching Guide](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)
- **RTK (Rust Token Killer)**: [github.com/rtk-ai/rtk](https://github.com/rtk-ai/rtk)
- **Context-Mode**: [github.com/mksglu/context-mode](https://github.com/mksglu/context-mode)
- **Headroom**: [github.com/headroomlabs-ai/headroom](https://github.com/headroomlabs-ai/headroom)
- **codebase-memory-mcp**: [github.com/DeusData/codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp)
- **Caveman Skill**: [github.com/JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman)
- **ast-grep**: [github.com/ast-grep/ast-grep](https://github.com/ast-grep/ast-grep)
