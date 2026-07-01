---
title: "수익형 블로그 글 7개를 쌓고 나서 워크스페이스 구조를 다시 정리했다"
categories:
  - Project AI Autoblog
  - Dev Log
tags:
  - AI
  - Autoblog
  - Smart Suburbanite
  - Workspace
  - Workflow
  - QA
  - Dev Log
description: "The Smart Suburbanite 글 7개 발행 이후, 글 품질뿐 아니라 README, source of truth, 폴더 역할까지 반복 가능한 운영 흐름의 일부로 정리한 Dev Blog 초안."
published: false
---

# 수익형 블로그 글 7개를 쌓고 나서 워크스페이스 구조를 다시 정리했다

## 글 7개가 되자 다른 문제가 보이기 시작했다

The Smart Suburbanite에 공개 글이 7개까지 쌓였다.

아직 이 숫자로 수익, 트래픽, SEO 성과를 말할 단계는 아니다. 지금 확인한 것은 더 작고 실무적인 쪽에 가깝다. 글을 기획하고, 검수하고, 발행하고, 발행 후 다시 고치는 흐름을 몇 번 반복하자 콘텐츠 품질만큼이나 워크스페이스 구조가 중요해지기 시작했다.

처음에는 AI가 글을 얼마나 잘 쓰는지가 눈에 먼저 들어왔다. 그런데 실제로는 topic, draft, review, image, publish, QA, RSS, browser freshness, internal links, deploy, tracking docs가 이어져야 한 편의 글이 운영 흐름 안에 들어온다.

글이 하나나 둘일 때는 파일 위치를 대충 기억할 수 있다. 일곱 편이 되자 그렇게 하면 다음 작업자가 어디부터 봐야 하는지 흔들린다.

## Article 007은 발행 후 복구가 필요했다

Article 007인 `Video Doorbells With No Monthly Fee: What to Look For`는 helper/free model workflow를 함께 써서 발행까지 갔다. 발행 자체는 되었지만, 그 뒤에 다시 봐야 할 것들이 남아 있었다.

- prompt 예시의 `pubDatetime`과 `modDatetime`이 실제 발행 시각처럼 남아 있지 않은가
- exact term을 보존하려다가 본문 영어 대문자 사용이 어색해지지 않았는가
- HQ final publish tracking이 실제 공개 상태와 맞는가
- source freshness와 risk review가 실제로 끝났는가

이건 무료 모델을 쓰면 안 된다는 결론은 아니다. 보조 모델은 초안, 요약, 검토 보조에 쓸 수 있다. 다만 발행 단계에서는 작은 누락이 곧 public site 상태와 tracking 문서의 불일치로 이어진다.

그래서 Article 007 이후에는 발행 자체보다 발행 후 repair pass가 더 선명하게 남았다.

## 글 다음에는 워크스페이스도 봐야 했다

발행 흐름이 길어질수록 파일이 어디에 쌓이는지가 중요해진다.

`docs`, `content_lab`, `prompts`, `dev_blog`, `monetized_blogs`는 각각 역할이 다르다. 그런데 README나 index 문서가 얇으면 폴더 구조가 맞더라도 실제 작업자는 매번 멈칫하게 된다.

어떤 문서가 current snapshot인지, 어떤 파일이 운영 명령 모음인지, Dev Blog draft는 어디에 두는지, Smart Suburbanite site repo는 언제 건드리면 안 되는지 같은 기준이 빠르게 보여야 한다.

글이 늘어난 뒤에는 좋은 초안만으로는 부족했다. 다음 작업을 다시 시작할 수 있는 지도가 필요했다.

## 구조가 크게 틀린 것은 아니었다

이번 workspace audit에서 확인한 핵심은 의외로 단순했다.

폴더 배치가 크게 잘못된 것은 아니었다. `docs`는 project-level decisions와 status를 담고, `content_lab`은 article planning과 publishing records를 담고, `prompts`는 reusable workflow를 담고, site repo들은 각각 독립된 repo로 남아 있는 구조가 맞았다.

문제는 구조 자체라기보다 안내 부족에 가까웠다. 일부 README가 없거나 얇었고, 일부 status text는 오래된 상태를 가리키고 있었다. 그러면 실제 폴더는 맞는 자리에 있어도 다음 작업자는 어디를 source of truth로 봐야 할지 헷갈린다.

이런 혼란은 자동화 이전에도 비용이 된다. 나중에 자동화를 얹으면 더 커질 수 있다.

## README와 source of truth를 보강했다

그래서 이번에는 파일을 크게 옮기지 않았다.

대신 README와 index 문서를 보강했다. root README, docs README, content_lab README, prompts README, scripts README, assets README, content_lab/monetized README, docs/milestones README가 각자의 역할을 더 분명히 말하도록 정리했다.

workspace structure audit 문서도 추가했다. 이 문서는 "지금 당장 폴더를 갈아엎자"는 제안이 아니라, 현재 구조가 어떤 의도로 유지되고 있고 어디서 헷갈림이 생겼는지 기록하는 용도다.

이번 정리의 결론은 보수적이었다.

지금은 파일 이동보다 안내 보강이 먼저다. 폴더를 옮기는 일은 반복적인 혼란이 더 쌓인 뒤에 해도 늦지 않다.

## 자동화 대상이 글쓰기만은 아니었다

처음에는 자동화라고 하면 AI가 글을 쓰고, 파일을 만들고, 발행하는 모습을 떠올리기 쉽다.

하지만 일곱 편을 지나오면서 더 중요한 자동화 대상이 보이기 시작했다. 글을 빨리 쓰는 것보다, 같은 기준으로 다음 작업을 시작하고 끝낼 수 있게 만드는 일이 더 중요했다.

반복 가능한 운영 흐름에는 최소한 이런 것들이 들어간다.

- 현재 상태를 빠르게 읽는 snapshot
- 반복 명령을 모아 둔 operating commands
- draft, review, publish, QA의 단계 구분
- site repo와 HQ repo의 역할 분리
- 발행 후 public state와 tracking docs의 일치 확인
- helper model을 쓴 뒤의 repair QA

Project AI Autoblog는 점점 autoblog script라기보다 AI-assisted content operations system에 가까워지고 있다. 이 차이는 작지 않다.

자동화의 목표가 "글을 자동으로 채우기"라면 위험한 shortcut이 많아진다. 반대로 목표가 "사람이 검수할 수 있는 반복 운영 흐름을 덜 흔들리게 만들기"라면 다음에 무엇을 자동화해야 하는지도 조금 더 분명해진다.

## 다음 단계

다음에는 다시 Smart Suburbanite 글 작업으로 돌아갈 수 있다.

다만 이제는 새 글 하나만 보지 않으려고 한다. 그 글이 어떤 review packet을 거쳤는지, 어떤 source/risk check를 통과했는지, 발행 후 public site에서 어떻게 확인되었는지, 그리고 HQ 문서가 실제 상태를 제대로 따라가고 있는지까지 같이 봐야 한다.

이번 정리는 큰 구조 개편이 아니었다. 하지만 일곱 번째 글 이후에 필요한 종류의 정리였다. 콘텐츠가 늘어날수록, 글 자체만큼이나 그 글을 둘러싼 운영 지도가 중요해진다.
