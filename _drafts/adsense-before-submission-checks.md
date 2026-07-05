---
title: "애드센스 신청 직전엔 글보다 점검이 먼저였다"
categories:
  - Project AI Autoblog
  - Dev Log
tags:
  - AI
  - Autoblog
  - Smart Suburbanite
  - Google AdSense
  - Search Console
  - QA
  - Dev Log
description: "The Smart Suburbanite가 10개 글을 채운 뒤 Google AdSense 신청으로 바로 넘어가지 않고 public polish, Search Console verification, sitemap fallback, 최종 pre-submission checklist를 먼저 거친 운영 기록."
published: false
---

# 애드센스 신청 직전엔 글보다 점검이 먼저였다

## 10개를 채우면 바로 신청해도 될 줄 알았다

The Smart Suburbanite가 공개 글 10개를 채웠다. 처음 계획만 보면 다음 순서는 단순해 보였다. 글을 10개까지 만들고, 공개 상태를 확인하고, Google AdSense 신청 화면으로 넘어가면 될 것 같았다.

그런데 막상 10개를 채우고 보니 다음 질문은 글 개수가 아니었다. 이 사이트가 외부 심사를 받을 준비가 되었는지, 공개 페이지들이 현재 상태를 제대로 설명하는지, 사용자가 보는 카드와 메타 설명이 본문만큼 정리되어 있는지부터 봐야 했다.

그래서 신청 버튼으로 바로 가지 않았다. 잠깐 멈추고, public polish pass와 Search Console verification, final AdSense pre-submission checklist, sitemap troubleshooting을 먼저 했다. 이번 기록은 AdSense 신청 방법이 아니라 그 멈춤에 대한 프로젝트 로그다.

## 글보다 먼저 본 것들

10번째 글을 발행한 뒤에는 본문보다 사이트 표면을 더 많이 봤다.

favicon이 정상으로 뜨는지, About과 Contact가 아직 scaffold 느낌을 남기지 않는지, Privacy와 Disclosure가 공개되어 있는지, 각 글의 description과 excerpt가 현재 상태를 제대로 말하는지 확인했다. cover와 alt text도 다시 봤고, 내부 링크가 독자에게 실제로 도움이 되는지, RSS와 sitemap과 robots가 공개 URL에서 열리는지도 확인했다.

이 과정에서 배운 점은 꽤 현실적이었다. 본문이 괜찮아도 공개 카드가 낡아 있으면 사이트 전체가 덜 준비된 것처럼 보인다. 실제로 Dev Blog의 이전 글에서 published post인데도 description 쪽에 초안처럼 읽히는 표현이 남아 있었다. 본문만 읽으면 지나칠 수 있지만, 홈 카드나 generated HTML, RSS excerpt에서는 바로 보이는 문제였다.

그 뒤로 public metadata와 excerpt QA는 별도 항목이 되었다. 이 프로젝트에서 "발행했다"는 말은 파일을 옮겼다는 뜻이 아니라, 공개 표면까지 현재 상태와 맞는지 확인했다는 뜻이어야 한다.

## Search Console에서 바로 깔끔하게 끝나지 않았다

Search Console ownership verification은 HTML file upload 방식으로 완료했다. The Smart Suburbanite는 GitHub Pages project site라서 verification file을 공개 경로에 올리고, 실제 URL이 HTTP 200을 반환하는지 확인했다. 사용자가 Search Console UI에서 Verify를 눌렀고, verification은 성공으로 기록되었다.

하지만 sitemap 쪽은 바로 깔끔하게 끝나지 않았다. public URL에서는 `sitemap-index.xml`과 `sitemap-0.xml`이 열렸고 HTTP 200도 확인되었는데, Search Console Sitemaps 화면에서는 `/sitemap-index.xml` submission row가 `Couldn't fetch` / `가져올 수 없음` 상태로 보였다. type은 unknown, discovered pages는 0, last read도 비어 있었다.

이 상태를 실패나 재난처럼 다루지는 않았다. public availability와 Search Console acceptance는 같은 말이 아니라고 분리해서 기록했다. 그리고 fallback discovery aid로 `sitemap.txt`를 추가하고, `robots.txt`에 `sitemap-index.xml`, `rss.xml`, `sitemap.txt` hints를 넣었다. `rss.xml`도 secondary fallback submission 후보로 남겼다.

중요한 건 "이제 인덱싱이 된다"라고 말하지 않는 것이다. 지금 확인한 것은 public URLs가 열리고 fallback paths를 마련했다는 사실이지, crawling이나 indexing이나 ranking이나 traffic이 보장된다는 뜻이 아니다. Search Console은 시간이 걸릴 수 있고, sitemap row가 바로 마음 편한 상태로 바뀌지 않을 수도 있다.

## 자동화가 해주는 일은 글 생성만이 아니었다

이번 단계에서 다시 보인 것은 Project AI Autoblog가 단순히 글을 많이 만드는 프로젝트가 아니라는 점이다.

처음에는 자동화라고 하면 topic을 고르고, outline을 만들고, draft를 쓰고, publish file로 옮기는 장면을 떠올리기 쉽다. 하지만 실제 운영에서 더 자주 막히는 지점은 그 뒤에 있었다.

공개 페이지가 현재 상태를 반영하는가. 메타 설명이 낡지 않았는가. RSS에 글이 들어갔는가. sitemap과 robots가 열리는가. trust pages가 최소한의 역할을 하는가. cover와 alt text가 글과 맞는가. browser freshness가 맞는가. Search Console 상태를 과장하지 않고 기록했는가.

이런 확인이 빠지면 자동화는 글 생성 속도만 높인다. 반대로 이 확인들이 workflow 안에 들어오면 AI는 초안 작성 도구를 넘어 운영 보조 도구가 된다. 이번 pause는 그 방향을 더 분명하게 만들었다.

## 아직 승인받은 것은 아니다

최종 checklist의 결론은 `Ready to submit to AdSense with caution`이었다. 이 문장은 중요하지만, 오해하면 안 된다.

이것은 Google AdSense approval이 아니다. 신청도 아직 하지 않았다. AdSense code도 넣지 않았다. `ads.txt`, analytics, Google Tag Manager, affiliate links, comments, custom domain, Article 011도 추가하지 않았다.

더 정확히 말하면 지금 상태는 "신청해볼 만큼 가까워졌다"에 가깝다. 공개 글 10개, trust pages, Search Console verification, sitemap fallback, public polish, final pre-submission checklist를 거쳤지만, Google의 review 결과를 예측할 수는 없다. 승인, 수익, 트래픽, SEO 성과, ranking, indexing을 보장하는 표현은 전부 피해야 한다.

그래서 이번에는 한 발 더 기다리기로 했다. Search Console sitemap 상태를 조금 더 보고, fallback submission이 어떻게 처리되는지 관찰한 다음, 큰 blocker가 없으면 조심스럽게 다음 단계로 가는 편이 맞다.

## 다음에 바로 할 일

다음 workflow에는 AdSense 신청 직전 Search Console / sitemap 상태를 별도 gate로 넣어야 한다. `sitemap-index.xml`, `sitemap.txt`, `rss.xml` 중 적어도 하나가 받아들여졌는지 보거나, fallback submission 뒤 24-48시간 정도 기다린 다음 진행하는 식이다.

그리고 public metadata / excerpt QA도 더 강하게 넣어야 한다. 본문이 아니라 공개 카드, archive snippet, RSS excerpt, generated HTML에서 틀리는 경우가 있기 때문이다.

이번 멈춤은 속도를 늦춘 일이지만, 프로젝트를 멈춘 일은 아니다. 오히려 반대에 가깝다. 신청 직전에 확인해야 할 것들이 문서와 checklist에 들어가면, 다음 신청은 덜 즉흥적이고 더 재현 가능한 작업이 된다.

## Source note

- Google AdSense, [AdSense eligibility requirements](https://support.google.com/adsense/answer/9724): AdSense participation depends on owned content and policy compliance; this draft does not claim approval.
- Google AdSense, [Make sure your site's pages are ready for AdSense](https://support.google.com/adsense/answer/7299563): page readiness includes useful content and working navigation, so public QA belongs before submission.
- Google Search Console Help, [Sitemaps report](https://support.google.com/webmasters/answer/7451001): sitemap report processing can differ from public URL availability, so the current sitemap issue is recorded cautiously.
