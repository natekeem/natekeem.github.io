---
title: "Claude Code로 예쁜 다이어그램 그리기, Diagram Design"
categories: [AI & Tech, Claude]
tags:
  - AI
  - Web Dev
  - Dev Log
description: "AI가 그려주는 촌스러운 다이어그램에서 벗어나는 법. 브랜드 컬러와 타이포그래피를 반영해 단일 HTML+SVG로 깔끔하게 떨어지는 오픈소스 스킬 Diagram Design의 원리와 공식 갤러리 분석."
date: 2026-10-20 14:00:00 +0900
published: true
image:
  path: /assets/img/posts/2026-10-20-diagram-design-skill-for-claude-code/cover.jpg
  alt: "Diagram Design 공식 아키텍처 다이어그램 템플릿 실물 렌더링"
---

블로그 글이나 기술 문서를 작성하다 보면, 백 마디 말보다 잘 그린 아키텍처 다이어그램 하나가 훨씬 강력할 때가 많다. 

하지만 AI 도구에게 다이어그램을 그려달라고 하면 십중팔구 천편일률적인 결과물이 나온다. 둥글넓적한 회색 사각형에 촌스러운 청록색 선, 그리고 기계 번역된 폰트가 어우러진 전형적인 'Mermaid 슬롭(Slop)'이다. 읽는 사람 입장에서는 회사 주간 보고용 발표 자료를 복사해 붙여넣은 것 같아 몰입이 깨지기 일쑤다.

캐서린 레이블리(Cathryn Lavery)가 공개한 오픈소스 에이전트 스킬인 **`cathrynlavery/diagram-design`**은 정확히 이 지점을 파고든 도구다. 

Claude Code, Cursor, Copilot 같은 AI 코딩 에이전트에 붙여서, 기술 문서나 테크 블로그에 곧바로 게재할 수 있는 에디토리얼급 다이어그램을 생성해준다. 이 도구의 작동 방식과 공식 갤러리 구성, 그리고 실제 템플릿의 기술적 구조를 살펴봤다.

---

## 1. Diagram Design이란 무엇인가?

Diagram Design은 단순한 프롬프트 모음이 아니라, AI 에이전트 규격(Agent Skills)을 따르는 오픈소스 다이어그램 생성 모듈이다.

기존 AI 생성 시각자료의 고질적 한계는 두 가지였다.  
1. **Mermaid**: 가볍고 마크다운 렌더링이 쉽지만, 시각적 커스텀이 극히 제한적이고 레이아웃이 종종 엉킨다.
2. **draw.io / Excalidraw**: 미적 감각은 좋지만, 별도의 데스크톱 툴을 켜서 마우스로 드래그해야 하고 Git 버전 관리나 자동화 파이프라인과 따로 논다.

Diagram Design은 이 간극을 메우기 위해 **단일 자체 완결형(Self-contained) HTML + SVG** 방식을 채택했다. 별도의 번들러나 무거운 자바스크립트 런타임 종속성 없이 브라우저에서 바로 열리고, 어떤 마크다운 렌더러나 정적 사이트에도 즉시 고해상도로 삽입할 수 있는 표준 코드를 뱉어낸다.

### 공식 설치 및 연동 방법

Claude Code 환경에서는 표준 스킬 추가 명령어 또는 플러그인 마켓플레이스를 통해 바로 설치할 수 있다:

```bash
# npx를 통한 에이전트 스킬 설치
npx skills add cathrynlavery/diagram-design

# 또는 Claude Code 플러그인 마켓플레이스 등록
/plugin marketplace add cathrynlavery/diagram-design
/plugin install diagram-design@diagram-design
```

설치하고 나면 Claude Code는 40여 종 이상의 다이어그램 구조(시스템 아키텍처, 배포 파이프라인, 시퀀스, 스윔레인, 사분면 매트릭스, 결정 트리 등)와 타이포그래피 규칙을 자동으로 컨텍스트에 탑재한다.

---

## 2. 공식 온라인 갤러리와 40여 종 다이어그램 카탈로그

Diagram Design의 가장 큰 자산은 공식 GitHub Pages로 배포된 방대한 실물 갤러리다.

![Diagram Design 공식 온라인 갤러리 인터페이스](/assets/img/posts/2026-10-20-diagram-design-skill-for-claude-code/01-diagram-design-official-gallery.png)

공식 갤러리([cathrynlavery.github.io/diagram-design](https://cathrynlavery.github.io/diagram-design/))에 접속하면, 에이전트가 어떤 다이어그램을 그릴 수 있는지 실시간으로 탐색할 수 있다:

- **상단 탭 카테고리**: 아키텍처(Architecture), 흐름도(Flowchart), 시퀀스(Sequence), 상태 머신(State Machine), 엑소노메트릭(Axonometric), 비스웜(BeeSwarm), 사분면(Quadrant), 조직도(Org Chart) 등 40여 종의 전문 시각화 구조를 탭별로 둘러볼 수 있다.
- **3대 테마 모드**: 동일한 다이어그램을 **Light**, **Dark**, 그리고 고급 잡지 스타일의 **Full-Editorial** 모드로 즉시 전환하여 비교할 수 있다.
- **Specimen Icons 라이브러리**: 기술 블로그에 필수적인 인라인 SVG 테크 아이콘과 브랜드 실루엣이 내장되어 있어 외부 에셋 없이 일관된 비주얼을 유지한다.

---

## 3. Mermaid 및 draw.io와의 3가지 핵심 차별점

![Diagram Design 공식 템플릿(example-architecture) 구조와 특징](/assets/img/posts/2026-10-20-diagram-design-skill-for-claude-code/02-diagram-design-official-sample-architecture.png)

### 1) 브랜드 시스템 자동 동기화 (Paper & Ink Palette)
공식 프로젝트의 템플릿 코드(`skills/diagram-design/assets/example-architecture.html`)를 열어보면, CSS 변수를 활용한 미니멀 디자인 시스템이 돋보인다:
- `--paper: #f5f4ed;` (따뜻한 미색 종이 질감의 캔버스)
- `--ink: #0b0d0b;` (선명하고 깊은 먹색 텍스트)
- `--accent: #f7591f;` (절제된 오렌지 포인트 컬러)
- 폰트 페어링: 제목에는 클래식한 `Instrument Serif`, 본문과 노드 라벨에는 모던한 `Geist` 및 `Geist Mono`를 매핑해, 전형적인 개발자 PPT 슬라이드가 아니라 Stripe나 Apple Docs를 연상시키는 에디토리얼 테크 룩을 완성한다.

### 2) 제로 종속성 단일 파일 출력 (Pure Self-Contained SVG)
외부 CDN 자바스크립트나 무거운 렌더러에 의존하지 않는다. 모든 노드, 화살표, 라벨이 순수 인라인 SVG 벡터로 구성되어 있어, 지킬(Jekyll)이나 아스트로(Astro) 같은 정적 사이트에서 404 에러나 렌더링 딜레이 없이 2x Retina 품질로 선명하게 표시된다.

### 3) 기존 자산 재작성(Import) 지원
이미 프로젝트에 작성해둔 Mermaid(`.mmd`) 코드나 draw.io XML 파일을 넘겨주면, 기존 논리 흐름을 그대로 살린 채 Diagram Design의 고유 레이아웃과 폰트 스타일로 변환해준다. 기존 문서의 못생긴 다이어그램을 마우스 작업 없이 에이전트 명령 한 줄로 리뉴얼할 수 있다.

---

## 4. 라이선스와 실제 사용 시 주의할 점

Diagram Design은 자유로운 상업적 이용과 수정이 가능한 **MIT 라이선스**를 따른다. 개인 기술 블로그는 물론 사내 테크니컬 문서화에도 법적 제약 없이 도입할 수 있다.

다만 실제 도입을 고려할 때 알아두어야 할 기술적 제약도 있다:
- **반응형 텍스트 길이 제한**: 노드 안에 들어가는 텍스트가 지나치게 길어지면 SVG `viewBox` 경계를 넘어 글자가 잘리는 클리핑 현상이 생길 수 있다. 노드 타이틀은 10~15자 내외의 명료한 단어로 다듬어야 시각적 균형이 유지된다.
- **정적 시각화 전용**: 툴팁 호버나 클릭 드릴다운 같은 복잡한 대화형(Interactive) 기능은 지원하지 않는다. 순수한 정적 벡터 다이어그램 출력에 집중하는 것이 도구의 강점을 극대화하는 길이다.

AI에게 그림을 그려달라고 할 때 가장 중요한 건 '그럴듯함'이 아니라 '문서의 결'에 맞느냐다. 화려한 3D 장식 대신 정보 전달에 집중된 담백한 에디토리얼 다이어그램이 필요하다면, Diagram Design은 Claude Code 워크플로우에 충분히 추가해볼 만한 오픈소스 스킬이다.

---

## 5. 핵심 공식 링크

- **공식 GitHub 저장소**: [`cathrynlavery/diagram-design`](https://github.com/cathrynlavery/diagram-design)
- **공식 온라인 갤러리**: [cathrynlavery.github.io/diagram-design](https://cathrynlavery.github.io/diagram-design/)
- **공식 스킬 명세**: `skills/diagram-design/assets/`
