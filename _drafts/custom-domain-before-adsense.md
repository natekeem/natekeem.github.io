---
title: "애드센스 신청 버튼 앞에서 결국 도메인을 붙였다"
categories:
  - Project AI Autoblog
  - Dev Log
tags:
  - AI
  - Autoblog
  - Smart Suburbanite
  - Google AdSense
  - Search Console
  - GitHub Pages
  - Dev Log
description: "The Smart Suburbanite가 Google AdSense 신청 직전 GitHub Pages project subpath 한계 때문에 smartsuburbanite.com을 붙이고, canonical, sitemap, Search Console, AdSense 흐름을 다시 정리한 프로젝트 로그."
published: false
---

# 애드센스 신청 버튼 앞에서 결국 도메인을 붙였다

## 도메인은 나중 문제라고 생각했다

처음에는 GitHub Pages project URL만으로도 충분할 거라고 생각했다. The Smart Suburbanite는 이미 공개 글 10개를 채웠고, About, Contact, Privacy, Disclosure 같은 trust page도 있었다. RSS와 sitemap도 열렸고, Search Console verification과 sitemap fallback 작업도 해 둔 상태였다.

그때 기준으로는 `https://natekeem.github.io/the-smart-suburbanite/`가 proof blog의 공개 주소였다. Project AI Autoblog 입장에서는 아직 수익화가 아니라 workflow 검증이 먼저였고, GitHub Pages project subpath는 MVP 단계에서 꽤 실용적인 선택처럼 보였다.

그래서 당연히 다음 질문은 글 품질이나 checklist 쪽일 줄 알았다. 글이 충분한가, 공개 페이지가 어색하지 않은가, sitemap이 보이는가, Search Console이 확인되는가. 그런데 실제 병목은 조금 다른 곳에서 나왔다.

## 애드센스 입력창에서 막혔다

Google AdSense Add site 화면에 `https://natekeem.github.io/the-smart-suburbanite/`를 넣었을 때, UI는 이 project subpath를 The Smart Suburbanite의 site URL로 받아들이지 않았다. 대신 `http://natekeem.github.io`처럼 domain-level URL에 가까운 값을 제안했다.

이건 content rejection이 아니었다. policy rejection도 아니었다. 글 10개가 부족하다는 판단도 아니었고, Smart Suburbanite라는 niche가 거절된 것도 아니었다. 지금 확인한 범위에서는 site URL structure blocker였다.

`natekeem.github.io`를 그대로 넣는 선택지도 생각할 수는 있었다. 하지만 그 주소는 Dev Blog root site와 섞인다. 그러면 site identity, AdSense review scope, Search Console property, `ads.txt`, trust page 기준이 모두 흐려진다. The Smart Suburbanite를 별도 monetized brand blog로 검증하려는 프로젝트라면, 그 흐림은 작지 않은 문제였다.

## 그래서 smartsuburbanite.com을 붙였다

결국 Route 53에서 `smartsuburbanite.com`을 등록하고, GitHub Pages custom domain으로 연결했다. 이 작업은 단순히 주소 하나를 예쁘게 바꾸는 일이 아니었다.

AstroPaper 쪽에서는 기존 GitHub Pages project base를 제거하고, canonical URL을 custom domain 기준으로 바꾸고, internal link와 RSS, sitemap, robots, cover URL이 모두 `https://smartsuburbanite.com/` 기준으로 맞는지 다시 봐야 했다. `www.smartsuburbanite.com`은 apex로 redirect되게 했고, 예전 GitHub Pages project URL도 apex로 redirect되는지 확인했다.

겉으로 보면 "도메인을 붙였다" 한 줄이지만, 실제로는 site identity를 다시 정렬한 작업에 가까웠다. Google AdSense에 제출하려는 site와 public canonical, sitemap, Search Console, `ads.txt` root가 같은 방향을 바라보게 만드는 일이었다.

## 검색 결과는 바로 바뀌지 않았다

custom domain으로 옮긴 뒤 public output은 정리됐다. home, posts, archives, RSS, sitemap, robots, trust pages, 10개 article URL, 10개 cover URL이 custom domain에서 열렸고, canonical도 `https://smartsuburbanite.com/` 기준으로 잡혔다. Search Console custom-domain property도 등록했고, `sitemap.txt`, `sitemap-index.xml`, `rss.xml` submissions는 Success로 기록됐다.

하지만 Google Search 결과 화면은 바로 바뀌지 않았다. 여전히 예전 GitHub Pages result처럼 보이는 표시나 어색한 `GitHub Pages documentation` labeling이 남아 있을 수 있었다. 그래서 이 문제를 content quality failure처럼 다루지 않기로 했다. 더 정확히는 recrawl과 canonical transition, search appearance display가 업데이트되는 데 시간이 필요한 문제로 기록했다.

그다음에는 home page의 title, description, canonical, `og:site_name`이 맞는지 다시 확인했고, WebSite JSON-LD도 추가했다. 이것도 "검색 결과가 고쳐졌다"는 뜻은 아니다. site name과 canonical signal을 더 분명하게 만든 뒤, Search Console에서 URL Inspection, live test, indexing request를 진행할 수 있게 해 둔 것이다.

## 아직 승인받은 것은 아니다

현재 Google AdSense 쪽에서는 `smartsuburbanite.com` site add/registration은 완료된 상태로 기록되어 있다. 하지만 AdSense approval은 아니다. 지금은 site ad serving eligibility review in progress 상태다.

display ad code도 넣지 않았다. analytics도 넣지 않았다. Google Tag Manager도 없다. affiliate links나 comments도 붙이지 않았다. Article 011도 만들지 않았다. `ads.txt`는 별도 준비 단계로 이미 root에 두었지만, 그것도 approval이나 ad serving success를 뜻하지 않는다.

이 부분을 계속 분리해서 써야 한다. "신청 준비를 했다", "site add를 했다", "`ads.txt`를 준비했다", "review 중이다", "approved다"는 모두 다른 상태다. 이 프로젝트에서 가장 위험한 문장은 실제 상태보다 한 단계 앞서가는 문장이다.

## 이번에 배운 것

이번 일은 글쓰기 workflow의 문제가 아니라 monetized blog infrastructure workflow의 문제였다.

AI가 outline을 만들고, draft를 쓰고, Korean review packet과 claim/fact table을 거치고, final publish checklist를 통과하는 것만으로는 충분하지 않았다. monetized blog를 운영하려면 domain, DNS, GitHub Pages custom domain, canonical, sitemap, robots, Search Console, Google AdSense site input rule, `ads.txt`, search appearance까지 하나의 흐름으로 봐야 했다.

Project AI Autoblog가 만들려는 것은 단순한 article generator가 아니다. 글을 만드는 흐름만 자동화하면, 이런 단계에서 사람이 다시 전체 지도를 펼쳐야 한다. 반대로 domain과 verification, sitemap, canonical, review-state tracking까지 workflow에 들어오면, 다음 proof blog는 조금 덜 흔들릴 수 있다.

자동화의 대상은 "글을 많이 쓰기"가 아니라 "사람이 검토할 수 있는 상태 전이를 반복 가능하게 만들기"에 더 가깝다. 이번 custom domain 전환은 그 사실을 꽤 선명하게 보여줬다.

## 다음부터 바꿀 것

다음 monetized blog를 만들 때는 10개 글 이후에 domain을 고민하지 않는 쪽이 낫다. proof blog 단계에서는 GitHub Pages project subpath도 충분히 실용적이다. 하지만 Google AdSense까지 목표로 하는 site라면 domain-level URL, Search Console property, sitemap, canonical, `ads.txt` root handling을 더 일찍 같이 설계해야 한다.

그리고 Google Search display는 public HTML이 바뀌었다고 바로 따라오지 않을 수 있다. Search Console에서 live URL을 테스트하고 indexing을 요청할 수는 있지만, 검색 결과의 title, source, site name이 언제 어떻게 바뀔지는 보장할 수 없다. 그래서 앞으로도 "준비했다"와 "반영됐다"를 분리해서 기록해야 한다.

이번 글은 승리 기록이 아니다. The Smart Suburbanite가 수익화에 성공했다는 말도 아니다. 다만 글 10개를 채운 뒤에도, 실제 monetization workflow에는 글 바깥의 운영 단계가 많이 남아 있다는 걸 확인한 기록이다.

## Source note

- Google AdSense, [AdSense eligibility requirements](https://support.google.com/adsense/answer/9724): this draft does not claim Google AdSense approval.
- GitHub Docs, [Configuring a custom domain for your GitHub Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site): custom domain setup changes the public site identity and routing surface.
- Google Search Console Help, [Sitemaps report](https://support.google.com/webmasters/answer/7451001): sitemap submission status and public URL availability should be recorded separately.
- Google Search Central, [Influencing your title links in search results](https://developers.google.com/search/docs/appearance/title-link): this draft does not claim search result display has already changed.
