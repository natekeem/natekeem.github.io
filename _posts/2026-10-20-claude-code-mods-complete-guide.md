---
title: "Claude Mods 완전 가이드, 설치부터 직접 만들기"
categories: [AI & Tech, Claude]
tags:
  - AI
  - Dev Log
  - Web Dev
description: "Claude Code Mods와 Function Hooks의 구조, 기존 Hooks·Skills·MCP와의 차이, 추천 Mod 10개, 설치·보안·호환성, 검증한 TypeScript 예제까지 정리했다."
date: 2026-10-20 23:00:00 +0900
published: true
image:
  path: /assets/img/posts/2026-10-20-claude-code-mods-complete-guide/cover.jpg
  alt: "Anthropic 공식 Blast Radius Mod가 위험한 rm 명령의 영향 범위를 보여주는 실제 Claude Code 화면"
---

**Claude Mods는 Plan Mode나 Accept Edits Mode 같은 실행 모드가 아니다.** 2026년 10월 1일 Anthropic이 공개한, Claude Code 내부 이벤트를 JavaScript·TypeScript 함수로 가로채 UI와 동작을 바꾸는 **플러그인의 한 종류**다. 현재 공식 문서는 터미널에서 Claude Code **2.1.287 이상**, Desktop 앱에서는 내장 Claude Code **2.1.286 이상**을 요구한다.

이 글을 쓰는 PC의 전역 설치본은 2.1.215였다. 그래서 전역 업데이트나 서드파티 Mod 설치는 하지 않았다. 대신 공식 npm 패키지 2.1.304를 임시 캐시에서 실행해 직접 만든 Mod의 `validate`와 자동 테스트만 통과시켰다. 로그인된 대화형 세션에서 토스트가 뜨는 장면까지 확인한 것은 아니다. 통과한 범위와 못 한 범위를 먼저 까놓고 시작한다.

## 이 글에서 다루는 순서

1. Claude Mods의 정확한 정체
2. Settings Hooks·Skills·MCP·Plugins와의 차이
3. Function Hooks의 `$`, `e`, `next`와 실행 순서
4. 내장 Mod와 CLI·Desktop 지원 범위
5. 설치·활성화·비활성화·제거 절차
6. 실용성과 보안으로 고른 Mod 10개
7. `turn-done-toast`를 직접 만드는 전체 코드
8. 로컬 검증 결과와 실패 경계
9. 보안·호환성 체크리스트

---

## 1. Claude Mods는 정확히 무엇인가

[Anthropic 발표](https://claude.com/ko/resources/articles/claude-code-mods)는 Mod를 Claude Code의 작동 방식과 외관을 바꾸는 작은 TypeScript 함수라고 설명한다. 더 정확히 말하면 **`hooks/hooks.json`이 함수 모듈을 가리키는 Claude Code plugin**이다. Claude Code가 프롬프트 제출, 도구 호출, 턴 종료, UI 렌더링 같은 이벤트를 발생시키면 Mod의 핸들러가 그 사이에 들어간다.

할 수 있는 일의 폭이 꽤 넓다.

- 모델로 보내기 전 프롬프트를 수정하거나 중단한다.
- Bash·Edit·MCP 도구 호출을 허용, 수정, 거절, 재시도한다.
- 도구 결과에서 비밀 값을 가린 뒤 모델에 넘긴다.
- 프롬프트 위 밴드, 사이드 패널, 버튼, 입력창을 그린다.
- `/command`를 추가하고 모델 턴 없이 함수를 실행한다.
- Claude Code가 그리는 스피너, 도구 행, 질문 UI 일부를 다시 그린다.

여기서 이름 때문에 한 번 미끄러지기 쉽다. `default`, `acceptEdits`, `plan`, `dontAsk`는 **권한 모드**다. 무엇을 자동으로 실행할지 정한다. Mod는 그 모드가 아니다. Mod는 이벤트 핸들러를 심는 확장 코드이며, Plan Mode에서도 정책이 허용하는 범위 안에서 실행될 수 있다. 공식 가이드는 Plan Mode에서 `tool.check` 훅이 결정을 더 엄격하게 만들 수는 있지만 읽기 전용 제한을 느슨하게 만들 수는 없다고 명시한다.

### 왜 Settings Hooks만으로는 부족했나

기존 Settings Hooks도 `PreToolUse`, `PostToolUse`, `Stop` 같은 시점에 셸 명령, HTTP 요청, 프롬프트를 실행한다. 로그 남기기나 명령 차단에는 충분했다. 하지만 Claude Code 내부 UI 트리를 그리거나, 여러 이벤트 사이에서 메모리를 공유하거나, Claude Code의 기본 기능을 교체하는 데는 한계가 있었다.

Function Hooks는 Claude Code 안에서 함수로 돈다. 이벤트를 `next(e)`로 넘기기 전후에 코드를 실행하고, 결과를 직접 반환해 뒤쪽 체인을 끊을 수도 있다. UI와 동작을 한 플러그인 안에서 묶는 이유가 여기 있다.

## 2. Mod, Hooks, Skills, MCP, Plugin은 같은 말이 아니다

| 구성 요소 | 정체 | 잘하는 일 | 못 하는 일 또는 비용 |
|---|---|---|---|
| **Mod** | plugin 안에서 실행되는 JS/TS Function Hooks | 이벤트 수정·차단, UI, 명령어, 상태 공유 | 사용자 권한으로 실행되며 샌드박스가 아니다 |
| **Settings Hook** | `settings.json`에 연결한 셸·HTTP·prompt·agent hook | 기존 스크립트로 기록, 검사, 차단 | Claude Code UI를 직접 그리지 못한다 |
| **Skill** | `SKILL.md` 지침 묶음 | 반복 작업법과 도메인 지식을 Claude에게 가르친다 | 이벤트를 직접 가로채지 않는다 |
| **MCP server** | 외부 프로세스·서비스가 제공하는 도구 | 데이터베이스, 브라우저, 사내 API 연결 | 프로세스·도구 스키마·컨텍스트 비용이 생길 수 있다 |
| **Plugin** | Skills, agents, hooks, MCP, Mod를 한 번에 배포하는 폴더 | 설치·버전·공유 단위 | 그 자체가 특정 동작 방식은 아니다 |

핵심은 **Mod와 Plugin이 동급 개념이 아니라는 점**이다. 모든 Mod는 plugin이지만, 모든 plugin이 Mod는 아니다. `SKILL.md` 하나만 든 plugin도 있고, MCP 서버와 설정 훅만 든 plugin도 있다. `hooks.json`의 `modules`가 JS·TS 엔트리를 가리킬 때 그 plugin을 Mod라고 부른다.

![Function Hooks의 이벤트 전달과 결과 반환 순서](/assets/img/posts/2026-10-20-claude-code-mods-complete-guide/05-function-hooks-chain.png)
_그림 1. 공식 문서의 middleware 설명을 바탕으로 직접 그린 흐름도다. 이벤트는 안쪽으로, 결과는 바깥쪽으로 돌아온다._

## 3. Function Hooks를 읽는 최소 문법

작은 Mod의 핵심은 `register` 함수 하나다.

```ts
import type { On } from 'claude-code'

export function register(on: On): void {
  on('tool.call', { tool: 'Bash' }, async ($, e, next) => {
    if (/git push .*--force/.test(e.command)) {
      return { deny: 'Force push는 이 저장소에서 허용하지 않는다.' }
    }
    return next(e)
  })
}
```

### `register(on)`

Claude Code가 Mod를 로드할 때 한 번 호출하는 진입점이다. `on`은 이벤트와 핸들러를 연결하는 등록 함수다. 첫 인자는 `tool.call`, `prompt.submit`, `turn.complete`, `ui.render` 같은 이벤트명이다. 둘째 인자에 `{ tool: 'Bash' }` 같은 matcher를 넣으면 해당 이벤트만 좁힌다. 마지막 인자가 실제 handler다.

### `$`

Mod가 바깥 세계에 접근하는 공식 API다. `$.ui.toast`, `$.ui.open`, `$.fs.read`, `$.process.run`, `$.model.complete`, `$.state` 같은 namespace가 여기에 매달린다. 함수 모듈은 Node의 임의 전역을 쓰는 방식이 아니라 이 표면을 중심으로 작성한다. 그렇다고 보안 샌드박스가 되는 것은 아니다. 공식 문서는 Mod가 파일을 읽고 쓰고, 프로세스를 실행하고, 네트워크를 쓰고, 모델 호출로 사용량을 소비할 수 있다고 경고한다.

### `e`

지금 발생한 이벤트의 불변 데이터다. Bash `tool.call`이면 `e.tool`과 `e.command`, `prompt.submit`이면 `e.text`, `turn.complete`이면 `e.durationMs`, `e.isAborted`, `e.usage` 같은 값이 온다. 이벤트 객체는 깊게 동결되어 있으므로 직접 수정하지 않고 `{ ...e, text: ... }`처럼 복사해 넘긴다.

### `next`

다음 Mod의 핸들러, 마지막에는 Claude Code 기본 동작을 호출한다.

- `return next(e)`: 그대로 통과시킨다.
- `return next({ ...e, text: trimmed })`: 이벤트를 고쳐 통과시킨다.
- `const result = await next(e)`: 원래 동작 뒤에 후처리한다.
- `return { deny: 'reason' }`: `next`를 부르지 않고 직접 답해 뒤쪽을 끊는다.

이 구조는 Express 같은 middleware와 닮았다. 첫 번째로 로드된 Mod는 이벤트를 가장 먼저 보고 결과를 가장 늦게 본다. 그래서 Mod A가 `next`를 호출하지 않으면 Mod B와 Claude Code 기본 동작은 아예 실행되지 않는다.

### 여러 Mod의 순서와 충돌

[공식 이벤트 문서](https://code.claude.com/docs/en/plugins/mods/events)는 체인 순서를 다음처럼 정의한다.

1. `sec-default`와 조직의 `prependPlugins`
2. 사용자가 설치한 Mods
3. 조직의 `appendPlugins`
4. 나머지 Claude Code 내장 Mods

설치한 Mod끼리는 manifest의 `dependencies`가 순서에 영향을 준다. 한 모듈 안에서는 `on()`을 등록한 순서대로 돈다. 충돌은 꽤 현실적이다. 같은 `AbovePrompt` 밴드를 두 Mod가 바꾸거나, 하나가 도구 호출을 short-circuit하면 뒤쪽 Mod는 기회조차 못 얻는다. 보안 Mod라면 `.catch(...)`로 실패 시 차단하는 fail-closed 동작을 따로 설계해야 한다. 기본값은 훅이 실패하면 그 훅을 건너뛰는 쪽이라, “보안용이니 알아서 막아주겠지”가 통하지 않는다.

## 4. 현재 내장 Mod와 실행 환경

Claude Code 자체 기능 일부도 이미 Mod다. 공개 소스와 현재 공식 문서를 합치면 다음이 눈에 띈다.

- `cc-plugin-diff`: `/diff` 패널을 그린다. 비활성화하면 기본 `/diff`가 대신 답한다.
- `cc-plugin-agents-md`: 프로젝트의 `AGENTS.md`를 지침으로 로드한다.
- `cc-plugin-sec-default`: 조직 관리 설정과 권한 규칙을 사용자 Mod가 우회하지 못하도록 바깥쪽에서 감싼다.
- `cc-plugin-telemetry`: Claude Code 자체 analytics가 켜진 환경에서 내장 기록을 연결한다.
- `cc-plugin-you-should-know`: 놓친 정보를 찾아 프롬프트 위에 표시하는 기능이며 기본은 꺼져 있다.
- `cc-plugin-plugin-authoring`: Mod 코드를 만드는 내장 Skill이다. 이름은 plugin이지만 자체 Mod 코드는 없다.

공개 구현 저장소의 `mods/README.md`는 `sec-default`, `diff`, `telemetry`, `agents-md` 네 개의 소스를 제공한다. 설치한 바이너리 안의 복사본이 실제 실행본이고, GitHub `main`은 구조를 읽는 자료다. API가 아직 Early Access라 저장소 코드와 내 설치 버전이 어긋날 수 있다.

### CLI와 Desktop 차이

- 터미널 CLI: 2.1.287 이상에서 Mods가 기본 활성화된다.
- Claude Desktop Code 탭: 내장 Claude Code 2.1.286 이상이 필요하다. 로컬 세션에서 `/status`로 버전을 본다.
- Desktop의 WSL 세션: 공식 표에는 plugin을 쓸 수 없어 Mod도 동작하지 않는다고 적혀 있다.
- VS Code 채팅 패널: Function Hooks는 실행되지만 UI 렌더링은 확장 2.1.299 이상 같은 별도 최소 버전이 있다.
- `claude -p`와 Agent SDK: 훅은 실행되지만 그린 UI는 보이지 않는다.
- 클라우드 세션: 해당 세션에 plugin이 전달된 경우 훅은 실행될 수 있지만 Mod가 그린 UI는 표시되지 않는다.

![Anthropic 공식 Blast Radius Mod가 위험 명령의 삭제 대상을 보여주는 실제 화면](/assets/img/posts/2026-10-20-claude-code-mods-complete-guide/01-official-blast-radius-pane.png)
_그림 2. Anthropic의 Apache-2.0 `claude-code-playground`에 포함된 실제 Blast Radius 화면. `rm -rf build`가 지울 파일을 먼저 보여준다._

## 5. 설치 전에 할 일, 설치한 뒤 되돌리는 법

### 5-1. 버전부터 확인한다

```powershell
claude --version
```

2.1.287보다 낮으면 Mod 코드를 탓하기 전에 Claude Code를 업데이트해야 한다. Desktop은 Code 탭의 `/status`에서 앱 안쪽 Claude Code 버전을 따로 확인한다. CLI를 업데이트했다고 Desktop 내장본까지 바뀌는 것은 아니다.

### 5-2. 설치 전 소스를 받되 실행하지 않고 검사한다

```powershell
git clone https://github.com/anthropics/claude-code-playground.git
claude plugin validate .\claude-code-playground\claude-code\mods\blast-radius
```

`validate`는 manifest와 hooks module을 정적으로 분석한다. 출력의 `hooks:`에는 가로채는 이벤트가, `calls:`에는 `$.fs`, `$.process`, `$.model`처럼 Mod가 쓰는 API가 나온다. 통과는 “안전 인증”이 아니다. 숨은 의도, 정규식 누락, shell parsing 허점, 네트워크 목적까지 판단해주지는 않는다.

### 5-3. 한 세션에서만 불러본다

```powershell
claude --plugin-dir .\claude-code-playground\claude-code\mods\blast-radius
```

`--plugin-dir`는 marketplace에 설치하지 않고 그 세션에만 폴더를 로드한다. 출처를 막 검토한 Mod라면 이 방식이 첫 선택이다. `/plugin`의 Installed 탭에서 활성 Mod를 확인한다.

### 5-4. marketplace에서 설치한다

```text
/plugin marketplace add plagemes/claude-mods
/plugin install scope-lock@claude-mods
/reload-plugins
```

셸에서도 같은 일을 할 수 있다.

```powershell
claude plugin marketplace add plagemes/claude-mods
claude plugin install scope-lock@claude-mods --scope local
```

`user`는 모든 프로젝트, `project`는 저장소 구성에 공유, `local`은 내 컴퓨터의 현재 저장소에만 적용된다. 낯선 Mod는 `local`이나 `--plugin-dir`로 시작하는 편이 낫다.

주의할 것이 하나 있다. `Plagemes/claude-mods`와 `mishgoldenberg/claude-mods`는 둘 다 marketplace 내부 이름을 **`claude-mods`**로 쓴다. 두 카탈로그를 한 설정에 무심코 겹쳐 등록하면 이름 충돌을 일으킬 수 있다. 하나를 고르거나, 저장소를 각각 clone한 뒤 원하는 plugin 폴더만 `--plugin-dir`로 시험하는 편이 명확하다.

### 5-5. 끄기, 다시 켜기, 제거하기

```powershell
claude plugin disable scope-lock@claude-mods
claude plugin enable scope-lock@claude-mods
claude plugin uninstall scope-lock@claude-mods --scope local
```

세션 안에서는 `/plugin`의 Installed 탭에서 Space로 켜고 끄거나 상세 메뉴에서 제거한다. 열린 세션에 반영하려면 `/reload-plugins`를 쓴다. 모든 사용자 Mod를 한 세션만 끄고 원인 분리하려면 `claude --safe-mode`가 빠르다. `~/.claude/settings.json`의 `"disableAllHooks": true`는 설치된 Mod와 Settings Hooks, custom status line을 함께 멈추므로 범위가 넓다.

marketplace 자체를 제거하면 그 marketplace에서 설치한 plugins도 함께 제거된다. `claude plugin marketplace remove <name>`을 습관처럼 치기 전에 목록을 먼저 봐야 하는 이유다.

## 6. 보안과 실용성으로 고른 Mods 10개

아래는 별 개수 순위가 아니다. 이벤트 범위가 읽기 쉬운가, 모델을 추가 호출하는가, 실패했을 때 무엇이 깨지는가, 일상 작업에서 시간을 돌려주는가를 기준으로 골랐다. 유지보수 상태는 2026년 10월 20일 GitHub API 기준이다. 세 저장소 모두 기능이 발표된 지 얼마 안 된 초기 프로젝트다.

| Mod | 정확한 기능과 UI | 설치 또는 일회성 실행 | 추가 모델 호출 | 실용 사례·제한 | 유지보수 신호 |
|---|---|---|---|---|---|
| **Blast Radius** ([Anthropic](https://github.com/anthropics/claude-code-playground/tree/main/claude-code/mods/blast-radius)) | 위험한 `rm`, hard reset, force push 전에 영향 파일·커밋을 pane과 버튼으로 보여준다 | clone 후 `claude --plugin-dir ...\blast-radius` | 없음 | 삭제 범위를 눈으로 확인한다. 패턴 기반 예제라 샌드박스 대체재는 아니다 | Apache-2.0, repo push 2026-10-14 |
| **scope-lock** ([Plagemes](https://github.com/Plagemes/claude-mods/tree/main/mods/scope-lock)) | `/scope src/auth/** tests/auth/` 밖의 Edit·Write와 흔한 Bash 쓰기를 막고 status line에 범위를 표시한다 | `claude plugin install scope-lock@claude-mods` | 없음 | 큰 저장소에서 작업 범위 이탈을 줄인다. interpreter·script 안의 숨은 쓰기와 symlink는 놓칠 수 있다 | MIT, repo push 2026-10-15 |
| **cost-meter** ([Plagemes](https://github.com/Plagemes/claude-mods/tree/main/mods/cost-meter)) | `turn.complete` 사용량으로 `$0.42 · 128k tok` status line을 만든다 | `claude plugin install cost-meter@claude-mods` | 없음 | 세션 비교에 유용하다. 가격표 기반 추정치이며 청구서가 아니고, 오류 턴과 일부 side request는 빠진다 | MIT, repo push 2026-10-15 |
| **tool-timeline** ([Plagemes](https://github.com/Plagemes/claude-mods/tree/main/mods/tool-timeline)) | `/timeline` pane에 도구명, 성공·실패·거절, 시간, 입력 요약을 300건까지 그린다 | `claude plugin install tool-timeline@claude-mods` | 없음 | 느린 빌드와 반복 실패를 찾는다. 세션이 끝나면 목록이 사라진다 | MIT, repo push 2026-10-15 |
| **context-keeper** ([mishgoldenberg](https://github.com/mishgoldenberg/claude-mods)) | context panel, compact 전 transcript archive, `.claude/checkpoints/` handoff를 제공한다 | `claude plugin install context-keeper@claude-mods` | handoff 요약을 켜면 호출 | `/clear` 뒤 맥락 복원에 좋다. checkpoint 파일이 생기며 handoff 요약은 토큰을 쓴다 | MIT, repo push 2026-10-06 |
| **usage-meter** ([mishgoldenberg](https://github.com/mishgoldenberg/claude-mods)) | 5시간·7일 한도, reset countdown, burn rate, session cost, cache ratio, sparkline을 panel로 그린다 | `claude plugin install usage-meter@claude-mods` | 없음 | 한도 소진 속도를 본다. UI 값은 Claude Code가 노출하는 usage에 의존한다 | MIT, repo push 2026-10-06 |
| **notify** ([mishgoldenberg](https://github.com/mishgoldenberg/claude-mods)) | 긴 턴, subagent 종료, 질문, 15초 넘은 승인 대기를 inbox와 OS 알림으로 보낸다 | `claude plugin install notify@claude-mods` | 없음 | 탭을 떠나 있어도 대기를 알아챈다. OS 알림 도구와 권한 차이가 있다 | MIT, repo push 2026-10-06 |
| **activity** ([mishgoldenberg](https://github.com/mishgoldenberg/claude-mods)) | 현재 실행 중 명령, 경과 시간, 승인 대기, subagent와 할 일 진행률을 live panel로 보여준다 | `claude plugin install activity@claude-mods` | 없음 | “멈춘 건가, 묻고 있는 건가”를 구분한다. UI 폭과 surface에 따라 panel 배치가 달라진다 | MIT, repo push 2026-10-06 |
| **guardrails** ([mishgoldenberg](https://github.com/mishgoldenberg/claude-mods)) | `rm -rf`, force push, destructive Git, secrets, install·network를 rule preset으로 차단한다 | `claude plugin install guardrails@claude-mods` | 없음 | 실수 방지용 seatbelt다. 난독화 명령이나 이미 존재하는 script는 빠질 수 있다 | MIT, repo push 2026-10-06 |
| **changes** ([mishgoldenberg](https://github.com/mishgoldenberg/claude-mods)) | 세션에서 바뀐 파일과 `+/-`를 pane에 모으고 why·summary·self-review 버튼을 제공한다 | `claude plugin install changes@claude-mods` | 버튼이 모델 동작을 부를 수 있음 | 리뷰 전 변경 범위를 훑기 좋다. `git diff --numstat` process를 실행한다 | MIT, repo push 2026-10-06 |

Plagemes 저장소는 200개가 넘는 카탈로그를 내세우지만 2026년 10월 15일 기준 별 2개, mishgoldenberg 저장소는 별 9개였다. 별이 적다고 나쁘다는 뜻도, README가 길다고 안전하다는 뜻도 아니다. 이제 막 생긴 생태계라는 신호로 읽는 편이 맞다. 자동 업데이트도 서드파티 marketplace는 기본 off이므로, 업데이트 전에 diff를 다시 보는 습관이 필요하다.

![mishgoldenberg context-keeper와 usage-meter의 공식 데모](/assets/img/posts/2026-10-20-claude-code-mods-complete-guide/02-community-context-usage.gif)
_그림 3. MIT 저장소가 제공한 실제 Desktop Code 탭 데모. context panel, checkpoint, usage meter의 동작을 순서대로 보여준다._

![mishgoldenberg activity Mod의 공식 데모](/assets/img/posts/2026-10-20-claude-code-mods-complete-guide/03-community-activity.gif)
_그림 4. MIT 저장소의 실제 activity 데모. 명령 실행과 승인 대기가 panel에 표시된다. 이 글 작성 환경에서 직접 설치한 화면은 아니다._

`activity`는 실행 대신 임시 clone에서 소스와 manifest를 검토하고 최신 2.1.304 validator에 넣었다. commit `0954c978c1101ebea60edee1331c8586a5d0ff6f`에서 `session.start`, `command.run`, `turn.start`, `turn.complete`, `tool.call`, `ui.render`가 탐지됐고, `$.agent.list`, `$.clock`, `$.command.register`, `$.state`, `$.tool.check`, `$.ui` 호출이 보고됐다. `$.model`과 네트워크 호출은 없었다. validator는 `tool.call`에 `.catch`가 없는 gating hook도 조언으로 표시했다. 통과는 했지만, 이 한 줄 때문에 보안 장치로 오해하지 않고 관찰용 UI로만 추천했다.

### 무엇부터 설치할까

내 기준의 시작 순서는 단순하다.

1. 처음에는 공식 **Blast Radius**를 `--plugin-dir`로 읽고 시험한다.
2. 작업 범위 이탈이 문제라면 **scope-lock** 하나만 추가한다.
3. 기다림이 문제라면 **notify**, 원인 파악이 문제라면 **activity**를 고른다.
4. 비용 숫자가 필요하면 **usage-meter**나 **cost-meter** 중 하나만 먼저 쓴다.
5. 여러 guard를 겹치기 전 같은 이벤트를 누가 먼저 잡는지 확인한다.

한꺼번에 열 개를 켜면 편해지는 게 아니라 디버깅 표면이 열 배로 늘어난다. Mod도 의존성이다.

## 7. 직접 만든 미니 Mod, `turn-done-toast`

실습은 “작업 완료 알림”으로 정했다. 위험 명령 guard는 정규식과 fail-closed 정책을 잘못 잡으면 오히려 안전하다고 착각하기 쉽다. 반면 완료 토스트는 읽기 전용 이벤트를 관찰하고 UI 호출 한 번만 한다. 망가져도 작업 결과를 바꾸지 않는다.

### 디렉터리 구조

```text
turn-done-toast/
├── .claude-plugin/
│   └── plugin.json
├── hooks/
│   ├── hooks.json
│   └── register.ts
├── tests/
│   └── turn-done-toast.test.ts
└── README.md
```

### `.claude-plugin/plugin.json`

```json
{
  "name": "turn-done-toast",
  "version": "0.1.0",
  "description": "Shows a local Claude Code toast after a long main-session turn completes",
  "author": { "name": "AI Autoblog editorial example" },
  "license": "MIT"
}
```

Mod 전용 magic field는 없다. 이것은 평범한 plugin manifest다.

### `hooks/hooks.json`

```json
{
  "description": "Long-turn completion toast",
  "modules": ["./register.ts"]
}
```

`modules`가 TypeScript 엔트리를 가리키는 순간 Function Hooks module이 생긴다. 경로는 `hooks.json` 기준 상대 경로다.

### `hooks/register.ts`

```ts
import type { On } from 'claude-code'

const MINIMUM_DURATION_MS = 20_000

export function register(on: On): void {
  on('turn.complete', async ($, e, next) => {
    const result = await next(e)

    if (!e.agentId && !e.isAborted && e.durationMs >= MINIMUM_DURATION_MS) {
      $.ui.toast(`Claude 작업 완료 · ${Math.round(e.durationMs / 1000)}초`)
    }

    return result
  })
}
```

`await next(e)`로 Claude Code의 원래 턴 종료 처리를 먼저 끝낸다. `agentId`가 있으면 subagent이므로 제외하고, 중단하지 않은 20초 이상 main turn만 토스트를 띄운다. `$.ui.toast`는 기본 약 4초 표시된다. 파일, 프로세스, 네트워크, 모델 API는 건드리지 않는다.

### 의존성

없다. 공식 문서는 `.js`와 `.ts`를 Claude Code가 직접 로드하므로 Node, bundler, build step이 필요 없다고 설명한다. `import type`은 실행 의존성이 아니라 현재 Claude Code가 생성하는 `claude-code` 타입 선언을 위한 것이다. 대화형 `--plugin-dir` 세션에서 `.claude-plugin/types/`에 내 버전의 선언이 생성되며, 문서보다 그 파일을 우선해야 한다.

### 자동 테스트

테스트는 긴 턴에서 토스트 한 번, 짧은 턴에서 0번을 확인한다. 테스트 kit가 `turn.complete`와 `ui.toast`를 메모리에서 흉내 내므로 실제 모델, 로그인, 네트워크를 쓰지 않는다.

```ts
import { expect, test } from 'claude-code/testing'

test('a long main-session turn shows one completion toast', async ($, on) => {
  const toasts: string[] = []
  on('turn.complete', () => ({ text: '' }))
  on('ui.toast', ($, e) => {
    toasts.push(e.text)
    return { value: undefined }
  })

  await $.turn.complete({
    turnId: 'turn-1',
    answer: 'Done',
    durationMs: 25_000,
    isAborted: false,
    usage: null,
  })

  expect(toasts).toEqual(['Claude 작업 완료 · 25초'])
})

test('a short turn stays quiet', async ($, on) => {
  const toasts: string[] = []
  on('turn.complete', () => ({ text: '' }))
  on('ui.toast', ($, e) => {
    toasts.push(e.text)
    return { value: undefined }
  })

  await $.turn.complete({
    turnId: 'turn-2',
    answer: 'Done',
    durationMs: 3_000,
    isAborted: false,
    usage: null,
  })

  expect(toasts).toEqual([])
})
```

```powershell
claude plugin validate .\turn-done-toast
claude plugin test .\turn-done-toast
```

내 시스템 설치본이 너무 오래되어 최신 공식 패키지를 전역 설치하지 않고 이렇게 실행했다.

```powershell
npm exec --yes --package=@anthropic-ai/claude-code@2.1.304 -- `
  claude plugin validate .\turn-done-toast

npm exec --yes --package=@anthropic-ai/claude-code@2.1.304 -- `
  claude plugin test .\turn-done-toast
```

![직접 만든 Mod 소스와 Claude Code 2.1.304 실제 검증 출력](/assets/img/posts/2026-10-20-claude-code-mods-complete-guide/04-local-mod-source-test.png)
_그림 5. 로컬 소스와 실제 stdout을 그대로 재현한 캡처. `turn.complete`와 `$.ui.toast`가 탐지됐고 테스트 2건이 통과했다._

실측 결과는 다음과 같다.

```text
2.1.304 (Claude Code)
./register.ts hooks: turn.complete
./register.ts calls: $.ui.toast
Validation passed

(pass) a long main-session turn shows one completion toast
(pass) a short turn stays quiet
2 pass
0 fail
```

### Claude Code에서 활성화한다

```powershell
claude --plugin-dir .\turn-done-toast
```

그다음 `/plugin` Installed 탭에서 `turn-done-toast`가 active인지 본다. 짧은 답에는 변화가 없어야 한다. 20초 넘게 끝난 정상 main turn에는 `Claude 작업 완료 · N초` 토스트가 떠야 한다.

여기에는 중요한 선이 있다. **이 글은 대화형 세션의 실제 토스트 렌더링까지 확인하지 않았다.** 로컬 설치본 2.1.215가 공식 최소 버전보다 낮았고, 로그인 상태나 전역 바이너리를 바꾸지 않았기 때문이다. 자동 테스트는 hook이 `$.ui.toast`를 정확히 호출하는 것까지 검증하지만 Desktop·terminal renderer의 최종 픽셀까지 보증하지 않는다.

### 실행 전후를 정직하게 비교하면

- 전: `turn.complete` 뒤에 기본 Claude Code UI만 남는다.
- 후: 조건을 만족한 턴에서 토스트 API 호출이 한 번 발생한다. 이 부분은 자동 테스트로 확인했다.
- 미확인: 이 PC의 로그인된 interactive terminal과 Desktop에서 토스트가 실제로 보이는 장면.
- 추가 토큰: 이 Mod 자체는 `$.model.*`을 부르지 않아 추가 모델 호출이 없다. plugin에 다른 skill·agent를 함께 넣으면 그 설명이 상시 컨텍스트에 들어갈 수 있으므로 plugin 전체 비용은 별도다.

## 8. 버전 차이를 몸으로 맞은 기록

시스템 설치본 2.1.215에서 같은 폴더를 검증하자 이런 결과가 나왔다.

```text
hooks: Invalid input: expected record, received undefined
Validation failed

error: unknown command 'test'
(Did you mean list?)
```

코드가 틀린 게 아니라 CLI가 Function Hooks schema와 `plugin test`를 아직 모르는 버전이었다. 최신 2.1.304에서는 같은 파일이 그대로 통과했다. Mods API를 “한번 나온 안정 API”처럼 복사해 블로그에 박아두면 안 되는 이유다.

문제가 생기면 이 순서로 좁힌다.

1. `claude --version`과 Desktop `/status`를 각각 본다.
2. `claude plugin validate <dir>`의 `hooks:`와 `calls:`가 비어 있지 않은지 본다.
3. `claude plugin test`가 “hooks modules are turned off”라고 말하는지 확인한다.
4. `/plugin`의 active Mods 줄과 Installed 탭을 본다.
5. `claude --debug`로 hooks module load·skip 이유를 찾는다.
6. `--safe-mode`에서 문제가 사라지면 설치 Mod를 하나씩 다시 켠다.
7. UI만 안 보이면 terminal 폭, Desktop·VS Code 버전, 해당 render site 지원 여부를 본다.

## 9. 보안 체크리스트, 여기서는 겁을 좀 먹는 게 맞다

Anthropic의 경고는 짧고 세다. **Mod는 Claude Code와 같은 사용자 권한으로 실행되며 sandboxed가 아니다.** Claude Code의 Bash sandbox를 켜도 Mod가 직접 시작한 프로세스는 그 바깥에서 돈다.

설치 전에는 최소한 이것을 확인한다.

- `plugin.json`, `hooks.json`, 모든 module import를 읽는다.
- `claude plugin validate`의 `calls:`에서 `$.fs`, `$.process`, `$.http`, `$.model`, `$.prompt.submit`을 특히 본다.
- 환경 변수, `~/.claude/settings.json`, `.env`, SSH·cloud credential을 읽는 경로가 있는지 찾는다.
- `prompt.submit`, `prompt.section`, `tool.call`, `tool.check`가 프롬프트·도구를 어떻게 고치는지 본다.
- `next(e)`를 부르지 않는 분기와 `.catch` 실패 정책을 확인한다.
- model call이 있다면 어떤 입력을 보내고 몇 번 호출하는지 확인한다.
- marketplace auto-update가 켜져 있는지, 검토한 commit이나 tag에 pin할 수 있는지 본다.
- 조직의 `allowManagedModsOnly`, `prependPlugins`, `appendPlugins`, marketplace allowlist를 확인한다.

보안 Mod도 안전의 최종선이 아니다. 정규식 guard는 shell quoting, 난독화, interpreter 안쪽 실행을 놓친다. scope guard는 symlink나 실행된 script의 쓰기를 놓칠 수 있다. 알림 Mod는 OS 권한 때문에 조용히 실패할 수 있다. context·usage panel은 API가 노출한 수치에 의존한다.

그리고 “Mod가 토큰을 무조건 아낀다”는 말도 틀렸다. `$.session.usage()`를 읽어 UI만 그리는 Mod는 모델 호출을 추가하지 않는다. 반대로 `$.model.complete`로 prompt를 평가하거나 handoff를 쓰는 Mod는 사용량을 더 쓴다. plugin 안의 Skills·agents 설명은 활성화된 세션의 컨텍스트에도 일정 비용을 준다. `claude plugin details <name>`의 Always-on 수치를 확인해야 한다.

## 마지막 판단

Claude Mods는 Claude Code를 “설정 가능한 도구”에서 “이벤트와 UI를 교체할 수 있는 플랫폼”으로 한 단계 밀었다. 동시에 신뢰 경계도 훨씬 넓혔다. 좋은 Mod 하나는 승인 대기를 알려주고, 작업 범위를 지키고, 긴 도구 체인을 보이게 만든다. 나쁜 Mod 하나는 프롬프트와 비밀 값, 파일, 네트워크, 모델 사용량을 모두 만질 수 있다.

그래서 첫 설치는 화려한 200개 묶음이 아니라 **소스가 짧은 한 개**가 맞다. `validate`로 표면을 보고, `plugin test`로 동작을 고정하고, `--plugin-dir`로 한 세션만 돌린다. 이상이 없을 때만 local scope로 남긴다. Mods는 마법이 아니라 코드다. 코드답게 의심하고, 테스트하고, 버전을 적어두면 된다.

---

## 공식·원본 자료

- [Anthropic 발표: mod로 Claude Code 사용자 지정](https://claude.com/ko/resources/articles/claude-code-mods)
- [Claude Code 공식 Mods 개요](https://code.claude.com/docs/en/plugins/mods/overview)
- [Claude Code 공식 Mod 제작 가이드](https://code.claude.com/docs/en/plugins/mods/create)
- [Claude Code 공식 Events 가이드](https://code.claude.com/docs/en/plugins/mods/events)
- [Claude Code 공식 Mods Reference](https://code.claude.com/docs/en/plugins/mods/reference)
- [Claude Code 공식 Plugin 설치·관리](https://code.claude.com/docs/en/plugins/install)
- [Claude Code 공식 Plugin 보안](https://code.claude.com/docs/en/plugins/security)
- [Anthropic 공개 내장 Mods 구현](https://github.com/anthropics/claude-code/tree/main/mods)
- [Function Hooks 설계 논의 #91870](https://github.com/anthropics/claude-code/issues/91870)
- [현재 공개 TypeScript API 선언](https://github.com/anthropics/claude-code/blob/main/mods/types/claude-code.d.ts)
- [Anthropic sample Mods](https://github.com/anthropics/claude-code-playground/tree/main/claude-code/mods)
- [Plagemes/claude-mods](https://github.com/Plagemes/claude-mods)
- [mishgoldenberg/claude-mods](https://github.com/mishgoldenberg/claude-mods)
