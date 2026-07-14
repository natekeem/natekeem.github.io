---
title: "프롬프트를 줄였더니 QA 문서가 더 중요해졌다"
categories:
  - Project AI Autoblog
  - Dev Log
tags:
  - AI
  - Autoblog
  - Prompt Engineering
  - QA
  - Workflow
  - Dev Log
description: "The Smart Suburbanite Article 014 작업에서 긴 일회성 프롬프트를 줄이고 site profile, QA module, 저장소 경계, 이미지 정책, 최종 보고 규칙으로 품질 계약을 옮긴 과정을 기록한 프로젝트 로그."
date: 2026-07-14 21:26:43 +0900
published: true
image:
  path: /assets/img/posts/2026-07-14-short-prompts-need-qa-documents/cover.png
  alt: "짧은 작업 메모와 두꺼운 QA 문서가 함께 놓인 조용한 작업 책상"
---

# 프롬프트를 줄였더니 QA 문서가 더 중요해졌다

## 긴 프롬프트를 매번 다시 쓰지 않기로 했다

최근 The Smart Suburbanite의 Article 014를 만들면서 작업 지시가 눈에 띄게 짧아졌다. 예전 같으면 글의 톤, 금지할 주장, 이미지 규칙, 저장소 경계, 발행 후 확인 항목을 한 프롬프트에 길게 적었다. 이번에는 해야 할 단계를 짧게 지정하고, 이미 저장소에 있는 site profile과 QA module을 읽도록 했다.

처음에는 조금 불안했다. 프롬프트가 짧아지면 빠뜨리는 항목도 많아지는 것 아닐까. 특히 public post를 다루는 작업에서 "알아서 잘해 줘"에 가까운 지시는 품질 기준이 될 수 없다.

그래서 줄인 것은 품질 계약이 아니라 일회성 설명이었다. 긴 규칙은 사라진 것이 아니라 다음 문서로 옮겨 갔다.

- site profile에는 블로그의 역할, 독자, 현재 상태와 금지선을 적었다.
- reusable QA module에는 글, cover, inline image, public route, final report 검사를 나눠 두었다.
- repository boundary rule에는 HQ, Dev Blog, The Smart Suburbanite 중 어디까지 바꿀 수 있는지 적었다.
- cover policy에는 최근 표지와 겹치지 않는 방향, 금지할 시각 요소, 생성과 보정 횟수를 적었다.
- inline-image policy에는 이미지가 설명이나 증거 역할을 하지 못하면 넣지 않는다는 기본값을 뒀다.
- final-report requirement에는 실행한 검사, 건너뛴 검사와 이유, 변경 파일, commit, 남은 위험을 빠뜨리지 않게 했다.

짧아진 프롬프트 뒤에 더 긴 문서가 생긴 셈이다. 다만 이 문서는 한 번 쓰고 버리는 지시문이 아니라, 버전 관리되고 다음 작업에서도 다시 실행되는 운영 규칙이다.

## Article 014가 실제 시험대였다

Article 014 `Smart Sensors Explained: Motion, Contact, Leak, and Temperature`는 이 구조를 한 번에 시험한 사례였다. 작업은 한 번의 거대한 요청으로 끝나지 않았다.

먼저 brief package에서 독자 질문, 글의 범위, 출처 범주, 위험 주장을 정리했다. 다음 draft package에서는 English draft, Korean review packet, claim/fact table, review card를 함께 만들었다. human-review revision에서는 네 종류의 sensor를 똑같은 형식으로 반복하던 느낌을 줄이고, 실제 집 안의 질문과 전환으로 흐름을 다시 잡았다.

그 뒤 cover-ready package에서는 표지를 별도 검토 대상으로 다뤘다. 글 전체를 도식으로 설명하는 board나 네 개의 sensor를 나열하는 grid 대신, 조용한 utility 공간과 작은 awareness cue를 쓰는 방향을 정했다. site-ready publish 단계에서는 front matter, 공개 경로, cover 경로, RSS와 sitemap 노출, 금지한 claim과 script 추가 여부를 다시 확인했다.

각 단계가 나뉘어 있었기 때문에 짧은 요청도 "이전 검토는 생략하고 바로 발행하라"는 뜻이 되지 않았다. 단계마다 읽어야 할 문서와 통과해야 할 gate가 남아 있었다.

## 넣을 수 있는 이미지도 넣지 않았다

Article 014에서는 본문 inline image도 검토했다. laundry 또는 utility 장면을 하나 더 넣으면 글이 덜 빽빽해 보일 수 있었다. 하지만 실제로 따져 보니 그 이미지는 leak sensor 부분과 cover 장면을 반복할 가능성이 컸다. 독자의 이해를 넓히는 설명 자료도 아니고, 작업 결과를 입증하는 screenshot도 아니었다.

그래서 후보 이미지를 찾거나 다운로드하지 않고 cover-only로 끝냈다. 이 결정은 "본문 이미지는 나쁘다"는 일반론이 아니다. 이미지를 넣을 수 있다는 사실과 지금 글에 이미지가 필요하다는 판단은 다르다는 뜻이다.

여기서 QA 문서의 역할이 선명해졌다. 이미지 정책은 이미지 수를 늘리는 규칙이 아니라, 추가하지 않는 결정을 근거와 함께 남길 수 있는 규칙이어야 했다. Article 014에는 `후보 0개`, `다운로드 없음`, `cover-only`라는 기록이 남았다.

## 짧은 프롬프트는 품질 기준이 아니다

이번 변화 뒤에 `짧은 프롬프트가 더 좋은 결과를 만든다`고 결론 내리면 곤란하다. 짧은 프롬프트만으로는 작업자가 어떤 이름을 지켜야 하는지, 어떤 성공 주장을 피해야 하는지, 어느 저장소를 건드리면 안 되는지 알 수 없다.

지금 구조에서 짧은 요청이 작동하는 이유는 세 가지다.

첫째, site profile과 module이 현재 저장소 안에 있고 변경 이력이 남는다. 둘째, 작업 시작 전에 관련 문서를 실제로 읽도록 한다. 셋째, 발행 전후의 review gate와 최종 보고가 생략되지 않는다.

즉 `짧은 prompt + reusable module + review gate`가 하나의 작업 단위다. prompt만 짧게 만드는 것은 축약이고, 반복 규칙을 문서로 옮겨 실행하는 것은 표준화다. 둘은 비슷해 보이지만 결과는 다르다.

이 방식도 품질을 보장하지는 않는다. 문서가 오래되거나, module을 읽기만 하고 검사를 실행하지 않거나, 최종 판단을 서두르면 그대로 실패할 수 있다. 그래서 "어떤 module을 사용했는가"뿐 아니라 "어떤 check를 실제로 실행했는가"까지 마지막에 남겨야 한다.

## 여전히 사람이 맡아야 하는 것

문서가 늘어나도 이 프로젝트는 full automation이 아니다. 지금 단계에서 사람이 계속 결정해야 하는 일이 분명히 남아 있다.

- 어떤 topic을 지금 다룰지 고르는 일
- 초안이 실제 독자에게 자연스럽고 유용한지 편집하는 일
- 어떤 source를 신뢰할지, 어떤 claim을 빼거나 낮출지 판단하는 일
- cover가 글의 분위기와 위험 수준에 맞는지 승인하는 일
- 최종 publish 여부를 결정하는 일
- Search Console의 상태를 직접 확인하고 과장 없이 기록하는 일
- Google AdSense의 review 상태를 확인하고 approval과 준비 단계를 구분하는 일

특히 Search Console이나 Google AdSense는 public URL이 열리고 workflow가 성공했다는 이유만으로 결과를 대신 말해 주지 않는다. 이 글을 쓰는 시점에도 The Smart Suburbanite의 AdSense review 결과를 성공으로 해석하지 않는다. indexing, ranking, traffic, revenue도 마찬가지다.

AI가 더 짧은 요청으로 더 많은 단계를 수행하게 됐지만, publish decision까지 안전하게 넘겨받았다는 뜻은 아니다. 오히려 사람이 어디에서 판단해야 하는지를 문서에 더 분명하게 남긴 쪽에 가깝다.

## 다음에는 문서가 낡는 문제를 본다

다음 작업은 prompt를 더 줄이는 일이 아니다. site profile의 현재 글 수와 review 상태가 실제 프로젝트와 맞는지, module끼리 중복되거나 충돌하는 규칙은 없는지, 실행하지 않은 check가 보고서에서 빠지지 않는지 확인할 차례다.

짧은 요청이 편해질수록 문서의 최신성과 실행 기록이 더 중요해진다. 앞으로는 새로운 규칙을 계속 추가하는 것만큼, 기존 규칙을 정리하고 실제 작업에서 통과했는지 확인하는 유지보수도 함께 기록하려 한다.

이번에 줄어든 것은 검수량이 아니다. 사람이 매번 다시 쓰던 설명의 양이다. 품질을 지키는 일은 prompt 밖으로 사라진 것이 아니라, 저장소 안의 QA 문서와 review gate로 자리를 옮겼다.
