---
title: "?섏씡??釉붾줈洹?湲 4媛쒕? 諛쒗뻾?섎ŉ 諛곗슫 寃? AI 湲?곌린蹂대떎 QA 泥댄겕由ъ뒪?멸? 癒쇱????"
categories:
  - Project AI Autoblog
  - Dev Log
tags:
  - AI
  - Autoblog
  - Smart Suburbanite
  - Workflow
  - Human Review
  - QA
  - Dev Log
description: "The Smart Suburbanite에 공개 글 4개를 발행하면서, AI 글쓰기보다 발행 전후 QA checklist와 사람 승인 구조가 먼저 필요하다는 점을 정리한 Dev Blog 초안."
published: false
---

# ?섏씡??釉붾줈洹?湲 4媛쒕? 諛쒗뻾?섎ŉ 諛곗슫 寃? AI 湲?곌린蹂대떎 QA 泥댄겕由ъ뒪?멸? 癒쇱????

## 시작하며

The Smart Suburbanite에 공개 글 4개를 발행했다.

처음에는 "AI가 글을 잘 쓰게 할 수 있을까"가 가장 큰 질문처럼 보였다. 그런데 실제로 네 편을 반복해서 발행해 보니 더 중요한 질문은 조금 달랐다.

글을 쓰는 것보다 어려웠던 것은 발행 전후에 무엇을 확인하고, 누가 승인하고, 어떤 문제를 다시 workflow에 반영할지 정하는 일이었다.

그래서 이번 기록은 성공담이 아니다. 글 4개를 발행하면서 드러난 작은 문제들을 모아 보고, 왜 AI 글쓰기보다 QA checklist가 먼저 필요했는지 정리하는 글이다.

## 글 4개가 쌓이면서 보인 것

현재 The Smart Suburbanite에 공개된 글은 네 개다.

1. `What Is a Smart Home? A Plain-English Guide for Homeowners`
2. `Questions to Ask Before Buying Your First Smart Home Device`
3. `Smart Home Terms Explained: Matter, Thread, Zigbee, and Z-Wave`
4. `Smart Home Setup Checklist for Non-Technical Homeowners`

이 흐름은 단순히 글 네 편을 채운 것이 아니다.

첫 글은 beginner reader에게 smart home의 기본 개념을 설명했다. 두 번째 글은 첫 기기를 사기 전에 물어봐야 할 질문을 정리했다. 세 번째 글은 Matter, Thread, Zigbee, Z-Wave 같은 용어를 풀었다. 네 번째 글은 실제 setup checklist로 이어졌다.

즉, 독자가 이해에서 구매 전 판단으로, 다시 용어 이해와 설치 준비로 이동하는 흐름을 시험한 셈이다.

아직 traffic, revenue, monetization을 말할 단계는 아니다. AdSense, affiliate link, analytics, Search Console verification, custom domain도 붙이지 않았다. 지금 검증한 것은 돈이 아니라 workflow다.

## 문제는 글쓰기보다 QA였다

반복하면서 드러난 문제는 대부분 "AI가 문장을 못 쓴다"가 아니었다.

오히려 문제는 발행 시스템 주변에서 나왔다.

- `ogImage`는 visible cover가 아니었다.
- RSS는 GitHub Pages base path를 정확히 반영해야 했다.
- `pubDatetime`이 미래 시간이면 `draft: false`여도 글이 숨겨질 수 있었다.
- YAML title에 colon이 있으면 quote가 필요했다.
- Korean review docs에 mojibake가 생기면 검수 문서의 의미가 무너진다.
- The Smart Suburbanite 같은 brand blog에는 개인 GitHub social link가 어색할 수 있었다.
- scaffold copy와 demo 흔적도 QA 대상이었다.

각 문제는 작아 보인다. 하지만 이런 것들이 쌓이면 "글은 있는데 공개 사이트에서는 안 보이는" 상태가 된다. 더 위험한 것은 사람이 발행했다고 생각했는데 RSS나 listing에는 빠져 있거나, cover가 metadata에만 있고 본문에는 보이지 않는 경우다.

그래서 발행 workflow는 글쓰기 prompt보다 더 구체적이어야 한다.

## 그래서 review card와 final publish card가 필요했다

처음에는 글을 만들고, 확인하고, 발행하면 될 것처럼 보였다.

하지만 실제로는 중간 단계마다 멈춤 지점이 필요했다.

`review_card`는 draft를 site repo로 옮겨도 되는지 판단하는 카드다. 여기서는 주제, 독자 가치, 위험 주장, 출처 필요 여부, brand fit을 본다.

`final_publish_card`는 실제 publish 전에 남기는 승인 기록이다. 여기서는 draft flag, publish time, cover, RSS, public URL, no monetization added 같은 항목을 확인한다.

`claim/fact table`은 사실 주장과 위험 주장을 분리하는 장치다. 스마트홈 주제는 건강이나 금융보다 낮은 리스크지만, 안전, 설치, 비용, 절약 같은 표현은 여전히 조심해야 한다.

`publish_checklist`는 사람의 기억을 대신하는 문서다. 한 번 겪은 문제를 다음에도 다시 검색하지 않게 만든다.

이 문서들이 조금 번거로워 보이지만, 지금 단계에서는 바로 그 번거로움이 안전장치다.

## 멀티 Agent가 필요해지는 지점

네 편을 발행하면서 멀티 Agent 아이디어도 더 선명해졌다.

Draft Agent가 자기 글을 스스로 승인하면 위험하다. SEO Agent가 clickbait 제목을 만들고 그대로 통과되면 위험하다. Publisher Agent가 approval 없이 publish할 수 있으면 더 위험하다.

그래서 미래 구조에서는 역할이 나뉘어야 한다.

- Topic Agent는 후보와 search intent를 제안한다.
- Brief Agent는 한국어 brief로 방향을 정리한다.
- Draft Agent는 English draft를 만든다.
- Claim/Risk Agent는 위험 주장과 출처 필요 항목을 잡는다.
- Review Packet Agent는 사람이 빠르게 판단할 수 있는 한국어 검수 패킷을 만든다.
- SEO Agent는 제목, meta, internal link 후보를 제안한다.
- Image Agent는 cover concept과 alt text를 만들되, fake screenshot이나 fake metric을 피한다.
- Publisher Agent는 승인된 것만 repo에 반영한다.
- Monitor Agent는 public URL, RSS, listing을 읽기 전용으로 확인한다.
- Decision Engine은 user setting, risk score, approval state로 다음 상태를 결정한다.

다만 지금 당장 필요한 것은 거대한 agent system이 아니다. 먼저 필요한 것은 역할과 책임 경계를 문서화하는 일이다. 어떤 작업은 AI가 제안하고, 어떤 작업은 사람이 승인하고, 어떤 작업은 시스템이 기계적으로 검증해야 하는지 나누는 것이 먼저다.

## 모든 테마를 지원하지 않기로 한 이유

이번 작업은 theme adapter 전략도 다시 확인시켜 줬다.

Dev Blog는 Chirpy이고, The Smart Suburbanite는 AstroPaper다. 둘 다 Markdown 기반 블로그처럼 보이지만 실제 운영 규칙은 다르다.

다른 점은 꽤 많다.

- post file path
- front matter field names
- date field
- draft handling
- visible cover behavior
- asset path
- build and deploy workflow
- base path
- category and tag schema
- RSS behavior

처음부터 arbitrary Git repo, arbitrary Jekyll theme, arbitrary Astro theme, WordPress까지 지원하려고 하면 제품은 시작하기도 전에 복잡해진다.

그래서 v0/v1 monetized blog 공식 지원 테마는 AstroPaper 하나로 제한하는 것이 맞다. 중앙 시스템은 title, slug, description, status, tags, cover image, body markdown 같은 normalized content model을 갖고, Theme Adapter가 실제 repo 구조로 변환해야 한다.

이건 확장성을 포기하는 결정이 아니다. 오히려 아무 theme나 지원한다고 말하지 않기 위한 정직한 제한이다.

## 공통 부분과 테마별 차이

정리하면 중앙 workflow가 알아야 할 공통 부분은 이렇다.

- title
- slug
- description
- draft/publish status
- tags/categories
- cover image concept
- SEO checklist
- RSS/public URL QA

반대로 adapter가 책임져야 할 테마별 차이는 이렇다.

- post path
- front matter field names
- date field name
- draft and scheduled post behavior
- visible cover rendering
- asset path
- base path
- build command
- deploy workflow
- RSS output

이 구분이 없으면 core workflow가 `_posts/`와 `src/content/`와 `ogImage`와 `image.path` 같은 세부 구현을 계속 직접 알아야 한다. 지금은 수동으로 할 수 있지만, 나중에 dashboard로 만들면 유지보수가 어려워진다.

## 지금 이 프로젝트가 향하는 방향

Project AI Autoblog는 단순한 AI autoblog generator가 아니다.

현재 방향은 이렇게 보고 있다.

```text
Dev Blog -> Proof Blog -> Standardized Workflow -> Control Dashboard -> Subscription Product
```

Dev Blog는 판단과 실패를 기록하는 곳이다. Proof Blog는 The Smart Suburbanite처럼 실제 운영 workflow를 검증하는 곳이다. Standardized Workflow는 반복되는 작업을 문서와 checklist로 굳히는 단계다. Control Dashboard는 그다음에야 만들 수 있다.

지금은 아직 Proof Blog 단계다. 글 4개를 공개했지만, 시스템이 완성된 것은 아니다. monetization도 시작하지 않았다. 성과 숫자도 없다.

그래도 얻은 것은 있다. AI가 글을 쓰는 것보다, 글이 안전하게 검수되고, 정확한 파일 구조로 들어가고, 공개 사이트에서 실제로 보이는지 확인하는 일이 먼저라는 점이다.

## 다음에는

다음 작업은 글을 무작정 늘리는 것이 아니다.

The Smart Suburbanite는 5개 이상 글로 확장하되, 같은 QA checklist와 운영 명령을 계속 다듬어야 한다. Dev Blog에서는 실패, 수정, 운영 판단을 계속 기록해야 한다.

그리고 언젠가 중앙 dashboard를 만들 때는 이번에 겪은 문제들이 버튼과 상태값으로 바뀌어야 한다.

- Draft created
- Review packet ready
- Needs source check
- Cover ready
- Final publish approved
- Published
- Post-publish QA passed

지금은 Markdown과 Git으로 이 상태를 남기고 있다. 나중에는 이 상태가 DB와 UI로 옮겨갈 것이다.

그 전까지는 계속 수동으로 확인한다. 자동화보다 먼저, 사람이 믿을 수 있는 checklist가 필요하다.
