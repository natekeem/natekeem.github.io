---
title: "AI 블로그 3개를 만들고 나서야 보인 진짜 문제"
date: 2026-07-17 02:05:00 +0900
categories:
  - Project AI Autoblog
  - Dev Log
tags:
  - AI
  - Codex
  - Content Operations
  - Search Readiness
  - Editorial QA
  - Dev Log
description: "AI-assisted blogging 아이디어로 시작한 Project AI Autoblog가 세 사이트의 장애와 검수를 거치며 저장소 경계, Search Readiness, 편집 품질, 사람 승인을 갖춘 운영체계로 바뀐 과정을 돌아본다."
published: true
image:
  path: /assets/img/posts/2026-07-17-project-ai-autoblog-operating-system-retrospective/cover.png
  alt: "서로 다른 세 블로그 화면이 하나의 구조화된 운영 흐름과 사람 검토 지점으로 연결된 편집 일러스트"
---

# AI 블로그 3개를 만들고 나서야 보인 진짜 문제

## 글을 자동으로 쓰면 끝날 줄 알았다

Project AI Autoblog를 시작할 때 머릿속에 있던 흐름은 단순했다. 주제를 고르고, AI로 초안을 만들고, 이미지를 붙인 다음 반복해서 발행한다. 어느 정도 검증되면 이 순서를 자동화할 수 있을 것 같았다.

지금 돌아보면 글 생성은 그 흐름에서 가장 설명하기 쉬운 부분이었다. 어려웠던 것은 생성된 글을 어느 사이트에, 어떤 목소리로, 어떤 근거를 남기며 공개할지 결정하는 일이었다. 배포 명령이 성공한 뒤에도 공개 주소, canonical, sitemap, cover가 실제로 같은 상태를 가리키는지 확인해야 했다. 설정 화면의 체크 표시와 독자가 만나는 결과가 다를 때는 어느 쪽을 프로젝트의 현재 상태로 기록할지도 정해야 했다.

나는 사이트의 방향과 공개 여부를 결정하고, 반복적인 파일 검사와 구현에는 Codex를 사용했다. 이 구분은 회고를 쓰는 방식에도 중요하다. 내가 모든 파일을 직접 열어 보고 모든 명령을 손으로 입력했다고 쓰면 실제 작업 방식과 다르다. 반대로 AI가 알아서 세 사이트를 운영했다고 말해도 사실이 아니다. 판단과 승인의 책임은 사람에게 있었고, agent는 정해진 범위 안에서 조사하고 만들고 검증하는 역할을 맡았다.

그렇게 세 사이트를 운영한 뒤 남은 것은 article generator가 아니었다. 어떤 상태를 믿을 수 있는지, 다음 작업자가 어디까지 바꿀 수 있는지, 어느 지점에서 사람이 반드시 멈춰 보고 승인해야 하는지를 정한 운영체계였다.

## 세 블로그는 같은 목소리로 말할 수 없었다

현재 portfolio에는 정확히 세 사이트가 있다.

`나태킴의 커밋로그`는 한국어 개발 기록이다. 성공한 결과만 정리하기보다 실제 결정과 실패, 남은 불확실성을 기록한다. 이 글을 발행하기 전까지 12편이 공개돼 있었고 canonical 주소는 `https://natekeem.com/`이다.

The Smart Suburbanite는 집 안의 기술을 어렵게 느끼는 독자를 위한 영어 evergreen guide다. 14편이 공개돼 있고, 차분한 homeowner voice를 사용한다. 사이트의 보이는 층은 Warm Utility Editorial로 바뀌었지만, 현재 AdSense 상태는 여전히 review in progress다. 승인이나 광고 송출, 수익을 확인한 상태가 아니다.

The Noise Filter는 현재 변화가 실제로 무엇을 뜻하는지 설명하는 영어 issue explainer다. Android 17 보안 변화나 Netflix mobile UI처럼 지금 일어나는 변화와 독자의 질문을 빠르게 연결한다. 공개 글은 2편이고, 더 긴장감 있는 Signal & Noise Editorial 정체성을 사용한다.

처음에는 한 workflow와 한 template을 잘 만들면 세 사이트에 재사용할 수 있을 것 같았다. 기술 기반은 공유할 수 있었지만 독자가 보는 리듬까지 같아지면 문제가 생겼다. Smart Suburbanite의 독자는 구매 전 불안을 낮추는 차분한 안내를 원한다. The Noise Filter의 독자는 최근 변화에서 무엇이 확정됐고 무엇을 더 지켜봐야 하는지 빨리 알고 싶어 한다. Dev Blog는 실제 작업의 순서와 판단 근거가 보여야 한다.

한 목소리와 한 화면을 세 번 복사하는 방식으로는 이 차이를 담을 수 없었다. 네 번째 블로그를 늘리는 대신, 이미 있는 세 사이트의 역할을 선명하게 유지하는 쪽을 택했다.

## 명령이 끝난 것과 프로젝트 상태가 바뀐 것은 달랐다

HQ 저장소와 세 사이트 저장소를 나눈 이유도 여기서 시작한다. HQ는 배포되는 사이트가 아니다. 정책, work package, claim table, review 기록, prompt, backlog, incident와 decision을 보관하는 source of truth다. 각 site repository는 실제로 build되고 공개되는 제품이다.

이 경계가 없으면 Smart Suburbanite 글을 고치다가 Dev Blog의 문서를 함께 commit하거나, The Noise Filter 조사 중에 다른 사이트의 설정을 건드리기 쉽다. 그래서 작업 전후에 네 저장소의 branch, HEAD, origin, ahead/behind, 변경 파일을 확인한다. 완료 뒤에는 site commit과 HQ 기록 commit을 따로 남긴다.

여기서 알게 된 것은 `command completed`가 곧 `project state changed`는 아니라는 점이다. 파일을 만들었다고 review가 끝난 것이 아니고, push했다고 deploy가 성공한 것도 아니다. sitemap을 제출했다고 indexing된 것도 아니며, URL Inspection에서 Request Indexing을 눌렀다고 검색 결과가 달라졌다고 말할 수도 없다.

prompt의 역할도 달라졌다. 긴 prompt 하나에 모든 규칙을 넣기보다, prompt는 어떤 module과 site profile을 읽고 어느 작업을 수행할지 지정하는 실행 인터페이스가 됐다. 현재 상태와 품질 기준은 version control 안의 문서에 남는다. prompt가 짧아져도 검수 단계가 줄지 않는 이유다.

## DNS 레코드보다 앞에 있던 문제

이 구분을 가장 아프게 배운 사건은 Dev Blog custom domain 전환이었다.

Route 53 hosted zone 안의 A record와 `www` CNAME은 맞아 보였다. 그런데 public DNS에서 `natekeem.com`은 `NXDOMAIN`을 반환했다. 빠진 것은 record 값이 아니라 authoritative nameserver delegation이었다. 도메인이 그 hosted zone의 nameserver를 실제 권한 서버로 사용하지 않으면, zone 안에 정확한 record를 적어도 외부에서는 보이지 않는다.

순서도 잘못 잡았다. public resolution을 확인하기 전에 GitHub Pages custom domain을 적용하자 기존 `natekeem.github.io`가 아직 열리지 않는 새 도메인으로 redirect되기 시작했다. 새 주소를 붙이는 작업이 원래 주소의 동작까지 바꾼다는 점을 늦게 확인했다.

복구는 custom domain을 잠시 제거해 기존 주소를 살리는 데서 시작했다. 그다음 nameserver delegation을 바로잡고, public DNS가 기대한 값을 반환하는 것을 확인한 뒤 domain을 다시 연결했다. HTTPS, 이전 주소와 `www` redirect, canonical, sitemap, feed, robots를 확인하고 나서야 migration을 완료 상태로 기록했다.

이 사건은 이미 별도 글로 자세히 남겼다. 이번 회고에서 중요한 것은 더 넓은 규칙이다. DNS record가 존재하는 것과 인터넷이 그 nameserver를 권한 서버로 사용하는 것은 다른 상태다. 외부 동작을 바꾸는 설정에는 API 성공 여부만이 아니라, 이전 공개 경로를 지킬 rollback 조건이 함께 있어야 했다.

## Search Console 경고를 그대로 고치지 않기로 했다

The Smart Suburbanite에서는 다른 종류의 불일치가 나왔다. Search Console에서 homepage의 user-declared canonical은 custom domain이었지만, Google-selected canonical은 한동안 이전 GitHub Pages host로 보고됐다.

이때 old host 문자열부터 지우는 식으로 대응하지 않았다. source, generated output, live page를 나눠 봤다. canonical, `og:url`, structured data, RSS, XML sitemap, robots와 redirect를 비교했지만 현재 공개 페이지에서 old host를 canonical로 선언하는 active signal은 찾지 못했다. Google이 다시 수집해 판단을 바꾸는 일은 코드 수정과 별개의 시간축에 있었다.

대신 감사 과정에서 실제 결함을 발견했다. 사람이 관리하던 `sitemap.txt`가 Article 010과 posts pagination 2페이지에서 멈춰 있었다. 새 글 011부터 014, 그리고 뒤에 생긴 pagination route가 빠져 있었다. XML sitemap은 자동으로 늘어나는데 text sitemap만 별도 목록으로 관리한 결과였다.

처음에는 누락 route를 맞췄고, 이후에는 같은 문제가 반복되지 않도록 XML sitemap의 canonical route 집합에서 `sitemap.txt`를 생성하게 바꿨다. 두 목록이 우연히 같기를 기대하지 않고 하나의 source에서 파생되게 한 것이다.

반면 세 번째 사이트의 public-host 문제는 아직 해결하지 않았다. The Noise Filter repository가 선언하는 canonical, sitemap, RSS는 GitHub Pages project URL을 사용한다. 하지만 그 주소로 요청하면 `natekeem.com/the-noise-filter/` 쪽으로 redirect된다. repository에는 CNAME이 없고 Pages API의 `cname`도 `null`이라 현재 증거만으로 동작 원인을 설명할 수 없다.

그래서 이 문제는 P0 incident로 열어 두고 Search Console 등록도 멈췄다. 편한 쪽의 hostname을 정답이라고 추측해 canonical을 바꾸면 redirect와 generated signal의 충돌을 더 굳힐 수 있다. final public host를 먼저 결정하고, Pages/domain 설정과 canonical migration을 rollback 계획까지 포함해 한 작업으로 다뤄야 한다.

## 검색 준비는 설정표가 아니라 비교 작업이 됐다

세 사건 뒤에 Search Readiness System이 생겼다. 핵심은 SEO 항목을 많이 적는 것이 아니라, 서로 다른 층이 같은 공개 상태를 말하는지 비교하는 데 있다.

canonical origin과 redirect chain, 공개 글 수와 sitemap/RSS item 수, XML과 text sitemap의 route parity를 확인한다. robots가 실제 200을 반환하는 sitemap을 가리키는지, Open Graph와 JSON-LD URL이 canonical host와 충돌하지 않는지, 옛 host가 generated output에 남았는지도 본다. custom domain에서는 DNS record와 nameserver delegation을 별도 단계로 검사한다.

검사는 source에서 끝나지 않는다. build output이 맞아도 배포된 페이지가 다를 수 있고, live page가 열려도 redirect가 deep path를 잃을 수 있다. 그래서 generated check와 live check를 분리했다. Search Console의 `submitted`, `registered`, `Request Indexing` 같은 행동도 실제 indexing 결과와 분리해 기록한다.

이 체계가 검색 성과를 보장하는 것은 아니다. 현재 글이 ranking이나 traffic을 얻었다는 증거도 제공하지 않는다. 다만 무엇을 확인했고 무엇은 아직 외부 플랫폼의 결과를 기다리는지 섞지 않게 해 준다.

## 사실이 맞는 글도 기계적으로 보일 수 있었다

영어 글이 늘자 검색 설정과는 다른 불안이 생겼다. claim이 사실이고 출처가 있어도 글의 움직임이 지나치게 균일하면 AI가 만든 template처럼 보일 수 있었다. 같은 순서의 heading, 비슷한 세 항목 목록, 매번 붙는 안전 문구가 반복됐다. 틀린 문장은 아니지만 여러 글을 연달아 보면 문체가 납작해졌다.

그렇다고 사람 냄새를 내기 위해 사용하지 않은 제품 경험이나 존재하지 않는 대화를 만들 수는 없다. 선택한 해법은 detector 점수를 높이는 일이 아니라, 관찰 가능한 언어 패턴을 점검하는 것이었다. 공통 Editorial Quality Standard 아래에 site-specific voice를 두고, 반복 heading, disclaimer 밀도, 전환 문구, 세 항목 나열, 거짓 1인칭 경험을 찾는다. 영어 원고에는 한국어 review summary와 claim/fact table을 붙이고, 사람이 공개 여부를 승인한다.

Netflix 글은 앱 화면이 갑자기 달라 보이는 독자의 순간에서 시작한다. Android 17 글은 은행 전화를 받는 현실적인 상황에서 출발한다. 둘 다 The Noise Filter의 글이고 독자가 질문을 품는 장면을 사용하지만, 그 뒤까지 같은 `무엇이 바뀌었나-무엇을 해야 하나-결론` 박자로 복사하면 시작 장면만 바꾼 template이 된다. template은 빠뜨릴 항목을 막는 scaffold로 쓰되, 독자가 보는 구조를 강제하는 틀로 만들지 않기로 했다.

## 기반은 남기고 독자가 보는 경험을 다시 설계했다

두 영어 사이트는 AstroPaper를 기술 기반으로 공유한다. content collection, route generation, pagination, RSS, sitemap 같은 검증된 부분을 버릴 이유는 없었다. 대신 독자가 보는 층은 사이트 목적에 맞게 달라져야 했다.

The Noise Filter는 modern issue-explainer magazine인 Signal & Noise Editorial로 정리했다. Smart Suburbanite는 차분하고 실용적인 home-technology guide인 Warm Utility Editorial로 바꿨다.

homepage만 바꾸면 site identity가 완성되지 않았다. article layout, category, search, card, About·Contact 같은 trust page, mobile navigation까지 같은 목적을 말해야 했다. Smart Suburbanite redesign에서는 기존 public URL과 cover를 보존한 채 이 층을 함께 손봤다. 기술 기반을 재사용하는 것과 독자 경험을 복제하는 것은 다른 결정이었다.

## 결국 17개 상태와 12개 역할이 생겼다

현재 Operating System v2에는 content가 거치는 17개 상태와 12개 agent role이 정의돼 있다. central site registry는 공개 origin, 글 수, build command, blocker를 모은다. active backlog는 다음에 할 일과 멈춘 일을 구분하고, incident와 decision index는 같은 판단을 다시 조사하지 않게 한다. HQ status와 consistency audit는 문서와 실제 repository가 어긋난 지점을 찾는다.

실제 흐름은 숫자보다 단순하게 읽힌다. topic을 찾고, research와 brief를 만들고, draft를 쓴다. claim과 문체를 검토하고 사람이 승인한다. 그다음 image와 SEO metadata를 준비하고, 선택된 site repository에만 발행한다. 배포 뒤 public route를 확인하고 HQ에 결과를 기록한다. 각 역할은 다음 작업자에게 입력, 근거, 변경 범위, 통과한 gate, 남은 위험을 같은 handoff 형식으로 넘긴다.

가장 중요한 상태 규칙은 두 줄이다. 사람 승인이 없으면 어떤 글도 `approved_for_publish`에 들어가지 않는다. 공개 페이지를 확인하기 전에는 `published_verified`가 되지 않는다.

이번 글도 예외가 아니다. 사용자는 회고의 작성과 공개를 요청했고, 원고와 claim table, 한국어 review packet, cover QA를 만든 뒤 그 승인을 발행 gate로 기록했다. push가 끝난 뒤에도 live article, cover, canonical, feed, sitemap과 mobile 화면을 확인해야 최종 상태가 바뀐다.

## 자동화하지 않기로 한 것들

이 체계는 아직 중앙 orchestrator도 아니고 완전 자동 발행 시스템도 아니다. 새로운 블로그를 알아서 만들지 않는다. domain을 구매하거나 DNS를 바꾸지 않고, Search Console property 등록과 indexing request도 자동으로 하지 않는다. AdSense를 활성화하거나 대량 발행하지 않으며, 기존 공개 글 전체를 한꺼번에 다시 쓰지도 않는다.

무엇보다 ranking, traffic, revenue를 선언하지 않는다. build와 deploy가 성공해도 business outcome은 별도 증거가 필요한 상태다.

사람 승인을 남긴 이유는 AI의 능력을 과소평가해서가 아니다. topic의 가치, 문장의 자연스러움, source의 충분성, 공개로 인해 생길 위험은 같은 숫자로 환산되지 않는다. domain과 외부 platform 설정은 잘못 바꾸면 정상 공개 경로까지 끊을 수 있다. agent가 반복 검사를 더 잘 수행할수록, 바꿔도 되는 범위와 멈춰야 할 조건도 더 명시적이어야 했다.

## 29번째 글 뒤에도 남는 일

이 글을 쓰기 전 공개 corpus는 Dev Blog 12편, The Smart Suburbanite 14편, The Noise Filter 2편으로 모두 28편이었다. 이 회고가 공개되면 Dev Blog는 13편, 전체는 29편이 된다.

숫자가 늘었지만 프로젝트가 끝난 것은 아니다. 가장 높은 우선순위에는 The Noise Filter의 final public-host conflict가 남아 있다. Smart Suburbanite에는 새 글마다 presentation category를 수동 mapping해야 하는 위험, GitHub Actions의 Node-version warning, 기존 formatting baseline이 남아 있다. AdSense review 결과도 아직 정해지지 않았다. 다음 content 작업은 The Noise Filter Article 003의 topic selection이지만, host 경계를 넘는 발행은 별도 판단 없이 진행하지 않는다.

이 항목들을 실패 목록으로 보지는 않는다. 모르는 상태를 모른다고 기록하고, 자동으로 넘어가지 못하게 막아 둔 backlog다. 운영체계가 해 주는 일은 문제를 없애는 것이 아니라 문제의 현재 상태와 다음 권한을 흐리지 않는 데 가깝다.

처음에는 더 많은 글을 더 빨리 만드는 것이 어려울 거라 생각했다. 실제로 더 어려웠던 것은 어느 상태가 사실인지, 어떤 증거면 다음 단계로 넘어가도 되는지, agent가 어디까지 바꿀 수 있는지, 사람이 어디에서 마지막으로 `예`라고 해야 하는지를 정하는 일이었다.

Project AI Autoblog가 지금 만든 것은 글을 쓰는 기계보다 그 질문을 반복해서 놓치지 않기 위한 운영체계에 더 가깝다.
