---
title: "수익형 블로그 1호에 글 2개를 발행하며 검수 흐름을 먼저 검증했다"
date: 2026-06-29 02:08:36 +0900
categories:
  - Project AI Autoblog
  - Dev Log
tags:
  - AI
  - Autoblog
  - Smart Suburbanite
  - Workflow
  - Human Review
  - Dev Log
description: "The Smart Suburbanite에 첫 두 개의 영어 글을 발행하면서, 자동화보다 먼저 검수 흐름과 배포 문제를 검증한 기록."
published: true
image:
  path: /assets/img/posts/2026-06-29-smart-suburbanite-first-two-posts-wrapup/cover.png
  alt: "첫 두 개의 The Smart Suburbanite 글과 검수 체크리스트, RSS 및 이미지 이슈를 정리한 프로젝트 워크플로 보드"
---

# 수익형 블로그 1호에 글 2개를 발행하며 검수 흐름을 먼저 검증했다

## 이번에 한 일

The Smart Suburbanite 사이트에 첫 두 글을 발행했다.

첫 글은 `What Is a Smart Home? A Plain-English Guide for Homeowners`였다. 스마트홈을 처음 접하는 교외 주택 소유자에게 기본 개념을 설명하는 입문 글이다.

두 번째 글은 `Questions to Ask Before Buying Your First Smart Home Device`였다. 첫 스마트홈 기기를 사기 전에 확인해야 할 질문들을 정리한 buyer-readiness 글이다.

둘 다 영어권 40대 이상 교외 주택 소유자를 대상으로 한 low-risk informational content다. 제품 순위, affiliate 링크, 정확한 가격, 절약률, 보안 보장 같은 위험한 요소는 넣지 않았다.

## 내가 검증하고 싶었던 것

이번 목표는 "AI로 글을 많이 찍어내는 것"이 아니었다.

정말 확인하고 싶었던 것은 AI가 만든 영어 글을 한국어 검수 흐름으로 안전하게 발행할 수 있는지였다. 영어 원문 전체를 매번 처음부터 끝까지 감수하는 방식은 오래가기 어렵다. 대신 사람이 판단해야 하는 지점을 한국어로 압축해서 보고, 위험한 주장과 발행 판단을 따로 확인하는 흐름을 만들고 싶었다.

그래서 글 하나마다 brief, English draft, Korean review packet, claim/fact table, review card, draft article file, cover image, final publish card, publish, post-publish verification 순서로 진행했다.

번거로워 보이지만 이 순서가 꽤 중요했다. 글을 쓰는 일과 발행을 승인하는 일을 분리해야 나중에 Codex, OpenRouter, GPT-5.5, Nemotron 같은 도구를 붙이더라도 어디까지가 초안이고 어디부터가 사람 판단인지 흐려지지 않는다.

## 영어 글을 한국어로 검수하기

내가 원하는 운영 방식은 영어 문장을 한국어로 번역해서 감상하는 것이 아니다.

사람이 봐야 하는 것은 더 구체적이다.

- 주제가 The Smart Suburbanite 포지션에 맞는가
- 독자에게 실제로 도움이 되는가
- 위험한 주장이나 과장이 있는가
- 제품 추천이나 수익화가 너무 앞서 나가지 않는가
- 발행해도 브랜드에 부담이 없는가

Korean review packet은 이 판단을 빠르게 하도록 도와줬다. claim/fact table은 특히 유용했다. "이 문장은 사실 확인이 필요한가", "이 표현은 보장처럼 들리는가", "이 글이 직접 사용 후기처럼 보이는가"를 따로 분리해서 볼 수 있었기 때문이다.

## 실제로 겪은 문제들

운영 문제도 꽤 나왔다.

먼저 `ogImage`는 social preview와 metadata용이지, 본문 cover image가 아니었다. AstroPaper에서 `ogImage`를 넣었다고 article page 상단에 이미지가 보이는 것은 아니었다. 결국 visible cover image는 Markdown image로 본문 맨 위에 넣어야 했다.

브라우저 캐시도 헷갈렸다. cover asset URL은 정상인데 화면에서 바로 안 보이는 순간이 있었다. 그래서 cover URL에 cache-busting query를 붙여 확인했고, 실제 배포 후 article URL과 asset URL을 따로 검증했다.

RSS 문제도 있었다. 처음에는 GitHub Pages base path가 반영되지 않아 RSS가 잘못된 root URL을 가리켰다. `https://natekeem.github.io/the-smart-suburbanite/` 아래에 배포되는 사이트라면 RSS도 그 base path를 알아야 한다.

두 번째 글에서는 `draft: false`로 바꿨는데도 공개되지 않는 문제가 있었다. 원인은 `pubDatetime`이 현재 기준 미래 시간으로 잡혀 있었기 때문이다. AstroPaper의 scheduled post filtering 때문에 draft를 해제해도 미래 글은 숨겨질 수 있었다.

OpenRouter 무료 모델도 검수 보조로는 쓸 만했지만, 긴 파일 수정 workflow에는 daily limit 리스크가 있었다. 리뷰에는 좋지만 발행과 배포처럼 끊기면 곤란한 작업은 더 보수적으로 다뤄야 한다.

마지막으로 `monetized_blogs/smart_suburbanite` 같은 legacy placeholder 폴더도 발견했다. 현재 활성 site repo는 `monetized_blogs/the-smart-suburbanite`이고, 예전 placeholder는 README 하나만 있는 상태였다. 바로 지우지는 않고 cleanup report로 남겼다.

## 자동화보다 먼저 시스템이 필요했다

이번에 배운 결론은 단순하다.

글 생성보다 review gate가 먼저다. 테마마다 front matter, 이미지 경로, RSS, draft 처리 방식이 다르다. 그래서 나중에 중앙 시스템을 만들더라도 arbitrary repo를 아무렇게나 지원하기보다 theme adapter가 필요하다. 지금 기준에서는 AstroPaper를 먼저 제대로 이해하는 쪽이 맞다.

무료 모델은 좋은 보조 인턴처럼 쓸 수 있다. 하지만 발행, 배포, 공개 URL 검증은 더 엄격한 절차가 필요하다. "AI가 알아서 돈 버는 블로그를 만든다"는 식의 이야기는 아직 아니다. 지금 하고 있는 것은 수동 검증을 통해 반복 가능한 운영 흐름을 찾는 일에 가깝다.

## 다음 글감

이 경험에서 이어질 글감은 꽤 많다.

이번 글에서는 모든 문제를 깊게 풀기보다, 첫 두 글을 발행하면서 드러난 운영 포인트를 한 번에 정리해두고 싶었다. 각각의 문제는 이후 글에서 따로 다룰 생각이다.

- 영어 글을 한국어로 검수하는 방법
- 왜 Theme Adapter가 필요한가
- 중앙 제어 사이트를 만든다면 어떤 상태를 추적해야 하는가
- 이미지와 RSS 배포 문제를 어떻게 잡았는가
- OpenRouter 무료 모델을 어디까지 믿을 수 있는가

아직 Search Console, Analytics, AdSense, affiliate, comments는 붙이지 않았다. 지금은 수익화보다 workflow 검증이 먼저다.

The Smart Suburbanite의 첫 두 글은 작은 발행이지만, Project AI Autoblog가 어디를 자동화해야 하고 어디를 사람이 책임져야 하는지 보여준 꽤 좋은 실험이었다.
