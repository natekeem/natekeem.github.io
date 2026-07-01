---
title: "글이 쌓이자 워크스페이스 지도부터 다시 그렸다"
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
description: "The Smart Suburbanite 글 7개 발행 이후, 글 목록보다 먼저 작업 지도가 필요해졌다는 점을 정리한 Dev Blog 초안."
published: false
---

# 글이 쌓이자 워크스페이스 지도부터 다시 그렸다

## 이번에는 글보다 작업 공간을 먼저 봤다

이번에는 글을 하나 더 쓰는 대신, 작업 공간을 다시 봤다.

The Smart Suburbanite에 공개 글이 7개까지 쌓이면서 문제는 글감 부족이 아니라 운영 흐름을 다시 찾는 일이 됐다. 아직 수익이나 트래픽, SEO 성과를 말할 단계는 아니다. 하지만 반복 가능한 방식으로 글을 만들고, 검수하고, 발행하고, 발행 후 확인하는 구조가 점점 중요해지고 있다.

처음에는 AI가 글을 얼마나 잘 쓰는지가 중심 질문처럼 보였다. 그런데 몇 편을 실제로 발행해 보니 글쓰기만 따로 떨어져 있지 않았다. topic, draft, review, image, publish, QA, RSS, browser freshness, internal links, deploy, tracking docs가 이어져야 한 편의 글이 운영 흐름 안에 들어온다.

글이 늘어나면 글 목록보다 먼저 작업 지도가 필요해진다. 이번 정리는 그 사실을 확인한 작업에 가까웠다.

## Article 007에서 발행 후 복구가 필요했다

Article 007인 `Video Doorbells With No Monthly Fee: What to Look For`는 helper/free model workflow를 함께 써서 발행까지 갔다. 발행 자체는 되었지만, 그 뒤에 다시 봐야 할 것들이 남아 있었다.

- prompt 예시의 `pubDatetime`과 `modDatetime`이 실제 발행 시각처럼 남아 있지 않은가
- exact term을 보존하려다가 본문 영어 대문자 사용이 어색해지지 않았는가
- HQ final publish tracking이 실제 공개 상태와 맞는가
- source freshness와 risk review가 실제로 끝났는가

이건 보조 모델을 쓰면 안 된다는 결론은 아니다. 초안, 요약, 검토 보조에는 충분히 쓸 수 있다. 다만 발행 단계에서는 작은 누락이 public site 상태와 tracking 문서의 불일치로 이어진다. Article 007 이후에 남은 교훈도 그래서 "더 빨리 발행하자"가 아니라 "발행 후 repair pass를 운영 흐름에 넣자"에 가까웠다.

## 지금 워크스페이스는 이렇게 나뉘어 있다

이번에 다시 본 프로젝트 구조는 대략 이렇게 나뉜다.

```text
ai-autoblog/
├─ docs/                 # 프로젝트 상태, 운영 기준, 설계 기록, milestone
├─ prompts/              # 반복 작업용 프롬프트와 workflow 템플릿
├─ content_lab/          # 글 기획, 초안, 검수 패키지, 발행 기록
├─ dev_blog/             # Dev Blog 독립 repo, Chirpy/Jekyll 사이트
├─ monetized_blogs/      # 수익형 블로그 repo 모음
│  └─ the-smart-suburbanite/  # Smart Suburbanite AstroPaper 사이트
├─ scripts/              # 반복 작업을 도울 로컬 스크립트
├─ assets/               # HQ 기준 참고/공유 asset
├─ playbooks/            # 사람이 따라갈 운영 절차
└─ checklists/           # 재사용 가능한 QA 체크리스트
```

이 구조에서 중요한 점은 `dev_blog/`와 `monetized_blogs/the-smart-suburbanite/`가 그냥 폴더가 아니라 각각 독립된 site repo라는 점이다. HQ 문서 작업과 실제 공개 사이트 작업을 섞으면, 어떤 변경이 문서 기록인지 실제 배포인지 헷갈리기 쉽다.

`content_lab/`은 공개 사이트가 아니라 글이 만들어지는 작업실이다. `docs/`는 현재 상태와 운영 기준을 기록하는 곳이고, `prompts/`는 같은 작업을 다시 실행하기 위한 지시문을 모아두는 곳이다. 이 구분이 흐려지면 글 자체보다 "어디를 고쳐야 하지?"에서 시간이 빠진다.

## 문제는 폴더 구조보다 안내 부족이었다

workspace audit에서 확인한 핵심은 폴더 배치가 크게 잘못된 것이 아니라는 점이었다.

`docs`, `content_lab`, `prompts`, `playbooks`, `checklists`는 역할이 대체로 맞았다. `dev_blog`와 `monetized_blogs/the-smart-suburbanite`를 독립 site repo로 둔 것도 지금 단계에서는 맞는 구조다. `orchestrator`는 아직 미래 placeholder로 남겨 두는 편이 현재 MVP 범위와도 맞다.

헷갈림은 다른 곳에서 나왔다. 일부 README가 없거나 얇었고, 일부 status text는 오래된 상태를 가리키고 있었다. 그러면 실제 폴더는 맞는 자리에 있어도 다음 작업자는 어디를 source of truth로 봐야 할지 다시 추적해야 한다.

자동화를 붙이기 전에도 이런 안내 부족은 비용이 된다. 나중에 자동화를 붙이면 더 조용히 커지는 비용이 될 수 있다.

## 그래서 파일을 옮기지 않고 README부터 보강했다

이번 정리에서는 큰 파일 이동을 하지 않았다. 폴더를 갈아엎는 대신, 어디에 무엇이 있고 무엇이 아닌지를 먼저 보강했다.

- root README와 `docs/README.md`에 현재 상태와 workspace map을 더 분명히 적었다.
- `content_lab/`, `prompts/`, `scripts/`, `assets/`의 README에 폴더 역할을 보강했다.
- `content_lab/monetized/`와 `docs/milestones/`에는 새 README를 추가했다.
- `docs/WORKSPACE_STRUCTURE_AUDIT.md`에 현재 구조, source-of-truth map, 보류한 작업을 기록했다.

결론은 보수적이었다. 지금 필요한 것은 파일 이동이 아니라 안내 정리였다. 폴더를 옮기는 일은 반복적인 혼란이 더 쌓인 뒤에 해도 늦지 않다.

## 자동화는 글쓰기보다 운영 흐름을 덜 흔들리게 만드는 일에 가까웠다

처음에는 자동화라고 하면 AI가 글을 쓰고, 파일을 만들고, 발행하는 모습을 떠올리기 쉽다.

하지만 The Smart Suburbanite를 운영하면서 더 중요한 대상이 보이기 시작했다. 글을 빨리 쓰는 것보다, 같은 기준으로 다음 작업을 시작하고 끝낼 수 있게 만드는 일이 더 중요했다.

Project AI Autoblog는 점점 autoblog script라기보다 AI-assisted content operations system에 가까워지고 있다. 이 차이는 작지 않다.

자동화의 목표가 "글을 자동으로 채우기"라면 위험한 shortcut이 많아진다. 반대로 목표가 "사람이 검수할 수 있는 반복 운영 흐름을 덜 흔들리게 만들기"라면 다음에 무엇을 자동화해야 하는지도 조금 더 분명해진다.

## 다음 작업을 시작하기 쉽게 만드는 것이 목표다

다음에는 다시 Smart Suburbanite 글 작업으로 돌아갈 수 있다.

다만 이제는 새 글 하나만 보지 않으려고 한다. 그 글이 어떤 review packet을 거쳤는지, 어떤 source/risk check를 통과했는지, 발행 후 public site에서 어떻게 확인되었는지, 그리고 HQ 문서가 실제 상태를 제대로 따라가고 있는지까지 같이 봐야 한다.

이번 정리는 큰 구조 개편이 아니었다. 하지만 필요한 종류의 정리였다. 콘텐츠가 늘어날수록 글 자체만큼이나 그 글을 둘러싼 작업 지도가 중요해진다.
