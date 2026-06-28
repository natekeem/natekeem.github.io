---
title: "Project AI Autoblog 시작기: 완전 자동화보다 먼저 HQ 워크스페이스를 만든 이유"
date: 2026-06-27 09:00:00 +0900
categories:
  - Project AI Autoblog
  - Dev Log
tags:
  - AI
  - Blogging
  - Automation
  - Codex
  - Jekyll
  - Build in Public
description: "AI 기반 수익형 블로그 자동화 프로젝트를 시작하면서 바로 자동화 서버를 만들지 않고 HQ 워크스페이스와 문서 체계부터 만든 이유를 정리했다."
image:
  path: /assets/img/posts/2026-06-27-project-ai-autoblog-start/cover.png
  alt: "Project AI Autoblog HQ 워크스페이스를 준비하는 개발자 작업 공간"
published: true
---

## 시작은 꽤 단순했다

처음 아이디어는 단순했다.

AI를 활용해 여러 개의 수익형 블로그를 만들고, 글감 발굴부터 초안 작성, SEO 검토, 발행까지 어느 정도 자동화해 보는 것.

대상도 어느 정도 정해져 있었다. 영어권 40대 이상 독자를 대상으로 한 niche blog를 만들고, 각 블로그마다 분명한 독자 문제를 두는 방식이다. 집 업그레이드, 에너지 효율, 스마트홈, 은퇴 전후 생활, 건강과 장수 같은 주제가 후보였다.

처음에는 자연스럽게 이런 그림을 떠올렸다.

```text
여러 블로그
  -> AI 초안 생성
  -> 자동 발행
  -> analytics 수집
  -> 다시 글감 추천
```

그런데 막상 시작하려고 보니, 제일 먼저 만들 것은 서버가 아니었다.

## 바로 자동화하면 위험하겠다고 느꼈다

자동화는 매력적이다. 반복 작업이 많아 보이는 프로젝트에서는 더 그렇다.

하지만 첫날 기준으로는 자동화할 workflow 자체가 아직 검증되지 않았다.

1. 어떤 글이 좋은 글인지
2. 어떤 주제가 실제로 쓸 만한지
3. 어떤 검수 단계가 필요한지
4. AdSense 관점에서 어떤 신뢰 요소가 필요한지

이 질문들에 아직 답하지 못했다.

> 자동화는 반복할 작업이 검증된 뒤에 의미가 있다. 검증되지 않은 판단을 빠르게 반복하는 것은 효율이 아니라 리스크다.

특히 건강과 금융 주제는 더 조심해야 한다. YMYL 리스크가 있고, 검수 기준이 약한 상태에서 AI 초안을 그대로 발행하면 프로젝트 전체의 신뢰가 흔들릴 수 있다.

그래서 health나 finance 성격이 강한 블로그는 뒤로 미뤘다. 지금 필요한 것은 "많이 발행하는 시스템"이 아니라 "제대로 판단할 수 있는 기준"이었다.

## 그래서 먼저 HQ 워크스페이스를 만들었다

첫 작업은 `ai-autoblog`라는 root HQ repository를 만드는 것이었다.

이 저장소는 실제 블로그 사이트가 아니다. Jekyll 사이트도 아니고, 중앙 orchestrator도 아니다. 프로젝트의 본부 역할을 하는 공간이다.

현재 구조는 대략 이렇게 잡았다.

```text
ai-autoblog/
  docs/
  prompts/
  playbooks/
  checklists/
  content_lab/
  dev_blog/
  monetized_blogs/
```

여기에는 전략, 결정 기록, prompt, workflow, AI agent instruction, checklist를 모아 둔다.

나중에 Dev Blog나 The Smart Suburbanite가 독립 repository로 움직이더라도, 전체 방향과 운영 기준은 이 HQ에서 관리한다.

## 바로 만들고 싶었던 것과 실제로 만든 것

처음에는 중앙 서버와 자동 발행 pipeline부터 만들고 싶었다. 하지만 실제로는 문서와 검수 기준을 먼저 만들었다.

| 바로 만들고 싶었던 것 | 실제로 먼저 만든 것 |
|---|---|
| 중앙 서버 | HQ 워크스페이스 |
| 자동 발행 pipeline | 수동 검수 기준 |
| 여러 개의 블로그 | Dev Blog + 첫 수익형 블로그 하나 |
| 수익 대시보드 | 결정 기록과 checklist |

이 결정은 속도를 늦추기 위한 것이 아니다. 오히려 나중에 덜 흔들리기 위한 준비에 가깝다.

## 이번에 만든 것들

초기 작업에서 만든 것은 대부분 문서와 운영 구조다.

`docs/`에는 PRD, roadmap, decisions, architecture, content policy, SEO strategy, personas, workflow, MVP scope, operating principles, AdSense notes를 넣었다.

`prompts/`에는 반복해서 쓸 수 있는 AI prompt template을 넣었다. 수익형 블로그 글 생성, SEO review, fact-check, image prompt, Codex publish post, Dev Blog 글 생성 흐름을 분리했다.

`playbooks/`와 `checklists/`도 추가했다. 이 프로젝트에서 말하는 local project skill은 별도 plugin이 아니라 Markdown 문서다.

- 반복 작업은 `playbooks/`에서 확인한다.
- 최종 검수는 `checklists/`로 확인한다.
- 도구가 바뀌어도 같은 운영 문서를 읽게 한다.

## 왜 문서부터 만들었나

문서부터 만드는 일은 느려 보인다. 당장 눈에 보이는 화면도 없고, 발행된 글도 적다.

하지만 이 프로젝트에서는 문서가 안전장치다.

처음부터 자동화하면 중요한 질문이 뒤로 밀릴 수 있다.

- AI가 쓴 글을 어디까지 믿을 것인가?
- 어떤 주제는 피해야 하는가?
- 어떤 단계에서 사람이 반드시 검수해야 하는가?
- 언제부터 AdSense를 생각할 수 있는가?

그래서 먼저 원칙을 정리했다.

```text
수동 먼저, 자동화는 나중에
품질이 수량보다 우선
신뢰와 독창성이 트래픽보다 우선
AI 생성 콘텐츠는 사람 검수 없이 발행하지 않음
건강과 금융은 high-risk category로 다룸
```

자동화에 대해서도 기준을 나눴다.

```text
좋은 자동화 = 검증된 반복 작업을 줄이는 것
성급한 자동화 = 검증되지 않은 판단을 빠르게 반복하는 것
```

## Codex, Roo Code, Claude Code를 같이 쓸 준비

이 프로젝트는 하나의 AI 도구만 쓰지 않는다.

ChatGPT는 전략, 아이디어, outline, draft, editorial review에 주로 쓴다. Codex, Roo Code, Claude Code는 파일 작업, Markdown 배치, local check, git commit, publishing task에 활용할 계획이다.

그래서 다음 파일을 만들었다.

- `AGENTS.md`
- `CLAUDE.md`
- `.roo/rules/00-project-routing.md`

목적은 단순하다. 어떤 도구를 열어도 같은 프로젝트 맥락을 읽고 시작하게 만드는 것이다.

특히 지금은 "무엇을 할 것인가"보다 "무엇을 아직 하지 않을 것인가"가 중요하다.

- 아직 monetized blog repository를 만들지 않는다.
- 아직 central orchestrator를 만들지 않는다.
- 아직 publishing automation을 만들지 않는다.
- 아직 analytics, AdSense, comments를 붙이지 않는다.

작은 프로젝트도 기준이 없으면 금방 복잡해진다.

## 수익형 블로그는 The Smart Suburbanite부터

첫 번째 수익형 블로그는 The Smart Suburbanite로 정했다.

대상은 영어권 40대 이상 교외 주택 소유자다. 주제는 smart home, energy efficiency, solar, EV charging, HVAC, home ROI, aging in place 쪽으로 잡았다.

이 주제는 건강이나 금융보다 YMYL 리스크가 낮다. 그리고 독자에게 실용적인 도움을 주기 좋다.

예를 들면 이런 글감이 가능하다.

- heat pump ROI for suburban homes
- EV charger installation cost for homeowners
- home energy audit checklist
- generator vs home battery comparison

물론 여기에도 조심할 점은 있다. 비용, 절감액, tax credit, incentive, installation cost 같은 정보는 지역과 시점에 따라 달라질 수 있다.

그래서 숫자와 최신 정보는 fact-check가 필요하다. 필요하면 last reviewed date도 관리해야 한다.

## 중앙 서버는 일부러 미뤘다

미래에는 central orchestrator가 필요할 수도 있다.

예상하는 기능은 content queue, AI draft generation, human approval, GitHub publishing, metrics aggregation, cost tracking 정도다. Admin UI, FastAPI backend, Supabase 같은 구성도 나중에는 검토할 수 있다.

하지만 지금은 아니다.

아직 반복 작업이 충분히 쌓이지 않았다. 어떤 자동화가 실제로 시간을 줄여 주는지도 모른다. 지금 서버를 만들면 실제 문제보다 상상 속 workflow를 구현할 가능성이 크다.

그래서 현재 단계는 MVP 0.1, Manual Publishing Validation이다.

먼저 사람이 직접 해 보고, 불편함이 반복될 때 작은 도구부터 추가한다.

## 이미지는 어떻게 쓸 생각인가

Dev Blog에는 나중에 스크린샷을 넣을 수 있다.

예를 들면 VS Code에서 열린 `ai-autoblog` 폴더 구조나 GitHub commit history 같은 화면이다. 하지만 아직 실제로 넣을 이미지 파일이 없으므로, 본문에는 이미지 경로를 추가하지 않았다.

AI-generated image도 사용할 수는 있다. 다만 가짜 chart, 가짜 product screenshot, 가짜 endorsement처럼 보이면 안 된다.

이미지는 글을 더 명확하게 만드는 역할이어야 한다. 독자를 속이는 장치가 되면 안 된다.

<!--
Image candidates for later:
1. Screenshot: root ai-autoblog folder structure in VS Code
   - alt: "VS Code에서 열린 ai-autoblog HQ 워크스페이스 폴더 구조"
2. Screenshot: GitHub repository commit history
   - alt: "Project AI Autoblog 초기 커밋 기록"
3. Concept image: a clean developer desk with multiple blog/workflow documents
   - prompt: "A clean developer workspace showing markdown documents, blog planning boards, and an automation workflow diagram on a monitor, minimal, realistic, no brand logos, no fake charts"
   - alt: "AI 블로그 자동화 프로젝트를 위한 개발자 워크스페이스"
-->

## 오늘 기준으로 배운 점

오늘 배운 것은 단순하다.

자동화는 목표가 아니라 결과여야 한다. 반복해서 해 본 일이 있고, 그 반복이 충분히 불편하고, 품질 기준이 명확할 때 자동화가 의미 있다.

반대로 기준이 없을 때 자동화하면 잘못된 판단을 더 빠르게 반복할 수 있다.

또 하나는 AI agent와 함께 일하려면 문서가 중요하다는 점이다. 사람에게도 문서가 필요하지만, AI agent에게는 더 필요하다.

읽어야 할 파일, 하지 말아야 할 일, 언어 정책, commit 전 확인 사항을 적어 두면 작업이 훨씬 안정된다.

오늘은 큰 시스템을 만들지 않았다. 대신 앞으로 흔들리지 않기 위한 작업대를 만들었다.

## 다음 작업

다음 작업은 Dev Blog 자체를 조금씩 다듬는 것이다.

1. 첫 글의 가독성을 개선한다.
2. GitHub Actions build status를 확인한다.
3. site URL, sitemap, robots.txt를 확인한다.
4. Search Console, analytics, comments는 계정과 방향이 정해진 뒤 붙인다.

그 다음에는 The Smart Suburbanite의 첫 topic 후보를 고르고, 실제 영어 draft package를 만들어 볼 수 있다.

지금 기준의 목표는 명확하다.

작게 시작하고, 수동으로 검증하고, 기록을 남긴다. 자동화는 그 다음이다.
