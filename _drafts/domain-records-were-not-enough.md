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

## 레코드가 있으면 연결된 줄 알았다

오늘 `나태킴의 커밋로그`를 `natekeem.github.io`에서 `natekeem.com`으로 옮겼다. 목표는 주소만 짧게 만드는 것이 아니었다. Dev Blog의 canonical domain을 정리하고, Search Console을 custom domain 기준으로 연결하고, 나중에 Google AdSense readiness를 검토할 수 있는 기반을 만들려는 작업이었다.

Route 53 hosted zone 안에서 본 DNS 레코드는 맞아 보였다.

- apex에는 GitHub Pages용 A 레코드가 들어 있었다.
- `www`에는 `natekeem.github.io`를 가리키는 CNAME이 들어 있었다.

그래서 나는 레코드를 다 넣었으니 이제 기다리면 된다고 생각했다. 하지만 public DNS에서 `natekeem.com`을 조회하면 결과는 `NXDOMAIN`이었다. hosted zone 화면에 레코드가 존재한다는 사실과, 인터넷에서 그 hosted zone을 실제로 사용하고 있다는 사실을 같은 것으로 본 게 문제였다.

## 빠뜨린 것은 네임서버 위임이었다

내 설정에서 먼저 확인했어야 할 것은 도메인의 name server였다. 도메인이 Route 53 hosted zone에 지정된 name server를 사용하지 않으면, 그 hosted zone 안에 A 레코드와 CNAME을 정확히 넣어도 public DNS 조회에는 반영되지 않는다.

쉽게 말하면 주소록 안의 내용은 작성했지만, 사람들이 어느 주소록을 찾아가야 하는지는 제대로 연결하지 않은 상태였다. 나는 레코드 값만 확인했고, 도메인의 name server가 기대한 hosted zone을 가리키는지는 뒤늦게 확인했다.

이 경험을 모든 DNS 환경에 그대로 적용할 수 있는 보편적인 안내로 쓰려는 것은 아니다. 등록기관과 DNS 제공자 구성은 다를 수 있다. 다만 이번 `natekeem.com` 전환에서는 레코드보다 앞단의 name server 위임이 빠져 있었고, 그것이 `NXDOMAIN`의 직접적인 원인이었다.

## GitHub Pages custom domain을 너무 일찍 넣었다

문제는 DNS만으로 끝나지 않았다. 나는 public DNS가 정상인지 확인하기 전에 GitHub Pages에 `natekeem.com` custom domain을 먼저 추가했다.

그러자 기존 주소인 `natekeem.github.io`가 `natekeem.com`으로 redirect되기 시작했다. 그런데 새 도메인은 아직 public DNS에서 해석되지 않았다. 결과적으로 정상적으로 열리던 기존 주소까지, 열리지 않는 새 주소로 방문자를 보내는 상태가 됐다.

이것은 GitHub Pages가 문제를 일으킨 것이 아니다. custom domain을 적용하면 기존 GitHub Pages 주소가 새 canonical domain으로 이동하는 동작을 예상하고, 그 전에 새 도메인이 실제로 해석되는지 확인했어야 했다. 순서를 앞당긴 내 판단이 문제였다.

우선 사이트를 복구하기 위해 GitHub Pages custom domain을 제거했다. 그러자 `natekeem.github.io`에서 Dev Blog가 다시 열렸다. 새 도메인 준비가 끝날 때까지 기존 공개 경로를 살려 둔 임시 복구였다.

## name server부터 바로잡고 다시 시작했다

도메인의 name server가 올바른 Route 53 hosted zone을 가리키도록 수정한 뒤에는 apex DNS가 GitHub Pages IP들로 해석되기 시작했다. 여기서도 DNS 변경이 즉시 전 세계에 반영된다고 가정하지 않았다. 내가 확인할 수 있는 public DNS 결과가 정상으로 바뀐 뒤에 다음 단계로 넘어갔다.

그 다음에야 GitHub Pages custom domain 설정을 다시 적용했다. certificate와 HTTPS 상태를 확인하고, canonical, sitemap, feed, robots가 모두 새 도메인을 기준으로 향하는지 점검했다. `www`와 예전 GitHub Pages 주소의 redirect도 따로 확인했다.

최종적으로 확인한 상태는 다음과 같다.

- canonical domain은 `https://natekeem.com/`이다.
- HTTPS가 enforce된 상태다.
- `www.natekeem.com`은 apex로 redirect된다.
- `natekeem.github.io`는 `natekeem.com`으로 redirect된다.
- Dev Blog의 공개 글 수는 여전히 11개다.
- Search Console Domain property를 등록했다.
- `sitemap.xml`을 제출했다.

여기서 Search Console 등록과 sitemap 제출은 indexing 성공을 뜻하지 않는다. 검색 순위, traffic, revenue 성과를 확인한 것도 아니다. Google AdSense approval을 받은 것도 아니며, 이번 작업에서 광고나 tracking code를 추가하지 않았다.

## 다음부터 지킬 순서

이번에 헷갈린 이유는 domain, DNS, GitHub Pages, Search Console을 각각의 설정 화면으로만 봤기 때문이다. 실제로는 앞 단계가 준비되지 않으면 다음 단계를 안전하게 진행하기 어렵다.

다음 custom domain 작업에서는 아래 순서를 기준으로 삼으려 한다.

1. **domain name servers**: 도메인이 어떤 DNS hosted zone을 사용하도록 위임됐는지 먼저 확인한다.
2. **hosted zone records**: 그 다음 apex A 레코드와 `www` CNAME 같은 필요한 레코드를 확인한다.
3. **GitHub Pages custom domain**: public DNS가 기대한 값으로 해석되는 것을 확인한 뒤 custom domain을 적용한다.
4. **HTTPS와 certificate**: certificate 상태와 HTTPS enforce 여부를 확인한다.
5. **canonical, sitemap, feed, robots**: 공개 URL 신호가 새 domain으로 일관되게 바뀌었는지 확인한다.
6. **Search Console**: Domain property와 sitemap 제출 상태를 별도로 기록한다.
7. **future AdSense readiness review**: 앞 단계가 안정된 뒤에 별도 검토로 진행한다.

이 순서는 이번 `나태킴의 커밋로그` 구성에서 얻은 운영 메모다. 다른 환경에서는 세부 절차가 달라질 수 있고, DNS propagation 시간도 일정하지 않다. 그래서 다음에는 설정 화면의 체크 표시보다 public DNS와 실제 redirect 결과를 먼저 확인할 생각이다.

## 오늘 배운 것

내가 가장 크게 오해한 문장은 “DNS 레코드를 넣었다”였다. 정확히는 “hosted zone 안에 레코드를 작성했다”였고, 그 hosted zone이 도메인에서 실제로 사용되고 있는지는 별도의 확인 항목이었다.

또 custom domain은 새 주소를 추가하는 설정에 그치지 않았다. 기존 주소의 redirect, HTTPS certificate, canonical, sitemap, feed, robots, Search Console까지 이어지는 migration 단계였다. 새 도메인이 열리는지 확인하기 전에 redirect를 시작하면, 기존의 정상 경로까지 함께 끊을 수 있었다.

다음에는 레코드 표부터 보지 않겠다. 먼저 이 도메인이 어느 name server를 보고 있는지 확인하고, public DNS가 정상인 상태에서 GitHub Pages migration을 시작하겠다. 오늘의 장애는 DNS가 어려워서라기보다, 내가 확인 순서를 거꾸로 잡아서 생긴 문제였다.
