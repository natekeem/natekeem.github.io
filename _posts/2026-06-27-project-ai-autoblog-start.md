---
title: "Project AI Autoblog 시작기: 완전 자동화보다 먼저 HQ 워크스페이스를 만든 이유"
date: 2026-06-27
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
published: true
---

## 시작은 꽤 단순했다

처음 생각은 단순했다. AI를 활용해서 여러 개의 수익형 블로그를 만들고, 글감 발굴부터 초안 작성, SEO 검토, 발행까지 어느 정도 자동화하면 어떨까.

방향도 어느 정도 잡혀 있었다. 영어권 40대 이상 독자를 대상으로 하는 niche blog를 만들고, 각 블로그마다 명확한 주제와 독자 문제를 두는 방식이다. 예를 들면 집 업그레이드, 에너지 효율, 스마트홈, 은퇴 전후 생활, 건강과 장수 같은 주제가 후보였다.

처음에는 자연스럽게 "그럼 자동화 시스템부터 만들면 되겠네"라는 생각으로 흘렀다. 중앙 서버가 있고, 여러 블로그를 관리하고, AI가 초안을 만들고, GitHub에 올리고, 나중에는 analytics까지 모으는 그림이었다.

그런데 막상 시작하려고 보니, 제일 먼저 만들어야 할 것은 서버가 아니었다.

## 바로 자동화하면 위험하겠다고 느꼈다

자동화는 매력적이다. 특히 이런 프로젝트에서는 더 그렇다. 반복 작업이 많아 보이고, AI가 잘 도와줄 수 있는 일도 많다.

하지만 첫날 기준으로는 자동화할 workflow 자체가 아직 검증되지 않았다. 어떤 글이 좋은 글인지, 어떤 주제가 실제로 쓸 만한지, 어떤 검수 단계가 필요한지, AdSense 관점에서 어느 정도의 콘텐츠와 신뢰 요소가 필요한지도 아직 직접 확인하지 않았다.

이 상태에서 orchestrator부터 만들면 위험하다. 잘못된 작업 흐름을 빠르게 반복하는 시스템이 될 수 있기 때문이다.

특히 건강과 금융 주제는 더 조심해야 한다. YMYL 리스크가 있고, 신뢰와 검수 기준이 약한 상태에서 AI 초안을 그대로 발행하면 프로젝트 전체의 방향이 흔들릴 수 있다. 그래서 health나 finance 성격이 강한 블로그들은 일단 뒤로 미뤘다.

지금 단계에서 필요한 것은 "많이 발행하는 시스템"이 아니라 "제대로 판단할 수 있는 기준"이었다.

## 그래서 먼저 HQ 워크스페이스를 만들었다

첫 작업은 `ai-autoblog`라는 root HQ repository를 만드는 것이었다.

이 저장소는 실제 블로그 사이트가 아니다. Jekyll 사이트도 아니고, 중앙 orchestrator도 아니다. 말 그대로 프로젝트의 본부 역할을 하는 공간이다.

여기에는 전략, 결정 기록, prompt, draft workflow, AI agent instruction, checklist를 모아 둔다. 나중에 Dev Blog나 The Smart Suburbanite가 독립 repository로 만들어지더라도, 전체 방향과 운영 기준은 이 HQ에서 관리한다.

이렇게 분리해 두면 좋은 점이 있다. 블로그 하나를 만들다가도 "왜 이 방향으로 가기로 했지?", "자동화는 언제 시작하기로 했지?", "AI가 작성한 글은 어떤 기준으로 검수하지?" 같은 질문에 다시 답할 수 있다.

나 혼자 시작하는 프로젝트일수록 이런 기준 문서가 필요하다. 머릿속에만 있는 기준은 작업 도구를 바꾸거나 며칠 지나면 쉽게 흐려진다.

## 이번에 만든 것들

이번 초기 작업에서 만든 것은 대부분 문서와 운영 구조다.

먼저 `docs/`를 만들었다. 여기에는 PRD, roadmap, decisions, architecture, content policy, SEO strategy, personas, workflow, MVP scope, operating principles, AdSense notes를 넣었다. 그리고 프로젝트 오너가 한국어로 생각하고 운영하기 쉽도록 `docs/`는 한국어 우선으로 정리했다.

`prompts/`에는 반복해서 쓸 수 있는 AI prompt template을 넣었다. 수익형 블로그 글 생성, SEO review, fact-check, image prompt, Codex publish post, Dev Blog 글 생성 같은 흐름을 분리했다.

그리고 `playbooks/`와 `checklists/`를 추가했다. 이 프로젝트에서 말하는 local project skills는 별도 플러그인이 아니라 Markdown 문서다. 반복 작업은 playbook으로 보고, 최종 검수는 checklist로 확인하는 방식이다.

예를 들면 수익형 블로그 글을 쓸 때는 `playbooks/writing_workflow.md`를 보고, 발행 전에는 `checklists/pre_publish_checklist.md`나 `checklists/ai_content_quality_checklist.md`를 확인한다.

## 왜 문서부터 만들었나

솔직히 문서부터 만드는 일은 조금 느려 보인다. 당장 눈에 보이는 사이트 화면도 없고, 글이 발행되는 것도 아니다.

하지만 이 프로젝트에서는 문서가 곧 안전장치다.

처음부터 자동화하면 "AI가 쓴 글을 얼마나 믿을 것인가", "어떤 주제를 피해야 하는가", "어떤 단계에서 사람이 검수해야 하는가" 같은 질문이 뒤로 밀릴 수 있다. 그래서 먼저 판단 기준을 만들었다.

특히 이번 프로젝트의 원칙은 명확하게 잡았다.

- 수동 먼저, 자동화는 나중에
- 품질이 수량보다 우선
- 신뢰와 독창성이 트래픽보다 우선
- AI 생성 콘텐츠는 사람 검수 없이 발행하지 않음
- 건강과 금융은 high-risk category로 다룸

이 기준이 있어야 나중에 자동화를 하더라도 무엇을 자동화하고 무엇을 자동화하지 말아야 하는지 구분할 수 있다.

## Codex, Roo Code, Claude Code를 같이 쓸 준비

이 프로젝트는 하나의 AI 도구만 쓰는 방식이 아니다.

ChatGPT는 전략, 아이디어, outline, draft, editorial review에 주로 쓴다. Codex, Roo Code, Claude Code는 파일 작업, Markdown 배치, local check, git commit, 나중의 publishing task 같은 일에 활용할 계획이다.

그래서 `AGENTS.md`, `CLAUDE.md`, `.roo/rules/00-project-routing.md`를 만들었다.

각 도구가 같은 프로젝트 맥락을 읽고 시작하도록 하기 위해서다. 어떤 도구를 열었는지에 따라 규칙이 달라지면, 작은 프로젝트도 금방 어지러워진다.

특히 지금은 "하지 말아야 할 일"이 중요하다. 아직 Jekyll site를 만들지 않는다. nested git repository를 만들지 않는다. central orchestrator를 만들지 않는다. publishing automation도 만들지 않는다.

이런 제한을 agent routing file에 적어 두면, 다음 작업에서 실수로 너무 앞 단계로 넘어갈 가능성을 줄일 수 있다.

## 수익형 블로그는 The Smart Suburbanite부터

첫 번째 수익형 블로그는 The Smart Suburbanite로 정했다.

대상은 영어권 40대 이상 교외 주택 소유자다. 주제는 smart home, energy efficiency, solar, EV charging, HVAC, home ROI, aging in place 쪽으로 잡았다.

이 주제는 건강이나 금융보다 YMYL 리스크가 낮고, 독자에게 실용적인 도움을 주기 좋다. 예를 들면 heat pump ROI, EV charger installation cost, home energy audit checklist, generator vs home battery comparison 같은 글은 독자가 실제 결정을 내리는 데 도움이 될 수 있다.

물론 여기에도 조심할 점은 있다. 비용, 절감액, tax credit, incentive, installation cost 같은 정보는 지역과 시점에 따라 달라질 수 있다. 그래서 숫자와 최신 정보는 fact-check가 필요하고, 글에는 last reviewed date나 검수 메모가 필요할 수 있다.

## 중앙 서버는 일부러 미뤘다

미래에는 central orchestrator가 필요할 수도 있다.

예상하는 기능은 content queue, AI draft generation, human approval, GitHub publishing, metrics aggregation, cost tracking 정도다. Admin UI와 FastAPI backend, Supabase 같은 구성도 나중에는 검토할 수 있다.

하지만 지금은 아니다.

아직 반복 작업이 충분히 쌓이지 않았고, 어떤 자동화가 정말 시간을 줄여 주는지도 모른다. 지금 서버를 만들면 실제 문제를 해결하기보다 상상 속 workflow를 구현할 가능성이 크다.

그래서 MVP 0.1의 이름을 Manual Publishing Validation으로 잡았다. 먼저 사람이 직접 해 보고, 불편함이 반복될 때 작은 도구부터 추가한다.

## 이미지는 어떻게 쓸 생각인가

Dev Blog에는 나중에 스크린샷을 넣을 수 있다. 예를 들면 VS Code에서 열린 `ai-autoblog` 폴더 구조나 GitHub commit history 같은 화면이다. 아직 실제 Dev Blog site나 GitHub Pages 화면이 없으니, 지금 본문에는 존재하지 않는 이미지 경로를 넣지 않았다.

AI-generated image도 사용할 수는 있다. 다만 글의 내용을 과장하거나 가짜 chart, 가짜 product screenshot, 가짜 endorsement처럼 보이면 안 된다.

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

오늘 배운 것은 생각보다 단순하다.

자동화는 목표가 아니라 결과여야 한다. 반복해서 해 본 일이 있고, 그 반복이 충분히 아프고, 그 과정의 품질 기준이 명확할 때 자동화가 의미가 있다.

반대로 아직 기준이 없을 때 자동화하면, 잘못된 판단을 더 빠르게 반복할 수 있다.

또 하나는 AI agent와 함께 일하려면 문서가 중요하다는 점이다. 사람에게도 문서가 필요하지만, AI agent에게는 더 필요하다. 어떤 파일을 읽어야 하는지, 지금 하지 말아야 할 일이 무엇인지, 언어 정책은 어떤지, commit 전에 무엇을 확인해야 하는지 적어 두면 작업이 훨씬 안정된다.

오늘은 사이트를 만들지 않았다. 글을 발행하지도 않았다. 대신 앞으로 흔들리지 않기 위한 작업대를 만들었다.

## 다음 작업

다음에는 Dev Blog site setup을 시작할 수 있다. 다만 그때도 바로 많은 기능을 넣기보다, Jekyll/Chirpy 기반으로 첫 글을 안전하게 발행할 수 있는 최소 구조부터 확인하는 것이 좋다.

그 다음 단계에서는 The Smart Suburbanite의 첫 topic 후보를 고르고, 실제 영어 draft package를 만들어 볼 수 있다. 그 과정에서 writing workflow, SEO workflow, fact-check workflow가 실제로 도움이 되는지 검증해야 한다.

지금 기준의 목표는 명확하다.

작게 시작하고, 수동으로 검증하고, 기록을 남긴다. 자동화는 그 다음이다.
