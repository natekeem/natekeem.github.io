---
title: "수익형 블로그 글 6개를 발행하며 발행 후 QA의 중요성을 배웠다"
date: 2026-07-01 08:34:40 +0900
categories:
  - Project AI Autoblog
  - Dev Log
tags:
  - AI
  - Autoblog
  - Smart Suburbanite
  - Workflow
  - QA
  - Dev Log
description: "The Smart Suburbanite에 글 6개를 발행한 뒤, AI 초안 작성보다 발행 후 QA와 운영 점검이 더 중요해졌다는 점을 정리한 Dev Blog 기록."
published: true
image:
  path: /assets/img/posts/2026-07-01-smart-suburbanite-six-posts-post-publish-qa/cover.png
  alt: "The Smart Suburbanite 글 6개 발행 후 RSS, 브라우저, 내부 링크, 배포, 출처 확인을 점검하는 post-publish QA 보드"
---

# 수익형 블로그 글 6개를 발행하며 발행 후 QA의 중요성을 배웠다

## 글 6개는 작은 숫자지만 운영 테스트로는 충분했다

The Smart Suburbanite에 공개 글이 6개까지 쌓였다.

아직 이 숫자로 수익, 트래픽, SEO 성과를 말할 단계는 아니다. 오히려 지금 확인한 것은 더 작고 현실적인 것이다. 글을 기획하고, 영어 초안을 만들고, 한국어 검수 패킷으로 확인하고, 발행하고, 발행 뒤 실제 public site에서 다시 보는 흐름이 반복 가능한지였다.

처음에는 "AI가 글을 얼마나 잘 쓰는가"가 중심 질문처럼 보였다. 하지만 여섯 편을 지나오면서 질문이 바뀌었다.

AI가 글을 쓰는 것만으로는 콘텐츠 운영이 되지 않는다. 발행된 뒤 실제 사이트에서 어떻게 보이는지, RSS에 들어갔는지, 오래된 글과 새 글이 서로 연결되는지, 배포가 정말 끝났는지까지 확인해야 한다.

이제는 글쓰기 자체보다 post-publish QA가 더 크게 보이기 시작했다.

## 스마트 온도조절기 글에서 조심해야 했던 것

Article 005와 Article 006은 모두 smart thermostat 관련 글이었다.

- `Do You Really Need a Smart Thermostat?`
- `Smart Thermostat Compatibility: What to Check Before You Buy`

이 주제는 단순한 생활 팁처럼 보이지만, 에너지 비용, HVAC compatibility, 설치 조건과 연결된다. 그래서 완전히 low-risk라고 보기 어려웠다.

특히 Article 006에서는 compatibility를 다루면서 조심해야 할 지점이 많았다. 특정 제품을 추천하거나, 모든 집에 맞는다고 단정하거나, C-wire 설치와 HVAC troubleshooting을 쉽게 안내하는 식의 표현은 피해야 했다.

그래서 공식 자료와 현재 기준을 다시 확인하고, 문장도 더 조심스럽게 잡았다. energy savings, HVAC compatibility, C-wire, heat pump, line-voltage 같은 표현은 독자가 실제 구매나 설치 판단에 사용할 수 있기 때문이다.

"대체로", "확인해야 한다", "전문가에게 문의해야 할 수 있다" 같은 표현은 글을 약하게 만드는 장식이 아니었다. 독자를 잘못된 확신으로 밀어 넣지 않기 위한 안전장치였다.

## 배포가 끝나도 QA는 끝나지 않았다

Article 005 이후에는 AstroPaper에서 browser freshness 문제가 보였다.

GitHub Actions가 성공했고, 공개 URL도 200을 반환했고, RSS에도 글이 들어갔다. 그런데 브라우저 일반 새로고침 화면에서는 최신 글이 바로 보이지 않는 경우가 있었다. hard refresh나 incognito 확인까지 해야 실제 배포 상태를 더 정확히 볼 수 있었다.

service worker나 PWA 설정이 직접 원인이라고 단정할 수는 없었다. 다만 Astro client navigation, view transition, browser stale HTML, cache state 같은 가능성을 따로 기록할 필요는 있었다.

이 경험 때문에 post-publish QA 항목이 늘어났다.

- normal refresh
- hard refresh
- incognito/private window
- home/posts page의 fetched HTML 확인
- RSS 확인
- raw front matter 노출 여부 확인

이건 화려한 자동화 기능이 아니다. 하지만 실제 운영에서는 이런 항목이 더 중요할 때가 많다. 배포가 성공했다고 생각했는데 독자가 보는 화면이 다르면, 글을 잘 썼다는 사실만으로는 충분하지 않다.

이번 Dev Blog 글에서도 같은 기준을 다시 적용했다. 공개 URL에서 raw front matter가 그대로 보이면 GitHub Pages가 Jekyll build 결과가 아니라 source Markdown을 serving하고 있을 가능성을 의심해야 한다. 그래서 home, article, archives, categories를 직접 fetch하고, `---` front matter가 노출되는지 확인했다.

## GitHub 기록과 contributor 표시도 확인했다

Smart Suburbanite repo에서는 GitHub contributor/history cleanup도 조사했다.

AstroPaper 기반으로 scaffold를 만들었기 때문에, public GitHub 화면에서 오래된 template 흔적이나 contributor 표시가 이상하게 보일 수 있는지 확인해야 했다. 조사 결과 remote refs와 contributors API 기준으로는 `natekeem`만 남아 있었고, local tag cleanup으로 `git shortlog --all` 결과도 정리했다.

이 작업은 글 발행 자체와는 거리가 있어 보인다. 하지만 브랜드형 수익형 블로그를 운영하려면 repository history와 public metadata도 독자가 볼 수 있는 외부 신호가 된다. 그래서 콘텐츠 QA와 별개로 운영 QA에 포함될 수밖에 없었다.

## Article 006에서 lockfile 문제가 배포 QA로 들어왔다

Article 006 발행 과정에서는 `pnpm-lock.yaml` mismatch가 GitHub Actions deploy 문제로 드러났다.

이 문제는 글 내용과 무관했다. 하지만 발행 workflow 입장에서는 아주 중요했다. 글을 잘 만들고 검수까지 끝냈더라도, package manager lockfile이 맞지 않으면 public site 업데이트가 막힐 수 있기 때문이다.

그래서 앞으로는 GitHub Actions가 dependency install이나 build 단계에서 실패하면, 무작정 package를 바꾸기 전에 `package.json`과 active lockfile이 맞는지 먼저 확인하기로 했다.

이건 나중에 dashboard를 만들 때도 상태값으로 남아야 할 문제다. "글 준비 완료"와 "사이트 배포 완료"는 같은 상태가 아니다.

## 새 글을 발행하면 오래된 글도 봐야 했다

Article 006 이후에는 기존 Smart Suburbanite 글들의 internal link도 다시 봤다.

새 글 하나를 발행하면 그 글만 확인하면 될 것 같지만, 실제로는 오래된 글에서 새 글로 이어지는 길도 중요했다. 관련 있는 `What to Read Next` 섹션과 본문 주변 링크가 plain text mention으로 남아 있으면 독자는 다음 글로 자연스럽게 이동하기 어렵다.

그래서 older posts를 audit하고, 필요한 곳에는 실제 Markdown link를 추가했다. 다만 internal link를 무조건 많이 넣는 방향은 피했다. 링크는 독자에게 도움이 될 때만 의미가 있다.

여섯 번째 글 이후의 QA는 그래서 새 URL 하나를 보는 일이 아니라, 001부터 005까지의 글이 새 글과 어떻게 연결되는지 다시 보는 일이었다.

## 이제는 AI 글쓰기보다 운영 시스템에 가깝다

여섯 편을 지나오면서 Project AI Autoblog의 방향이 더 분명해졌다.

초기에는 "AI가 글을 써서 블로그를 자동으로 채울 수 있을까"에 가까웠다. 지금은 조금 다르다.

앞으로 필요한 것은 AI-assisted content operations system이다.

그 시스템은 초안만 만드는 것이 아니라, 최소한 다음을 확인해야 한다.

- topic fit
- English draft
- Korean review packet
- claim/fact table
- human approval
- image and alt text
- content risk와 source freshness
- cautious wording
- deployment result
- RSS inclusion
- browser freshness
- raw front matter exposure
- older post internal links
- lockfile/package manager state

자동화는 여전히 목표다. 다만 자동화할 대상이 바뀌었다. 글을 빨리 찍어내는 자동화가 아니라, 사람이 검수할 수 있는 지점과 발행 후 확인해야 할 지점을 놓치지 않게 해 주는 운영 자동화가 먼저다.

이번 6개 글은 성공을 증명했다기보다, 무엇을 확인해야 하는지 더 선명하게 보여 준 단계였다. 그래서 지금은 성과를 크게 말하기보다, post-publish QA를 workflow의 중심으로 끌어올리는 쪽이 맞다.
