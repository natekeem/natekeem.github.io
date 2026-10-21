---
title: "Claude Code 커스텀 Skills, SKILL.md부터 검증까지"
categories: [AI & Tech, Claude]
tags:
  - AI
  - Web Dev
  - Dev Log
description: "Anthropic 공식 문서를 기준으로 Claude Code 커스텀 Skill의 폴더 구조, YAML frontmatter, 자동·수동 호출, 권한 경계, 보조 파일 분리, 최소 예제와 실제 로컬 검증 결과를 정리했다."
date: 2026-10-20 21:00:00 +0900
published: true
image:
  path: /assets/img/posts/2026-10-20-claude-code-custom-skills-guide/cover.jpg
  alt: "실제 SKILL.md 파일과 Claude Code 명령이 종이 매뉴얼처럼 펼쳐진 편집 커버"
---

Claude Code에 긴 프롬프트를 매번 붙여 넣는다면, 그 프롬프트는 이미 Skill이 될 준비가 끝난 셈이다. Anthropic의 현재 공식 규격에서 최소 단위는 거창한 플러그인이 아니라 **이름이 정해진 폴더 안의 `SKILL.md` 한 장**이다.

직접 문서를 따라가 보니 핵심은 문법보다 경계였다. Claude가 언제 이 지침을 발견하는지, 사용자가 명시적으로 호출해야 하는지, 어떤 도구를 그 한 번의 호출에 허용할지, 그리고 실행 결과를 어디까지 믿을지가 더 중요했다.

이 글은 2026년 10월 20일 기준 [Anthropic의 공식 Claude Code Skills 문서](https://code.claude.com/docs/en/skills)와 공식 플러그인 예제를 대조해 정리했다. 로컬에는 Claude Code native 2.1.215가 설치되어 있었고 `claude doctor`는 설치 문제 없음으로 끝났다. 다만 계정 로그인이 없어 모델 호출은 `Not logged in · Please run /login`에서 멈췄다. 따라서 아래 예제는 **구조와 로컬 CLI 탐색까지 검증했지만, 실제 응답 품질까지 실행 완료했다고 주장하지 않는다.**

## Skill은 프롬프트 파일이 아니라 발견 가능한 작업 단위다

![프로젝트 Skill 폴더, 메타데이터 발견, 본문 지연 로딩 흐름](/assets/img/posts/2026-10-20-claude-code-custom-skills-guide/01-skill-folder-and-loading.png)

가장 작은 프로젝트 Skill의 구조는 이렇다.

```text
your-project/
└─ .claude/
   └─ skills/
      └─ change-summary/
         └─ SKILL.md
```

Claude Code는 시작 시 모든 Skill의 전체 본문을 통째로 넣지 않는다. 공식 문서 설명대로 이름과 설명 같은 메타데이터로 사용 가능성을 먼저 판단하고, 관련성이 생기거나 사용자가 `/change-summary`처럼 호출했을 때 본문을 로드한다. Skill이 많아져도 매번 모든 지침을 컨텍스트에 싣지 않는 이유다.

범위는 위치로 정한다.

| 위치 | 범위 | 적합한 용도 |
| --- | --- | --- |
| `~/.claude/skills/<name>/SKILL.md` | 개인의 모든 프로젝트 | 개인 글쓰기 규칙, 반복 리뷰 습관 |
| `.claude/skills/<name>/SKILL.md` | 현재 프로젝트 | 팀 코드리뷰, 배포 체크, 저장소 규칙 |
| 플러그인의 `skills/` | 플러그인 설치 사용자 | 여러 저장소에 배포할 조직 워크플로 |

프로젝트 Skill은 코드와 함께 리뷰할 수 있다는 장점이 있다. 대신 저장소를 클론한 사람의 Claude Code가 읽고 실행할 수 있는 지침이기도 하다. 출처가 불분명한 Skill을 “마크다운이니까 안전하겠지” 하고 설치하면 안 된다.

## 최소 SKILL.md를 실제로 써보자

아래 파일은 이 글과 함께 둔 최소 예제다.

```markdown
---
name: change-summary
description: Summarize the current Git working tree into changed files, risk points, and recommended verification. Use only when the user explicitly asks for a change summary, diff review, or commit preparation.
argument-hint: "[base-ref, defaults to HEAD]"
disable-model-invocation: true
allowed-tools: Bash(git status:*), Bash(git diff:*)
---

# Change Summary

Create a concise engineering handoff for the current working tree.

Use `$ARGUMENTS` as the base ref. If it is empty, use `HEAD`.

1. Run `git status --short`.
2. Run `git diff --stat $ARGUMENTS`.
3. Read `git diff $ARGUMENTS`.
4. Report:
   - changed files and intent
   - behavior or data risks
   - tests already evident from the diff
   - the smallest useful verification command

Do not modify files, stage changes, commit, push, or invent test results.
```

호출은 간단하다.

```text
/change-summary HEAD
```

`$ARGUMENTS`에는 `HEAD`가 들어간다. `argument-hint`는 명령 입력창에서 기대하는 인자를 알려준다. `disable-model-invocation: true`를 넣었으므로 Claude가 대화 도중 알아서 이 Skill을 부르지 못하고 사용자가 명시적으로 호출해야 한다.

이 설정은 일부러 보수적으로 잡았다. 변경 요약 자체는 읽기 작업이지만, 나중에 누군가 “요약 뒤 커밋까지”를 본문에 추가할 수 있다. 부작용이 생길 수 있는 워크플로는 자동 호출을 끄는 편이 낫다.

## frontmatter에서 실제로 중요한 여섯 줄

### `name`

생략할 수 있지만 폴더명과 일치하는 소문자·숫자·하이픈 이름을 명시하는 편이 읽기 쉽다. 호출 이름도 예측 가능해진다.

### `description`

가장 중요한 줄이다. “코드를 검토한다”보다 **언제 써야 하고 언제 쓰면 안 되는지**까지 써야 자동 선택 오작동이 줄어든다. 본문에 트리거 조건을 숨겨두면 Claude는 본문을 읽기 전까지 그 조건을 모른다.

### `disable-model-invocation`

`true`면 사용자만 슬래시 명령으로 부른다. 배포, 이메일 전송, 파일 대량 변경처럼 부작용이 있는 Skill은 이쪽이 기본값에 가깝다.

반대로 `user-invocable: false`를 쓰면 메뉴에는 숨기고 Claude만 배경 지식처럼 사용할 수 있다. 코딩 규칙이나 제품 도메인 설명에 어울린다.

### `allowed-tools`

이름 때문에 자주 오해한다. 공식 문서는 이것이 **Skill이 사용할 수 있는 도구를 제한하는 샌드박스가 아니라, 그 호출 동안 별도 승인 없이 쓸 수 있게 하는 권한 부여**라고 명시한다. 다른 도구를 원천 금지하려면 조직 정책과 세션 권한을 따로 설계해야 한다.

예제에서는 `git status`와 `git diff` 계열만 사전 허용했다. `git add`, `commit`, `push`는 넣지 않았다.

### `argument-hint`와 `$ARGUMENTS`

하나의 Skill을 “현재 변경”, “main 기준”, “특정 태그 기준”으로 재사용하려면 인자를 받는 게 낫다. 공식 문서는 `$ARGUMENTS` 외에 위치 인자도 지원한다. 인자를 그대로 셸에 넣는 Skill이라면 입력 검증이 빠지지 않았는지 반드시 리뷰해야 한다.

### `context`, `agent`, `hooks`

별도 컨텍스트나 서브에이전트, 훅이 필요한 고급 옵션이다. 처음부터 모두 쓰면 디버깅 지점만 늘어난다. 한 파일짜리 Skill이 안정적으로 호출되는 것을 확인한 다음 확장하는 게 낫다.

## SKILL.md가 길어지면 설명서를 옆으로 뺀다

Anthropic은 `SKILL.md` 본문을 500줄 이하로 유지하고 세부 자료를 같은 폴더의 보조 파일로 나누라고 권한다.

```text
release-check/
├─ SKILL.md
├─ references/
│  ├─ environments.md
│  └─ rollback.md
└─ scripts/
   └─ verify-release.ps1
```

`SKILL.md`에는 작업 순서와 어떤 자료를 언제 읽을지만 남긴다. 환경 목록이나 긴 명령 사전까지 본문에 몰아넣으면, 호출할 때마다 불필요한 토큰이 들어가고 중요한 금지 규칙이 묻힌다.

스크립트를 함께 둘 때는 더 엄격해야 한다.

- 상대 경로는 Skill 폴더를 기준으로 적는다.
- 네트워크 호출, 삭제, 배포, 비밀 접근을 문서에 드러낸다.
- 인자를 명령 문자열에 그대로 이어 붙이지 않는다.
- 성공 메시지는 실제 exit code와 산출물 검증 뒤에만 출력한다.

## 자동 호출과 수동 호출을 나누는 기준

| 작업 | 권장 호출 | 이유 |
| --- | --- | --- |
| 코드 스타일 안내 | 자동 허용 | 부작용이 없고 자주 필요하다 |
| 변경 요약·리뷰 | 수동 우선 | 기준 ref와 검토 범위를 사용자가 고르는 편이 안전하다 |
| 배포 체크리스트 | 수동 전용 | 외부 상태와 승인 여부가 얽힌다 |
| 커밋·푸시·이슈 생성 | 수동 전용 + 별도 승인 | 되돌리기 어렵거나 외부에 기록이 남는다 |
| 제품 용어집 | 모델 전용 가능 | 사용자가 직접 호출할 이유가 적다 |

“Claude가 알아서 해주면 편하다”는 이유만으로 자동 호출을 켜면, 관련 단어 하나에 무거운 워크플로가 튀어나온다. 자동화의 시작점은 편의가 아니라 오작동 비용이다.

## 로컬에서 어디까지 검증했나

![Claude Code 2.1.215 doctor 결과와 로그인에서 멈춘 실제 테스트 기록](/assets/img/posts/2026-10-20-claude-code-custom-skills-guide/02-claude-code-local-test.png)

이번 예제는 Windows의 독립 작업 폴더에서 다음 순서로 확인했다.

```powershell
claude --version
# 2.1.215 (Claude Code)

claude doctor
# exit 0
# No installation issues found.

claude -p "/change-summary HEAD" --output-format json
# exit 1
# Not logged in · Please run /login
```

여기서 선을 그어야 한다. `doctor` 성공은 네이티브 설치와 로컬 구성에 치명적인 문제가 없다는 뜻이지, Skill의 모델 실행이 성공했다는 뜻이 아니다. 실제 호출은 인증 전에 멈췄고 API 토큰 사용량도 0이었다.

로그인된 환경에서 최종 확인할 항목은 네 가지다.

1. `/change-summary`가 명령 후보에 나타나는가.
2. `HEAD`가 `$ARGUMENTS`로 전달되는가.
3. `git status`와 `git diff`만 승인 없이 실행되는가.
4. 파일 수정·stage·commit·push 없이 요약만 반환하는가.

실패 기록을 숨기고 “정상 동작했다”고 쓰는 순간 이 글 전체가 무너진다. 최소 예제는 제공하지만, 현재 환경에서의 실질적 발행 블로커도 바로 이 미완료 호출 검증이다.

## 품질은 트리거 테스트와 결과 테스트를 분리해야 한다

Anthropic 공식 문서도 Skill이 호출됐다는 사실만으로 품질을 입증할 수 없다고 말한다. 나는 다음처럼 두 묶음으로 나누는 게 맞다고 본다.

**트리거 평가**

- “현재 변경사항 요약해줘”에서 불리는가.
- “오늘 날씨 알려줘”에서 불리지 않는가.
- 수동 전용 Skill이 모델 판단으로 호출되지 않는가.

**결과 평가**

- 변경 파일을 빠뜨리지 않는가.
- diff에 없는 테스트 성공을 만들지 않는가.
- 위험과 사실을 구분하는가.
- 권장 검증 명령이 프로젝트에 실제로 존재하는가.

가능하면 새 세션에서 Skill을 켠 경우와 끈 경우를 같은 입력으로 비교한다. 대화 이력에 답이 묻어 있으면 Skill 효과를 측정한 게 아니다.

## 설치보다 먼저 읽을 것

GitHub에서 받은 Skill은 실행 가능한 지침이다. `allowed-tools`, backtick 명령 치환, `scripts/`, 훅, 외부 URL, 자격 증명 접근을 먼저 읽는다. 라이선스가 있어도 안전성이 자동으로 보증되지는 않는다.

커스텀 Skill의 가장 좋은 첫 과제는 거대한 배포 봇이 아니다. 팀이 이미 매주 복사하는 20줄짜리 읽기 전용 체크리스트다. 그 파일이 언제 로드되고, 무엇을 읽고, 무엇을 절대 하지 않는지 설명할 수 있다면 다음 단계로 가도 된다.

## 확인한 공식 자료

- [Extend Claude with skills](https://code.claude.com/docs/en/skills)
- [Claude Code plugins reference](https://code.claude.com/docs/en/plugins-reference)
- [Anthropic 공식 example-command SKILL.md](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/example-plugin/skills/example-command)
- [Anthropic Knowledge Work Plugins](https://github.com/anthropics/knowledge-work-plugins)

_예제 파일은 글의 검수 패킷에 함께 들어 있다. 2026-10-20 현재 로컬 CLI 설치 진단은 통과했지만 Claude 계정 인증이 없어 실제 모델 응답 검증은 보류했다._
