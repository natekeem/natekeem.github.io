---
title: "AdSense에서 ‘가치가 별로 없는 콘텐츠’ 판정을 받고 블로그 14편을 다시 설계한 과정"
categories:
  - Project AI Autoblog
  - Dev Log
tags:
  - AI
  - Smart Suburbanite
  - Google AdSense
  - Content Operations
  - Editorial QA
  - Dev Log
description: "The Smart Suburbanite의 AdSense low-value-content 거절 뒤 14편의 공개 근거, 고유 판단, 책임 구조를 감사하고 다시 설계한 과정과 아직 대기 중인 재심사 상태를 기록한다."
date: 2026-07-19 09:29:00 +0900
published: true
image:
  path: /assets/img/posts/adsense-low-value-content-blog-rebuild-retrospective/cover.png
  alt: "반복된 문서 더미가 서로 다른 역할의 묶음으로 재편되고 가장자리에 미완료 검토 표식이 놓인 작업대"
---

# AdSense에서 ‘가치가 별로 없는 콘텐츠’ 판정을 받고 블로그 14편을 다시 설계한 과정

## 거절 화면을 보고 가장 먼저 떠올린 설명

2026년 7월 17일, The Smart Suburbanite의 AdSense 화면에는 `rejected — low-value content`가 표시돼 있었다. 한국어 화면에서 보이는 표현은 ‘가치가 별로 없는 콘텐츠’였다. 글은 14편이었고, 모두 짧은 메모가 아니라 1,000단어를 훌쩍 넘는 영문 가이드였다. 사이트에는 About, Privacy, Disclosure 같은 기본 페이지도 있었고, 공개 URL과 sitemap, RSS, 검색 기능도 작동하고 있었다. 그런데 화면에 남은 판정은 한 줄로 단호했다.

처음 떠오른 설명은 디자인과 심사 시점이었다. 거절을 확인하기 며칠 전, 홈페이지와 탐색 구조를 크게 손봤다. 기존 AstroPaper 기반은 유지하면서도 카드, 카테고리, 검색, 본문 폭, 모바일 내비게이션을 The Smart Suburbanite에 맞게 다시 설계했다. 정확한 crawl 시각이나 심사 snapshot은 제공되지 않았으니 Google이 그보다 오래된 화면을 봤을 가능성도 있었다.

이 설명만 붙잡으면 다음 행동은 간단했다. “지금은 사이트가 달라졌으니 문제를 고쳤다”고 체크하고 곧바로 재심사를 요청하면 된다. 하지만 기록을 다시 보니 그렇게 결론 내릴 근거가 없었다. 최근 개편이 바꾼 것은 주로 사이트의 껍데기와 이동 경로였다. 14편의 article body는 그대로 보존돼 있었다. Google이 어느 버전을 봤는지는 몰라도, 독자가 읽는 글의 내용은 디자인 전후에 크게 달라지지 않았다.

그래서 오래된 화면이 심사됐을 수 있다는 가능성은 기록하되, 거절을 오래된 디자인 탓으로 정리하지 않기로 했다. 직접적인 내부 판정 근거는 알 수 없었다. 대신 지금 공개된 사이트가 독자에게 충분한 근거와 고유한 판단, 책임 주체를 보여 주는지 별도로 확인하기로 했다. 이 글은 “거절 원인을 알아냈다”는 성공담이 아니라, 그 질문을 14편 전체에 던진 뒤 운영 방식을 바꾼 기록이다.

## 디자인이 바뀌었다는 사실만으로는 충분하지 않았다

리디자인은 분명 필요한 작업이었다. 이전 홈페이지는 글을 최신순으로 보여 주는 데는 문제가 없었지만, 처음 방문한 사람이 어디서 시작해야 하는지 판단하기 어려웠다. Beginner Guides, Security & Privacy, Maintenance & Setup, Comfort & Convenience라는 네 갈래를 더 선명하게 만들고, 카드와 본문 폭을 다듬고, 검색과 아카이브를 찾기 쉽게 바꿨다. 독자가 길을 잃지 않도록 하는 일은 콘텐츠의 일부다.

다만 길을 잘 안내하는 것과 도착한 글이 읽을 가치가 있는 것은 다른 문제였다. 문법이 맞고 레이아웃이 깨지지 않는다고 해서 페이지의 존재 이유가 저절로 생기지는 않는다. 긴 글도 다른 사이트에서 볼 수 있는 설명을 매끄럽게 늘여 놓은 것이라면 독자에게 새 판단을 주지 못한다. 반대로 짧더라도 비교 기준이나 실패 조건, 확인해야 할 문서를 정확히 제시하면 실용적인 가치가 생길 수 있다.

이 차이는 프로젝트 내부에서 더 잘 보이지 않았다. 각 글에는 영문 초안뿐 아니라 한국어 검토 패킷, claim/fact table, 출처 메모, 사람 승인 기록이 있었다. 내부 workflow만 보면 꽤 많은 검토를 거친 글이었다. 그러나 독자는 그 폴더를 볼 수 없다. 공개 본문에서 확인할 수 있는 것은 완성된 문장뿐이었다. 내부에 공식 문서 목록이 있어도 공개 페이지의 근거가 0개라면, 독자가 판단할 수 있는 신뢰에는 거의 기여하지 못한다.

결국 리디자인과 콘텐츠 복구를 분리해서 보기로 했다. 전자는 탐색과 시각적 정체성의 문제였고, 후자는 “왜 이 글이 존재해야 하는가”의 문제였다. 화면을 개선한 사실을 콘텐츠 개선의 대리 지표로 쓰지 않는 것이 첫 번째 결정이었다.

## 14편을 한 줄씩 읽을 때보다 한꺼번에 펼쳤을 때 보인 것

개별 글만 읽으면 대부분 무난했다. 스마트홈이 무엇인지 설명하고, 첫 기기를 사기 전에 물어볼 질문을 정리하고, thermostat나 camera, smart lock의 trade-off를 알려 줬다. 지나치게 공격적인 구매 권유도 없었고, 써 보지 않은 제품을 직접 테스트했다고 말하지도 않았다. 각 글은 충분히 길었다.

문제는 14편을 하나의 포트폴리오로 놓고 비교했을 때 드러났다. 감사 당시 공개된 14개 article body의 외부 출처 링크는 모두 합쳐 0개였다. HQ에는 FTC, NIST, ENERGY STAR, 제조사 지원 문서가 정리돼 있었지만 공개 페이지에서는 확인할 수 없었다. 출처가 없는 문장이 반드시 틀렸다는 뜻은 아니다. 다만 구독 정책, 호환성, 저장 기간, 계정 권한처럼 바뀔 수 있는 정보가 무엇에 근거했는지 독자가 확인할 길이 없었다.

구조도 반복됐다. 14편 가운데 9편에 `Quick Answer`가 있었고, 8편은 `Who This Guide Is For` 계열의 절을 썼다. 구매 전 checklist는 8편, `Common Mistakes` 계열은 7편, Buy/Wait/Skip 판단은 7편에 나타났다. `What to Read Next`처럼 사이트 탐색에 유용한 공통 요소도 있었지만, Articles 008, 009, 010은 절의 이동 순서까지 비슷했다. 한 편에서는 친절한 구조가 열 편에서는 조립식 흔적으로 보일 수 있었다.

책임 주체도 흐렸다. article header에는 눈에 보이는 byline이 없었다. 설정에 들어 있던 `The Smart Suburbanite`라는 브랜드명은 JSON-LD에서 `Person` author로 선언돼 있었다. About는 human review를 언급했지만, 무엇을 조사하고 누가 최종 공개를 결정하며 오류를 어떻게 고치는지 설명하지 않았다. 실명을 공개해야 한다고 단정할 수는 없지만, 브랜드를 사람으로 표시하면서 책임 절차는 보이지 않는 상태는 정직한 표현이 아니었다.

초보자용 글 사이의 경계도 겹쳤다. “스마트홈이란 무엇인가”, “첫 기기 구매 전에 무엇을 물어볼까”, “처음 사면 후회하기 쉬운 기기”가 모두 compatibility, Wi-Fi, subscription, privacy, household access를 반복했다. 주제는 달랐지만 독자가 글을 다 읽고 얻는 판단은 비슷해질 위험이 있었다.

이 관찰을 Google의 내부 판정 사유라고 쓸 수는 없다. 심사자가 어떤 페이지를 언제 봤는지, 어느 항목을 문제로 판단했는지 알 수 없기 때문이다. 다만 공개 evidence가 보이지 않고, 구조가 반복되며, 책임 설명과 글 사이의 역할 경계가 약한 상태는 ‘가치가 별로 없는 콘텐츠’라는 평가와 연결될 수 있는 위험으로 봤다. 무엇보다 AdSense 결과와 무관하게 독자에게 고칠 가치가 있었다.

### 복구 과정을 한 장으로 정리하면

| 단계 | 발견 | 한 일 | 확인된 결과 |
| --- | --- | --- | --- |
| 7월 17일 거절 확인 | visible reason은 low-value content, 정확한 심사 snapshot은 알 수 없음 | 곧바로 재심사를 누르지 않고 14편과 trust surface를 감사 | 원인은 미확정, 고칠 만한 공개 가치 위험은 확인 |
| Phase 1 | byline·research method·correction path·schema 책임 구조가 약함 | About 등 신뢰 페이지와 article 책임 표시를 정비 | 신뢰 구조 공개, 14개 본문과 URL은 그대로 유지 |
| Phase 2 | 다섯 priority 글이 일반적인 checklist에 머묾 | 공식 출처와 글별 고유 decision artifact를 추가 | 006·007·008·009·011을 `STRENGTHENED — MONITORING`으로 기록 |
| Phase 3 | 001·002·010의 독자 일이 겹침 | 합치기보다 orientation·worksheet·casebook으로 역할 분리 | 세 글을 `REPOSITIONED — MONITORING`으로 기록 |
| 7월 18일 최종 감사 | 14편 모두에 질문·artifact·evidence·overlap 결정이 필요 | article value matrix와 local/live QA를 완료 | 사람에게 재심사 제출 여부를 판단할 근거 제공 |
| 7월 19일 01:04 | 계정 소유자의 외부 행동 필요 | 사용자가 수동으로 `Request review` 제출 | 현재 `re-review submitted — pending`; 결과는 아직 모름 |

## 첫 번째 수정은 글이 아니라 책임 구조였다

Phase 1에서 14편을 바로 다시 쓰지 않았다. 먼저 사이트가 어떤 기준으로 글을 만들고 누가 그 주장에 책임지는지 공개하는 층을 정비했다. About는 단순한 대상 독자 소개에서 벗어나 research-based guide와 실제 테스트의 차이, AI 보조와 사람 결정의 경계를 설명하도록 바꿨다. Editorial Process 페이지를 새로 만들어 topic selection, source hierarchy, 변경 가능한 정보의 재확인, correction과 update 절차를 공개했다.

Contact에는 오류와 오래된 정보, broken link를 알릴 수 있는 correction path를 넣었다. Disclosure와 Privacy는 아직 없는 광고, affiliate, analytics를 있는 것처럼 쓰지 않으면서 현재의 상업적 관계와 데이터 처리 범위를 설명하도록 다듬었다. 기사 상단에는 `By The Smart Suburbanite`와 `Research-based guide · Human reviewed`를 보이게 했고, 본문 뒤에는 published documentation과 editorial research에 기반하며 명시하지 않은 hands-on testing을 주장하지 않는다는 method note를 붙였다.

JSON-LD도 같은 책임 모델을 따르게 했다. 브랜드를 `Person`으로 표시하던 author를 `Organization`으로 바꾸고 publisher와 일치시켰다. article URL, description, `mainEntityOfPage`도 보완했다. 카테고리 breadcrumb가 가리키던 `/categories/` 404는 실제 index route를 만들어 닫았고, footer에서 About, Editorial Process, Contact, Privacy, Disclosure와 네 카테고리에 접근할 수 있게 했다.

이 작업은 site commit `27bb983`으로 배포됐다. 당시 14개 article body의 hash와 URL은 그대로였다. 이 범위가 중요했다. 신뢰 페이지를 고쳤다는 이유로 글의 내용까지 좋아졌다고 보고하지 않기 위해서다. Trust page는 책임과 방법을 보여 줄 수 있지만, 약한 article을 대신 읽어 주지는 않는다. Phase 1은 필요한 기반이었고 충분한 복구는 아니었다.

### 독자에게 보이는 포트폴리오 모델의 변화

| 영역 | 이전 | 이후 | 독자에게 달라진 점 |
| --- | --- | --- | --- |
| 출처 | 내부 package에는 있었지만 14개 공개 본문 링크는 0개 | 우선순위 글과 재배치 글에 공식·권위 출처와 확인 날짜 공개 | 변경 가능한 claim을 직접 확인할 수 있음 |
| Byline | article header에 없음 | 조직 byline과 About 연결 | 누가 공개 책임을 지는지 추적 가능 |
| 연구 방법 | “human review” 한 줄에 가까움 | source hierarchy, research/testing 경계, AI 보조, 사람 승인 공개 | 글이 만들어진 방식과 한계를 이해 가능 |
| 글 구조 | Quick Answer·audience·checklist·Buy/Wait/Skip 반복 | 주제별 decision tree, matrix, worksheet, casebook으로 분화 | 다음 글에서도 같은 답을 반복해 읽는 느낌 감소 |
| 고유 artifact | 일반적인 요약과 질문 목록 중심 | 호환 경고표, missed-alert test, privacy matrix, load screening 등 | 정보를 실제 선택에 적용할 수 있음 |
| 겹침 | 입문 글 세 편이 비슷한 risk list 공유 | system orientation, one-purchase worksheet, mismatch casebook으로 역할 분리 | 어떤 질문에 어느 글을 읽을지 선명해짐 |
| Schema | 브랜드를 `Person` author로 표현 | `Organization` author/publisher로 정렬 | 보이는 byline과 기계 표현이 일치 |
| 수정 경로 | 오류 제보와 update 기준이 모호 | Contact, corrections, modification/update 원칙 공개 | 오래된 정보를 신고하고 처리 기준을 확인 가능 |
| 발행 QA | build와 파일 검사가 중심 | source refresh, browser, mobile table, live route까지 분리 | 생성 성공과 실제 사용 가능성을 따로 확인 |

## 다섯 편을 더 길게 쓰는 대신 서로 다르게 다시 썼다

Phase 2에서는 위험과 장기 utility가 큰 다섯 편을 골랐다. 기준은 단순한 traffic 예상이 아니었다. 제품 세대, 구독, 저장, 호환성, 계정 권한처럼 자주 바뀌는 claim이 있고, 공식 문서를 바탕으로 독립적인 판단 도구를 만들 수 있는 글을 우선했다. Article 007, 006, 011, 008, 009 순으로 조사와 승인, 배포를 진행했다.

중요한 목표는 word count를 늘리는 것이 아니었다. 다섯 글 모두 이미 충분히 길었다. 추가해야 했던 것은 각 글에서만 얻을 수 있는 판단이었다. 공식 문서 문장을 이어 붙이거나 제품 순위를 만들지 않고, 여러 출처를 같은 질문 아래 비교해 독자가 선택과 중단 조건을 볼 수 있게 했다.

| 글 | 기존 약점 | 공개한 근거 | 새로 만든 artifact | 독자가 내릴 판단 |
| --- | --- | ---: | --- | --- |
| 006 Smart Thermostat Compatibility | 일반적인 호환 질문이 정확한 모델 pre-check로 이어지지 않음 | 공식 링크 10개 | 9개 경고 신호 matrix, 3개 ecosystem checker 비교, 확인 정보 worksheet | 구매 전에 멈추고 HVAC·배선·accessory 정보를 더 모아야 하는가 |
| 007 Video Doorbells With No Monthly Fee | “구독 없음”을 하나의 기능처럼 다룸 | 제조사 링크 11개 | 4개 모델 비교와 missed-alert decision test | 알림을 놓친 뒤에도 필요한 기록과 접근이 남는가 |
| 008 Camera Categories | indoor/outdoor/doorbell을 기능 목록으로 비교 | 공식·권위 링크 10개 | location·purpose·privacy boundary matrix | 카메라가 필요한지, 필요하다면 어느 위치와 종류가 맞는가 |
| 009 Smart Plugs vs Power Strips | outlet 수와 편의 기능에 치우침 | 공식·권위 링크 11개 | 12개 load risk screening과 plug/strip decision table | 자동화해도 되는 load인지, 아예 연결하지 말아야 하는지 |
| 011 Smart Locks | 편리함과 보안을 넓게 나열 | 공식·권위 링크 10개 | access routine과 failure-recovery planning | 배터리·계정·공유 권한·물리적 backup을 가정이 감당할 수 있는가 |

예를 들어 thermostat 글은 “C-wire를 확인하세요”로 끝내지 않았다. control voltage, heat pump와 auxiliary heat, multi-stage, accessory terminal, proprietary connector, panel access처럼 정확한 모델 확인 전에 멈춰야 할 신호를 나눴다. Google, ecobee, Resideo의 checker가 무엇을 묻고 무엇을 대신 보장하지 않는지도 비교했다. 그렇다고 특정 집에 설치할 수 있다고 판정하거나 안전을 보장하지는 않았다.

Doorbell 글은 네 생태계의 “무료”를 같은 말로 취급하지 않았다. live view, alert, missed-event history, local storage, hub나 card 같은 추가 하드웨어, household access를 분리했다. 핵심 질문은 월 구독료가 0원인가가 아니라, 알림을 놓쳤을 때 독자가 기대한 evidence가 남는가였다.

Camera 글은 제품을 먼저 놓지 않고 해결하려는 문제와 frame에 들어오는 사람을 먼저 놓았다. Plug와 strip 글은 편의 기능보다 load의 결과를 먼저 봤고, Smart lock 글은 잠금 해제 방식보다 계정 관리와 배터리 실패, 물리적 backup을 하나의 일상으로 묶었다. 직접 제품을 사서 써 보지 않았기 때문에 hands-on review를 만들 수는 없었다. 대신 공식 문서의 서로 다른 범위를 연결하고, 불일치와 확인 질문을 드러내는 편집 artifact를 만들었다.

여기서 독창성은 “세상에 없던 사실”을 발명하는 일이 아니었다. 같은 제품 문서를 읽어도 어떤 기준으로 나누고, 어떤 선택지를 제외하며, 어디서 결정을 멈추게 하는지는 편집 판단이다. 출처가 공개되고 한계가 명시돼 있다면 직접 사용 경험을 꾸미지 않고도 고유한 가치를 만들 수 있었다.

## 겹치는 입문 글 세 편은 합치지 않고 질문을 갈라 놓았다

Phase 3에서는 Articles 001, 002, 010을 다시 봤다. 처음에는 비슷한 내용을 합치고 redirect하는 편이 깔끔해 보였다. 세 글 모두 compatibility, network, subscription, privacy, household access를 다뤘기 때문이다. 중복을 줄이는 가장 쉬운 방법은 페이지 수를 줄이는 것이다.

하지만 겹치는 단어가 곧 같은 reader job을 뜻하지는 않았다. “스마트홈이 어떻게 구성되는가”를 처음 이해하려는 사람과 “지금 보고 있는 한 기기를 사도 되는가”를 판단하는 사람, “왜 첫 구매가 후회로 이어지는가”를 살피는 사람은 다른 결과를 원한다. 각 글이 그 결과를 분명히 만들 수 있다면 합치지 않는 편이 더 유용했다.

Article 001은 smart-home system orientation으로 좁혔다. device, connection, control, response, fallback이라는 다섯 부분을 통해 전체 구조를 설명하고, 네 카테고리 중 다음에 어디로 갈지 안내한다. Protocol detail은 Article 003으로 넘기고 구매 판단은 Article 002로 넘겼다.

Article 002는 one-device purchase worksheet가 됐다. “스마트홈을 시작할까요?” 같은 넓은 질문 대신 한 제품을 놓고 requirement, compatibility, account, support, cost, fallback을 채운 뒤 proceed, investigate, stop 중 하나를 고르게 했다. 설정 순서는 Article 004, 반복되는 실패 pattern은 Article 010의 몫으로 남겼다.

Article 010은 나쁜 제품 category 목록이 아니라 mismatch와 regret pattern의 casebook으로 바꿨다. 큰 약속 뒤에 숨은 dependency, early warning, lower-commitment first step을 연결했다. 같은 category도 어떤 집에서는 잘 맞고 다른 집에서는 부담이 될 수 있다는 경계를 유지했다.

세 글에는 각각 공개 공식 출처와 고유 artifact를 넣었다. 최종 14-article value matrix에는 모든 글의 reader question, original public value, evidence freshness, overlap decision, maintenance burden을 한 줄씩 기록했다. “대체로 generic하지만 결정은 미정”인 글을 0개로 만드는 것이 Phase 3의 종료 조건이었다. 이는 모든 글이 완벽하다는 뜻이 아니라, 유지·강화·재배치 이유를 설명할 수 없는 페이지를 남기지 않았다는 뜻이다.

## 빌드가 통과해도 자동화가 잡지 못한 것

복구 과정에서 deterministic check는 많은 실수를 막았다. formatting, lint, Astro diagnostics, internal link, canonical, RSS, sitemap, Pagefind, source count, private-path scan을 반복했다. 하지만 통과한 build가 공개 경험 전체를 증명하지는 않았다. 이번 프로젝트의 다른 사건들도 같은 경계를 보여 줬다.

첫 번째는 Dev Blog의 future-dated post였다. 파일과 build는 정상인데 KST 날짜 경계 때문에 기대한 public route가 404였다. source가 `_posts`에 있다는 사실만 확인했다면 발행 성공으로 잘못 기록했을 것이다. live route와 feed, sitemap을 확인한 뒤에야 날짜가 공개 조건에 영향을 준다는 것을 찾았다.

두 번째는 The Noise Filter Article 004를 발행했을 때였다. 네 번째 글이 공개됐는데 homepage 소개 문구에는 여전히 `Three`가 남아 있었다. article route, RSS, sitemap이 모두 정상이어도 독자가 처음 보는 문장 하나는 오래된 상태를 말하고 있었다. build는 숫자의 의미를 모른다. 공개 corpus 수와 설명 문구를 함께 봐야 했다.

세 번째는 모바일 표였다. Phase 2의 비교 matrix는 desktop에서 잘 보였지만 390px와 320px에서는 page 전체가 옆으로 밀리지 않고 table container 안에서만 가로 scroll이 돼야 했다. HTML에 표가 존재한다는 검사나 CSS build만으로 손가락으로 읽을 수 있는지 알 수 없었다. light/dark와 여러 폭에서 browser QA를 따로 했다.

네 번째는 출처 확인이었다. FTC의 공개 consumer page 두 개는 일반 브라우저에서 제목, 최종 URL, 관련 내용을 정상적으로 볼 수 있었지만 automated `curl` 요청에는 HTTP 403을 돌려줬다. 자동 검사 결과만 보면 broken source였고, 브라우저만 믿으면 반복 가능한 evidence가 약해졌다. 그래서 자동 접근 제한과 실제 공개 여부를 분리해 기록하고 publication-day browser recheck를 남겼다.

다섯 번째는 Windows와 OneDrive 환경의 Pagefind helper였다. production build의 생성 단계는 끝났지만 마지막 copy helper가 종료됐다. 생성된 source와 target 133개 파일의 SHA-256을 비교해 일치시킨 뒤 PowerShell fallback으로 복사했고, GitHub Actions의 깨끗한 환경에서 build가 다시 성공하는지 확인했다. 로컬 종료를 무시하지도 않았고, 한 환경의 helper 실패를 생성물 전체의 실패로 단정하지도 않았다.

이 사례들의 공통점은 간단하다. 자동 검사는 증거를 만들지만, 증거 하나가 공개 상태 전체를 대신하지는 않는다. source, generated output, deployment, live page, browser interaction은 서로 다른 층이다. 하나가 통과하면 다음 층을 볼 자격이 생길 뿐, 모든 층이 맞다고 선언할 수는 없다.

이 원칙은 이전의 더 넓은 회고인 [AI 블로그 3개를 만들고 나서야 보인 진짜 문제](/posts/project-ai-autoblog-operating-system-retrospective/)에서 운영체계 관점으로 정리했다. 이번 복구에서는 그 원칙이 “글이 존재한다”와 “글의 가치가 공개돼 있다”를 구분하는 데 쓰였다.

## 재심사를 눌렀지만 결과는 아직 모른다

Phase 1부터 3까지 마친 뒤, 14편은 모두 서로 다른 reader question과 public artifact, evidence state, overlap decision을 갖게 됐다. Local Search Readiness는 34개 항목을 통과했고, live audit는 DNS AAAA 한 건을 의도된 skip으로 남긴 채 84개 항목을 통과했다. sitemap XML과 text route는 51개로 맞았고, RSS와 Pagefind에는 공개 article 14편이 들어 있었다. 세 개의 Phase 3 글은 desktop, 390px, 320px의 light/dark 조합 18개에서 확인했다.

이 숫자들은 복구 작업이 계획한 범위대로 적용됐다는 evidence다. Google이 그 변화를 어떻게 평가할지는 말해 주지 않는다. 그래서 최종 dashboard의 문구도 “AdSense 승인 준비 완료”가 아니라 “재심사 제출 여부를 사람이 별도로 판단할 수 있는 상태”였다.

2026년 7월 19일 01:04 KST, 사용자가 계정 화면에서 문제가 해결됐다는 항목을 체크하고 `Request review`를 눌렀다. Codex가 계정 행동을 수행하거나 반복하지 않았다. 재심사 제출은 한 번으로 기록했다. 현재 프로젝트 상태는 `re-review submitted — pending`이다.

이 상태에는 꽤 많은 부정문이 필요하다. 제출은 승인이 아니다. 광고가 송출된다는 뜻도 아니고, indexing이나 ranking이 개선됐다는 뜻도 아니다. traffic과 revenue 변화도 확인하지 않았다. 정확한 이전 심사 snapshot과 Google 내부 판단은 여전히 모른다. 복구한 항목이 실제 거절 이유와 일치하는지조차 결과만으로 완전히 증명되지는 않을 수 있다.

심사 기간에는 큰 redesign, URL·slug·domain·canonical·taxonomy migration, 대량 발행, AdSense·analytics·affiliate·tracking 확장을 멈췄다. 사실 오류나 broken link처럼 독자에게 직접 피해를 주는 문제는 고칠 수 있지만, “더 좋아 보이게” 만들기 위한 큰 변화를 계속 쌓으면 어느 버전이 심사됐는지 더 흐려진다.

재심사 버튼은 복구의 마지막 증거가 아니라 외부 판단을 다시 요청한 한 번의 행동이었다. 이번 글도 결과를 기다렸다가 승인 사례로 포장하기 위해 쓰지 않았다. 결과가 나오기 전이라도 무엇을 관찰했고 어떤 경계에서 수정했는지는 사실로 기록할 수 있다.

## 이번 복구 뒤에 남긴 운영 원칙

첫째, 공개 글의 근거는 공개돼야 한다. 내부 claim table과 source note는 작성 과정에 필요하지만 독자는 그 존재를 알 수 없다. 특히 구독, 호환성, storage, account access처럼 바뀌는 정보는 공식 링크와 확인 시점을 글 안에서 볼 수 있어야 한다. 모든 문장에 각주를 붙이라는 뜻이 아니라, 중요한 판단이 어디에서 왔는지 추적 가능해야 한다는 뜻이다.

둘째, 반복 가능한 template과 반복되는 글은 다르다. source freshness, false experience, safety boundary를 확인하는 checklist는 반복돼야 한다. 하지만 독자가 보는 opening, heading, decision frame까지 매번 같을 필요는 없다. Template은 빠뜨리지 않기 위한 내부 scaffold이고, article structure는 독자의 실제 질문에 맞춰 달라져야 한다.

셋째, 직접 경험이 없어도 독창적인 편집 가치는 만들 수 있다. 써 보지 않은 제품을 사용한 척하는 대신 공식 문서를 비교하고, 서로 다른 제한을 한 표에 놓고, 선택을 멈춰야 할 조건을 만든다. 고유한 가치는 새 사실의 발명보다 근거 있는 구분과 정직한 한계에서 나올 수 있다.

넷째, 사람 검토는 사실 확인만으로 끝나지 않는다. 문장 하나가 맞는지뿐 아니라 이 글이 다른 글과 다른 질문에 답하는지, 독자가 읽은 뒤 어떤 결정을 더 잘할 수 있는지, 공개할 책임과 유지 비용을 감당할 수 있는지 봐야 한다. 14편을 한 편씩 승인했을 때 놓친 반복이 포트폴리오 감사에서는 보였다.

다섯째, AdSense recovery를 버튼 작업으로 다루지 않는다. 계정 화면에서 체크하고 제출하는 데는 몇 분이면 충분하지만, 그 전에 article value, trust surface, source freshness, responsive table, public discovery를 다시 설계하는 데 훨씬 긴 시간이 들었다. 그렇다고 이 과정을 승인 공식으로 만들 수는 없다. Google의 결과는 외부 상태이고, 이번에 만든 것은 그 결과와 별개로 유지할 편집 운영체계다.

14편이라는 숫자는 거절 전에도 같았고 재심사 제출 뒤에도 같다. 달라진 것은 각 글을 남겨 둘 이유를 설명하는 방식이다. 다섯 편은 서로 다른 decision artifact를 갖게 됐고, 세 편은 겹치던 역할을 갈라 놓았다. 책임 주체와 correction path, research method는 공개 페이지에 나타났다. build 이후에는 browser와 live surface를 별도 evidence로 보게 됐다.

이 변화가 AdSense 승인을 가져올지는 아직 모른다. 지금 확실히 말할 수 있는 것은 하나다. 길고 문법적으로 맞으며 기술적으로 정상인 글도, 독자에게 왜 존재해야 하는지 보여 주지 못할 수 있다. 거절 화면의 한 줄은 그 질문을 피하지 못하게 만들었고, 답은 더 많은 글이 아니라 이미 있는 14편을 서로 다른 이유로 존재하게 만드는 쪽에 있었다.
