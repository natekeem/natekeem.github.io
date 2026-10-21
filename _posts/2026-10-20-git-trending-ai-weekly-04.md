---
title: "[Git Trending AI #04] 별점보다 실무가 보인 5개"
categories: [AI & Tech, Git Trending]
tags:
  - AI
  - Git Trending
  - Dev Log
description: "2026년 10월 20일 GitHub Trending에서 겹치는 프로젝트를 걷어내고, 보안 감사·코드 그래프·에이전트 IDE·구직 워크플로·업무 플러그인이라는 서로 다른 실전 문제를 푸는 오픈소스 5개를 골랐다."
date: 2026-10-20 20:00:00 +0900
published: true
image:
  path: /assets/img/posts/2026-10-20-git-trending-ai-weekly-04/cover.jpg
  alt: "GitHub Trending AI 후보 다섯 개의 실제 프로젝트 화면과 소스 구조를 조합한 편집 커버"
---

오늘 GitHub Trending의 AI 저장소를 훑고 처음 든 생각은 “또 에이전트인가”였다. 하지만 README를 열고 릴리스 탭까지 따라가 보니 차이는 꽤 컸다. 어떤 저장소는 프롬프트 한 장이었고, 어떤 저장소는 데스크톱 앱과 모바일 동반 앱까지 내놓았으며, 어떤 저장소는 직무별 플러그인을 수십 개 묶어 배포하고 있었다.

이번 #04는 **2026년 10월 20일 KST에 확인한 GitHub Trending 페이지**를 출발점으로 삼았다. 페이지의 원래 노출 순서를 그대로 옮기지 않았다. 이전 #01~#03에서 다룬 저장소와 최근 단독 소개 글의 Ponytail·Headroom은 제외하고, 설치 경로가 공개되어 있고 라이선스를 확인할 수 있으며 당장 설명 가능한 작업 경계가 있는 프로젝트를 다시 순위화했다.

별점은 이 글을 준비한 시점의 스냅샷이다. 내일이면 바뀐다. 더 중요한 건 “무엇을 대신 해주는가”와 “어디서 멈추는가”다.

## 2026-10-20 실용성 기준 TOP 5

| 순위 | 저장소 | 스냅샷 | 라이선스 | 릴리스 상태 | 고른 이유 |
| ---: | --- | ---: | --- | --- | --- |
| 1위 | [cloudflare/security-audit-skill](https://github.com/cloudflare/security-audit-skill) | 33,314 stars / 2,084 forks | MIT | GitHub Release 없음 | 보안 감사를 “취약점처럼 보이는 것”이 아니라 증거 원장과 검증 단계로 나눈다 |
| 2위 | [Graphify-Labs/graphify](https://github.com/Graphify-Labs/graphify) | 129,698 / 12,493 | Apache-2.0 | v0.9.92, 2026-10-16 | 코드베이스를 탐색할 때 반복되는 관계 추적을 지속 가능한 그래프로 남긴다 |
| 3위 | [stablyai/orca](https://github.com/stablyai/orca) | 99,212 / 6,323 | MIT | v1.4.232, 2026-10-20 | 여러 CLI 에이전트를 한 데스크톱 작업면에서 감시하고 조종한다 |
| 4위 | [MadsLorentzen/ai-job-search](https://github.com/MadsLorentzen/ai-job-search) | 59,450 / 18,633 | MIT | v1.7.3, 2026-10-19 | 검색부터 지원서 PDF 검증까지 구직 흐름을 파일과 명령으로 드러낸다 |
| 5위 | [anthropics/knowledge-work-plugins](https://github.com/anthropics/knowledge-work-plugins) | 30,635 / 3,458 | Apache-2.0 | GitHub Release 없음 | 범용 챗봇을 영업·법무·재무·데이터 업무 묶음으로 바꾸는 공식 예시다 |

> **선정 기준**: Trending 노출 여부 → 기존 시리즈 중복 제거 → 공개 라이선스 → 재현 가능한 설치 경로 → 실제 업무 경계 → 릴리스·최근 푸시 확인 순서로 걸렀다. 이 표는 GitHub의 공식 인기 순위가 아니라 내 편집 순위다.

## 1위. Cloudflare Security Audit Skill

```text
저장소  cloudflare/security-audit-skill
형태    Claude Code 등 에이전트용 보안 감사 Skill
언어    JavaScript
현재성  2026-09-14까지 push, 별도 GitHub Release 없음
```

![Cloudflare Security Audit Skill의 실제 SKILL.md와 감사 단계 구조](/assets/img/posts/2026-10-20-git-trending-ai-weekly-04/01-cloudflare-security-audit-skill.png)

보안 리뷰를 LLM에게 시키면 흔히 두 극단으로 흐른다. “SQL 인젝션을 조심하세요” 같은 교과서 요약이 나오거나, 코드 문맥을 건너뛴 채 위험도를 과장한다. Cloudflare의 스킬은 그 사이에 **coverage ledger, candidate validation, evidence, reporting**이라는 작업 구조를 둔다. 웹 프로토콜·클라이언트·공급망·클라우드·RPC·자원 고갈·데이터 격리처럼 헌팅 클래스를 나눈 것도 구체적이다.

```bash
npx skills add https://github.com/cloudflare/security-audit-skill \
  --skill security-audit
```

README가 보장하는 것은 감사 절차와 설치 경로다. 이 스킬을 설치했다고 코드가 안전해지는 것은 아니다. 릴리스 태그가 없으므로 재현성이 중요한 팀은 설치 시점의 커밋 SHA를 따로 고정해야 한다.

**도입 전 → 후**: 막연한 “전체 보안 검토” 프롬프트에서, 공격면별 조사와 후보 검증·보고가 분리된 감사 절차로 바뀐다. 발견률 향상 수치는 저장소가 제시하지 않으므로 여기서 만들지 않는다.

**프로덕션 유즈케이스**

1. 인증·캐시·요청 파싱이 얽힌 엣지 서비스의 출시 전 수동 코드 리뷰 체크리스트를 만들 때.
2. 플러그인·업데이트·서명 경로가 있는 데스크톱 앱에서 공급망 위협을 빠뜨리지 않도록 감사 범위를 고정할 때.

**한 줄 평**: 결과를 대신 믿게 해주는 스킬이 아니라, 무엇을 확인했는지 남기게 만드는 스킬이다. 그 차이가 꽤 크다.

## 2위. Graphify

```text
저장소  Graphify-Labs/graphify
형태    로컬 코드 지식 그래프 + 에이전트 Skill
언어    Python
릴리스  v0.9.92 (2026-10-16)
```

![Graphify 공식 저장소의 실제 코드 그래프 히어로 화면](/assets/img/posts/2026-10-20-git-trending-ai-weekly-04/02-graphify-graph-hero.png)

Graphify는 저장소를 읽어 함수·클래스·호출 관계를 그래프로 만들고, 결과를 프로젝트에 지속 가능한 형태로 남긴다. 핵심은 “예쁜 그래프”보다 **다음 에이전트 세션도 같은 구조 지식을 재사용한다**는 점이다. README는 그래프에 `graph.schema_version`과 `graph.graphify_version`을 기록한다고 명시한다. 포맷 호환성을 감으로 처리하지 않겠다는 태도다.

```bash
uv tool install graphifyy
graphify install --project

# 에이전트 안에서
/graphify .
```

PyPI 이름은 `graphifyy`로 y가 두 개다. README도 비공식 동명 패키지를 주의하라고 경고한다. Python 3.10+와 uv 또는 pipx가 필요하며, 생성된 그래프가 코드의 의미를 완전히 이해했다는 보장은 없다.

**도입 전 → 후**: 매 세션마다 `rg`와 파일 열기를 반복하던 탐색에서, 버전이 기록된 관계 그래프를 먼저 조회하는 흐름으로 바뀐다. 시간 절감 수치는 공식 자료에 없다.

**프로덕션 유즈케이스**

1. 진입점이 여럿인 레거시 서비스에서 “이 함수 변경이 어느 호출자까지 번지는가”를 리뷰 전에 좁힐 때.
2. 새 팀원이 모노레포의 패키지 경계와 의존 관계를 읽기 위한 온보딩 지도를 만들 때.

**한 줄 평**: 코드 검색을 없애지는 않는다. 대신 같은 길을 매번 처음부터 걷는 일을 줄인다.

## 3위. Orca

```text
저장소  stablyai/orca
형태    멀티 에이전트 데스크톱 IDE
언어    TypeScript
릴리스  v1.4.232 (2026-10-20)
```

![Orca 공식 README의 실제 멀티 에이전트 작업 화면](/assets/img/posts/2026-10-20-git-trending-ai-weekly-04/03-orca-readme-hero.jpg)

Orca는 또 하나의 코딩 모델이 아니다. Claude Code, Codex, Gemini CLI 같은 기존 CLI 에이전트를 탭과 분할 화면에 올리고, 작업 상태를 한곳에서 보는 데스크톱 셸에 가깝다. 공식 저장소에는 split view, diff annotation, remote server, 모바일 동반 앱을 보여주는 실제 GIF와 스크린샷이 함께 있다.

Windows는 공식 릴리스의 setup EXE, Linux는 AppImage, macOS는 DMG 또는 Homebrew cask를 제공한다. 내가 확인한 최신 릴리스는 같은 날 나온 v1.4.232다.

**도입 전 → 후**: 여러 터미널 창에서 에이전트 상태를 눈으로 찾던 흐름에서, 워크스페이스별 탭·분할 화면·모바일 상태 확인으로 바뀐다. 성능이나 생산성 수치는 README에서 검증되지 않았다.

**프로덕션 유즈케이스**

1. 프론트엔드·백엔드·문서 에이전트를 분리해 돌리되 각 세션의 diff와 중단 상태를 한 화면에서 확인할 때.
2. 원격 개발 머신에서 돌아가는 장시간 작업을 휴대폰으로 확인하고 필요한 순간에만 개입할 때.

**한 줄 평**: 에이전트가 많아질수록 모델보다 관제 화면이 먼저 필요해진다는 걸 솔직하게 인정한 도구다.

## 4위. AI Job Search

```text
저장소  MadsLorentzen/ai-job-search
형태    파일 기반 구직·지원서 워크플로
언어    Python
릴리스  v1.7.3 (2026-10-19)
```

![AI Job Search 공식 저장소의 Pip 마스코트 애니메이션](/assets/img/posts/2026-10-20-git-trending-ai-weekly-04/04-ai-job-search-pip.gif)

이름만 보면 “AI가 일자리를 찾아준다”는 얇은 래퍼 같지만, 저장소 구조는 의외로 집요하다. 채용 포털별 검색 Skill, 프로필, 평가 기준, CV·커버레터 템플릿, PDF 레이아웃 검사, 숨은 프롬프트 인젝션을 걷어내는 posting guard가 파일로 분리되어 있다.

```bash
git clone https://github.com/MadsLorentzen/ai-job-search.git
cd ai-job-search
python tools/install_portal_clis.py
claude
# Claude Code 안에서 /setup
```

Python 3.10+, Bun, LaTeX 배포판이 필요하다. 포털 약관과 robots.txt는 국가와 서비스마다 다르다. 저장소도 타 포크의 포털 Skill을 복사하기 전에 네트워크 호출과 lifecycle script를 직접 읽고 오프라인 테스트하라고 권한다.

**도입 전 → 후**: 브라우저 탭과 임시 문서에 흩어졌던 공고·평가·지원서 생성이 버전 관리 가능한 프로필과 명령 묶음으로 모인다. 취업 성공률을 보장하거나 제시하지 않는다.

**프로덕션 유즈케이스**

1. 여러 채용 포털의 공고를 같은 평가 기준으로 비교하고, 탈락 이유와 보류 이유를 남길 때.
2. 역할별 CV와 커버레터를 생성한 뒤 ATS 파싱 가능성과 PDF 레이아웃을 제출 전에 점검할 때.

**한 줄 평**: 지원 버튼 자동 클릭보다 “왜 이 공고에 지원하는가”를 파일로 남기는 쪽을 택했다. 나는 그게 더 유용하다고 본다.

## 5위. Anthropic Knowledge Work Plugins

```text
저장소  anthropics/knowledge-work-plugins
형태    직무별 Claude 플러그인·Skill 모음
언어    Python 중심
현재성  2026-10-20까지 push, 별도 GitHub Release 없음
```

![Anthropic Knowledge Work Plugins의 실제 디렉터리와 Skill 구성](/assets/img/posts/2026-10-20-git-trending-ai-weekly-04/05-anthropic-plugin-map.png)

Anthropic이 공개한 이 저장소는 “업무용 AI”를 가장 덜 추상적으로 보여준다. 영업, 고객 지원, 재무, 법무, HR, 데이터, 디자인, 엔지니어링 폴더마다 `.claude-plugin/plugin.json`과 여러 `skills/*/SKILL.md`가 들어 있다. 범용 프롬프트를 길게 쓰는 대신 직무별 반복 작업을 플러그인 단위로 포장한 셈이다.

```bash
claude plugin marketplace add anthropics/knowledge-work-plugins
claude plugin install sales@knowledge-work-plugins
```

설치 후 관련 Skill은 문맥에 따라 로드되고, `/sales:call-prep`나 `/data:write-query` 같은 명령을 쓸 수 있다. 단, 공식 예시라는 이유만으로 조직 정책·데이터 접근권한·검토 책임이 해결되는 건 아니다. 릴리스 태그가 없으므로 운영 도입은 커밋 고정과 내부 검토가 먼저다.

**도입 전 → 후**: 긴 범용 시스템 프롬프트에서, 직무별 명령·Skill·플러그인 메타데이터가 분리된 배포 단위로 바뀐다. 업무 정확도 향상 수치는 제시되지 않았다.

**프로덕션 유즈케이스**

1. 고객지원 팀의 조사·답변 초안·에스컬레이션 절차를 서로 다른 Skill로 나눠 검토 가능하게 만들 때.
2. 데이터 팀의 탐색·SQL 작성·검증·시각화를 명령별로 분리하고 팀 저장소에서 버전 관리할 때.

**한 줄 평**: “AI를 회사에 도입한다”는 거대한 문장을, 누가 어떤 명령을 언제 쓰는지로 쪼갠 참고 구현이다.

## 이번 주의 결론은 모델이 아니라 포장 단위다

다섯 저장소의 공통점은 더 똑똑한 모델을 만들었다는 데 있지 않다. 기존 모델이 **감사 절차, 코드 그래프, 작업 관제, 구직 파일, 직무 플러그인**이라는 경계 안에서 일하도록 포장했다.

그래서 선택 기준도 단순하다.

- 보안 검토의 누락을 줄이고 싶다면 Cloudflare Skill부터 읽는다.
- 큰 코드베이스에서 같은 탐색을 반복한다면 Graphify를 시험한다.
- 에이전트 세션이 이미 여러 개라면 Orca가 맞는지 본다.
- 구직 자료가 흩어져 있다면 AI Job Search의 파일 모델만이라도 참고한다.
- 팀 업무를 Skill로 나누려면 Anthropic 저장소의 폴더 구조부터 복제하지 말고 읽는다.

Trending은 발견 도구다. 도입 승인서가 아니다. 설치 명령을 복사하기 전에 README, 라이선스, 권한, 릴리스 태그, 네트워크 호출을 확인해야 한다. 이번 주에는 별보다 그 다섯 줄이 더 오래 남았다.

## 확인한 1차 출처

- [GitHub Trending](https://github.com/trending) — 2026-10-20 KST 노출 후보 확인
- [Cloudflare Security Audit Skill README](https://github.com/cloudflare/security-audit-skill)
- [Graphify README 및 Releases](https://github.com/Graphify-Labs/graphify)
- [Orca README 및 Releases](https://github.com/stablyai/orca)
- [AI Job Search README 및 Releases](https://github.com/MadsLorentzen/ai-job-search)
- [Anthropic Knowledge Work Plugins README](https://github.com/anthropics/knowledge-work-plugins)

_메타데이터 스냅샷: 2026-10-20 KST. 별·포크·릴리스 정보는 이후 달라질 수 있다. 이 글에서는 프로젝트를 직접 설치·실행했다고 주장하지 않았으며, 공개 README·소스 트리·릴리스 API 범위만 검증했다._
