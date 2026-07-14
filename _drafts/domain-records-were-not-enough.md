---
title: "도메인 레코드를 다 넣었는데도 블로그가 안 열린 이유"
categories:
  - Project AI Autoblog
  - Dev Log
tags:
  - DNS
  - Route 53
  - GitHub Pages
  - Search Console
  - Custom Domain
  - Dev Log
description: "나태킴의 커밋로그를 natekeem.com으로 옮기면서 DNS 레코드보다 먼저 확인했어야 할 네임서버 위임과 GitHub Pages custom domain 적용 순서를 기록한 프로젝트 로그."
published: false
---

# 도메인 레코드를 다 넣었는데도 블로그가 안 열린 이유

## 새 주소도, 원래 주소도 열리지 않았다

`natekeem.com`을 브라우저에 입력했는데 사이트가 열리지 않았다. 잠시 뒤에는 원래 잘 열리던 `natekeem.github.io`도 새 주소로 이동한 다음 멈췄다. 주소 하나를 바꾸려다 Dev Blog의 공개 경로를 둘 다 막은 셈이었다.

그날 하려던 일은 `나태킴의 커밋로그`를 `natekeem.github.io`에서 `natekeem.com`으로 옮기는 것이었다. 새 주소를 canonical domain으로 쓰고, Search Console과 sitemap을 그 주소에 맞춘 뒤, 나중에 Google AdSense 준비 상태를 별도로 검토할 기반을 만들고 싶었다.

Route 53 hosted zone에서 본 DNS 레코드는 문제가 없어 보였다.

- apex에는 GitHub Pages용 A 레코드가 들어 있었다.
- `www`에는 `natekeem.github.io`를 가리키는 CNAME이 들어 있었다.

나는 레코드를 다 넣었으니 이제 기다리면 된다고 생각했다. 그런데 public DNS에서 `natekeem.com`을 조회한 결과는 `NXDOMAIN`이었다. A 레코드의 값이 틀린 것이 아니었다. GitHub Pages가 실패한 것도 아니었다. 도메인이 내가 보고 있던 Route 53 hosted zone을 사용하도록 연결되지 않은 것이 문제였다.

## 빠뜨린 것은 네임서버 위임이었다

먼저 확인했어야 할 것은 도메인의 네임서버였다. 도메인이 Route 53 hosted zone에 지정된 네임서버로 위임되지 않으면, 그 hosted zone 안에 A 레코드와 CNAME을 정확히 넣어도 public DNS는 그 내용을 보지 않는다.

쉽게 말하면 주소록 내용은 작성했지만, 사람들이 어느 주소록을 찾아가야 하는지는 연결하지 않은 상태였다. 나는 레코드 표만 확인했고, 도메인의 네임서버가 기대한 hosted zone을 가리키는지는 뒤늦게 봤다.

이것은 모든 DNS 환경에 그대로 적용할 수 있는 안내가 아니다. 등록기관과 DNS 제공자 구성은 다를 수 있다. 다만 이번 `natekeem.com` 전환에서는 레코드보다 앞단의 네임서버 위임이 빠져 있었고, 그것이 `NXDOMAIN`의 원인이었다.

## GitHub Pages custom domain을 너무 일찍 넣었다

여기에 내가 순서를 한 번 더 잘못 잡았다. public DNS가 정상인지 확인하기 전에 GitHub Pages에 `natekeem.com` custom domain을 먼저 추가했다.

그러자 기존 주소인 `natekeem.github.io`가 `natekeem.com`으로 redirect되기 시작했다. 그런데 새 도메인은 아직 public DNS에서 해석되지 않았다. 결과적으로 정상적으로 열리던 기존 주소까지, 열리지 않는 새 주소로 방문자를 보내는 상태가 됐다.

GitHub Pages가 장애를 만든 것은 아니다. custom domain을 적용하면 기존 GitHub Pages 주소가 새 canonical domain으로 이동할 수 있다는 점을 고려했어야 했다. 새 도메인이 실제로 해석되는지 확인하지 않은 채 redirect가 시작되도록 한 내 순서가 문제였다.

우선 사이트를 복구하기 위해 GitHub Pages custom domain을 제거했다. 그러자 `natekeem.github.io`에서 Dev Blog가 다시 열렸다. 새 도메인 준비가 끝날 때까지 기존 공개 경로를 살려 둔 임시 복구였다.

## 기존 주소를 먼저 살리고 순서대로 다시 했다

복구는 새 도메인을 억지로 살리는 것보다 기존 공개 경로를 되돌리는 데서 시작했다.

1. GitHub Pages에서 custom domain을 잠시 제거했다.
2. `natekeem.github.io`에서 Dev Blog가 다시 열리는지 확인했다.
3. 도메인의 네임서버를 기대한 Route 53 hosted zone에 맞췄다.
4. public DNS에서 apex A 레코드와 `www` CNAME이 기대한 값으로 보이는지 확인했다.
5. 그 뒤에 GitHub Pages custom domain을 다시 추가했다.
6. HTTPS, `www`와 이전 주소의 redirect, canonical, sitemap, feed, robots를 확인했다.
7. 마지막으로 Search Console Domain property를 등록하고 `sitemap.xml`을 제출했다.

네임서버를 바로잡은 뒤 apex DNS는 GitHub Pages IP들로 해석되기 시작했다. DNS 변경이 즉시 전 세계에 반영된다고 가정하지는 않았다. 내가 확인할 수 있는 public DNS 결과가 정상으로 바뀐 뒤에만 다음 단계로 넘어갔다.

최종적으로 확인한 상태는 다음과 같다.

- canonical domain은 `https://natekeem.com/`이다.
- HTTPS가 enforce된 상태다.
- `www.natekeem.com`은 apex로 redirect된다.
- `natekeem.github.io`는 `natekeem.com`으로 redirect된다.
- Dev Blog의 공개 글 수는 여전히 11개다.
- Search Console Domain property를 등록했다.
- `sitemap.xml`을 제출했다.

여기서 Search Console 등록과 sitemap 제출은 indexing 성공을 뜻하지 않는다. 검색 순위, traffic, revenue 성과를 확인한 것도 아니다. Google AdSense approval을 받은 것도 아니며, 이번 작업에서 광고나 tracking code를 추가하지 않았다.

## 다음부터 custom domain 앞에 중단 조건을 둔다

이번에 헷갈린 이유는 domain, DNS, GitHub Pages, Search Console을 각각의 설정 화면으로만 봤기 때문이다. 실제로는 앞 단계가 통과하지 않으면 다음 단계가 기존 공개 주소까지 흔들 수 있다.

그래서 다음 custom domain 작업에는 아래 순서와 중단 조건을 함께 두려 한다.

1. **domain name servers**: 도메인이 기대한 DNS hosted zone을 사용하도록 위임됐는지 먼저 확인한다.
2. **hosted zone records**: 그 다음 apex A 레코드와 `www` CNAME 같은 필요한 레코드를 확인한다.
3. **GitHub Pages custom domain**: public DNS가 기대한 값으로 해석되는 것을 확인한 뒤 custom domain을 적용한다.
4. **HTTPS와 certificate**: certificate 상태와 HTTPS enforce 여부를 확인한다.
5. **canonical, sitemap, feed, robots**: 공개 URL 신호가 새 domain으로 일관되게 바뀌었는지 확인한다.
6. **Search Console**: Domain property와 sitemap 제출 상태를 별도로 기록한다.
7. **future AdSense readiness review**: 앞 단계가 안정된 뒤에 별도 검토로 진행한다.

가장 중요한 중단 조건은 간단하다. 네임서버 위임과 public DNS resolution이 모두 통과하기 전에는 GitHub Pages custom domain을 적용하지 않는다. 둘 중 하나라도 기대와 다르면 작업을 멈추고 기존 공개 주소를 유지한다.

아직 이 절차를 실행하는 자동화 시스템을 만든 것은 아니다. 다만 나중에 custom domain 단계를 workflow로 옮긴다면, 이 확인은 참고용 checklist가 아니라 다음 상태로 넘어가지 못하게 하는 risk gate가 되어야 한다. 설정 API가 성공했다는 응답만으로 redirect를 시작해서는 안 된다.

이 순서는 이번 `나태킴의 커밋로그` 구성에서 얻은 운영 메모다. 다른 환경에서는 세부 절차가 달라질 수 있고, DNS propagation 시간도 일정하지 않다. 다음에는 설정 화면의 체크 표시보다 public DNS와 실제 redirect 결과를 먼저 보려 한다.

## 오늘 배운 것

내가 가장 크게 오해한 문장은 “DNS 레코드를 넣었다”였다. 정확히는 “hosted zone 안에 레코드를 작성했다”였고, 그 hosted zone이 도메인에서 실제로 사용되고 있는지는 별도의 확인 항목이었다.

또 custom domain은 새 주소를 추가하는 설정에 그치지 않았다. 기존 주소의 redirect, HTTPS certificate, canonical, sitemap, feed, robots, Search Console까지 이어지는 migration 단계였다. 새 도메인이 열리는지 확인하기 전에 redirect를 시작하면, 기존의 정상 경로까지 함께 끊을 수 있었다.

오늘의 장애는 DNS가 어려워서라기보다, 내가 확인 순서를 거꾸로 잡아서 생긴 문제였다. 다음 단계는 이 원고를 바로 발행하는 것이 아니다. cover와 final review는 별도 단계로 남겨 두고, 먼저 다음 custom domain 작업에서 사용할 checklist에 `네임서버 위임 확인`과 `public resolution 확인`을 실제 중단 조건으로 옮길 생각이다.
