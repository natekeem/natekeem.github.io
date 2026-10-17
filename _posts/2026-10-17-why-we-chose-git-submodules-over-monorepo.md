---
title: "모노레포 버리고 깃 서브모듈로 갈아탄 이유 (1인 개발 아키텍처)"
categories:
  - Dev Log
  - Autoblog
tags:
  - Git
  - Web Dev
  - Architecture
  - Dev Log
description: "서로 다른 정적 사이트 3개를 한 프로젝트에서 굴릴 때 모노레포는 왜 지옥이 될까? Git Mode 160000 서브모듈 연합 구조로 의존성 지옥을 탈출하고 GitHub Actions 독립 배포를 달성한 실전 인프라 기록."
date: 2026-10-17 15:00:00 +0900
published: true
image:
  path: /assets/img/posts/2026-10-17-why-we-chose-git-submodules-over-monorepo/cover.jpg
  alt: "3개의 모듈형 큐브와 중앙 독으로 비유된 깃 서브모듈 아키텍처 커버 그래픽"
---

블로그가 세 개로 늘어난 날, 나는 모노레포(Monorepo)를 버렸다.

하나의 저장소에 모든 코드를 때려넣는 모노레포는 처음엔 편해 보인다. 커밋 하나로 전체 사이트의 문서를 고칠 수 있고, 저장소 주소도 하나만 외우면 되니까. 

하지만 서로 다른 기술 스택이 얽히는 순간, 모노레포는 1인 개발자에게 지옥을 선사한다:
- **Dev Blog**는 Ruby 3.3 기반의 **Jekyll Chirpy** 테마를 쓴다.
- **The Smart Suburbanite**와 **The Noise Filter**는 Node.js 22 기반의 **AstroPaper** 테마를 쓴다.

루트 디렉터리에 `Gemfile`과 `package.json`, 그리고 수백 메가바이트짜리 `node_modules`와 `vendor/bundle`이 뒤섞이기 시작했다. 한쪽 블로그의 npm 패키지를 업데이트했는데 엉뚱하게 다른 블로그의 GitHub Actions 배포 워크플로우가 트리거되어 빌드 에러를 뿜었다.

커밋 로그는 뒤죽박죽 엉켰고, 어느 사이트가 어떤 변경으로 깨졌는지 추적조차 불가능해졌다.

이 혼란을 단번에 정리해 준 열쇠가 바로 **깃 서브모듈(Git Submodule - Mode 160000)**이었다.

---

## 모노레포가 실패한 이유: 1인 개발자의 의존성 오염

소프트웨어 공학에서 "모노레포가 최신 트렌드"라는 말은 절반만 맞다. 

구글이나 메타처럼 통일된 단일 빌드 도구(Bazel, Buck)와 인프라 전담 엔지니어링 팀이 있을 때나 모노레포가 작동한다. 1인 개발자가 서로 다른 언어 런타임(Ruby vs Node.js)을 단일 깃 트리에서 굴리는 것은 재앙에 가깝다.

실제로 겪었던 세 가지 치명적 병목은 다음과 같았다:

1. **배포 워크플로우 오염**: 데브로그에 오타 하나 고치고 푸시했을 뿐인데, CI 러너 3개가 동시에 돌면서 불필요한 빌드 시간과 액션(Action) 분(Minutes)을 갉아먹었다.
2. **독립 도메인 및 CNAME 충돌**: `smartsuburbanite.com`이라는 독립 apex 도메인 설정과 `natekeem.com`의 GitHub Pages 설정이 단일 루트 저장소의 CNAME 파일 하나를 두고 계속 충돌했다.
3. **버전 롤백의 불가능**: 영문 블로그 하나에서 레이아웃 버그가 터졌을 때, 깃 리셋(Reset)을 때리면 멀쩡하던 국문 데브로그의 최신 포스트까지 함께 과거로 날아가는 아찔한 경험을 했다.

단일 저장소는 격리(Isolation)가 불가능했다. 

![HQ Workspace vs. Submodule Federation Architecture (Mode 160000)](/assets/img/posts/2026-10-17-why-we-chose-git-submodules-over-monorepo/01-submodule-federation-architecture.png)

---

## Git Mode 160000: 서브모듈 연합 구조의 실체

해결책은 명확했다. **"블로그는 철저히 독립된 3개의 깃 저장소로 쪼개고, 본부(HQ) 저장소는 이들을 포인터로 가리키기만 한다."**

깃 내부 아키텍처에서 서브모듈은 파일이나 폴더가 아니다. 

깃 트리를 `git ls-tree HEAD`로 조회해보면 흥미로운 숫자가 나타난다:

```bash
$ git ls-tree HEAD
100644 blob e69de29bb2d1...    AGENTS.md
040000 tree d41d8cd98f00...    docs
160000 commit 206dd96a324...   dev_blog
160000 commit c0c4aefa781...   monetized_blogs/the-smart-suburbanite
160000 commit 7c212c3f901...   monetized_blogs/the-noise-filter
```

일반 파일은 `100644`, 디렉터리는 `040000`이다. 그런데 서브모듈은 **`160000 commit`**이라는 특수한 깃 객체 모드로 기록된다.

이것이 의미하는 바는 결정적이다:
- 본부 저장소(`ai-autoblog`)는 하위 블로그의 수만 줄 소스 코드를 단 한 글자도 담고 있지 않다.
- 오직 **"원격 저장소의 정확한 특정 커밋 해시(SHA-1 40자리)"**만을 가리키는 나침반 포인터만 들고 있다.

이 구조 덕분에 세 블로그는 물리적으로 완벽히 분리되었다:
- `dev_blog`는 자기만의 저장소(`natekeem.github.io`)에서 순수 Ruby/Jekyll로 돌며 GitHub Pages로 직행한다.
- `the-smart-suburbanite`는 자기만의 저장소에서 Astro 5.0과 Pagefind 검색 인덱스를 빌드하며 Cloudflare/Vercel로 날아간다.
- 중앙 본부(`ai-autoblog`)는 전체 사이트의 프롬프트 템플릿, 에디토리얼 가이드라인, 팩트체크 기록, 그리고 자동화 스크립트만 중앙에서 통제한다.

---

## 서브모듈의 악명 높은 함정: Detached HEAD와 동기화

깃 서브모듈을 써본 개발자 열에 아홉은 고개를 절레절레 흔든다. *"서브모듈 그거 커밋 날아가고 detached HEAD 뜨는 지옥 아니냐?"*

맞다. 규칙 없이 다루면 서브모듈은 정말 무시무시하다. 

하지만 그 혼란은 단 하나의 원인에서 발생한다: **"본부에서 먼저 커밋을 치고, 서브모듈 푸시를 깜빡했을 때."**

이 실수를 물리적으로 차단하기 위해 워크플로우에 **'철칙(Golden Rule)'**을 못 박았다.

![The Golden Rule: Submodule Commit First, HQ Pointer Second](/assets/img/posts/2026-10-17-why-we-chose-git-submodules-over-monorepo/02-submodule-hygiene-workflow.png)

### 서브모듈 무결성을 지키는 3단계 순서:

1. **1단계 (서브모듈 안에서 작업 및 검증)**:
   - 해당 블로그 디렉터리로 들어가 로컬 빌드를 검증한다.
   ```bash
   cd monetized_blogs/the-smart-suburbanite
   npm run build
   ```
2. **2단계 (서브모듈 단독 커밋 & 단독 푸시)**:
   - 본부로 나오지 말고, 서브모듈 내부에서 먼저 커밋을 치고 원격 `origin/main`으로 푸시한다.
   ```bash
   git add .
   git commit -m "feat: publish article 022"
   git push origin main
   ```
   - 이 순간 해당 블로그의 독립 배포 파이프라인만 격리되어 실행된다.
3. **3단계 (본부로 복귀하여 포인터만 갱신)**:
   - 본부 루트로 올라와 `git status`를 치면 `m monetized_blogs/the-smart-suburbanite`가 뜬다. 방금 푸시한 커밋 해시로 포인터를 업데이트하라는 뜻이다.
   ```bash
   cd ../..
   git add monetized_blogs/the-smart-suburbanite
   git commit -m "chore: update suburbanite pointer to c0c4aef"
   git push origin main
   ```

이 **"서브모듈 먼저, 본부는 나중에"**라는 역방향 규칙만 지키면, 깃 서브모듈에서 커밋이 꼬이거나 미아 커밋(Detached HEAD)이 발생할 확률은 정확히 **0%**가 된다.

---

## 1인 개발자를 위한 아키텍처 판정 가이드

지금 여러 개의 사이트나 서비스를 개발 중이라면, 유행을 따르기 전에 냉정하게 질문을 던져보자:

```
[다중 프로젝트 아키텍처 선택 기준]
  ├─ 모든 프로젝트가 동일한 런타임(예: 순수 Next.js)과 동일한 팀인가?
  │   └─> 모노레포(Turborepo / Nx)를 쓰자.
  │
  └─ 프로젝트마다 기술 스택(Ruby vs Node vs Python)이 다르고 배포 도메인이 다른가?
      ├─ 완전 남남이고 문서나 룰셋을 공유할 필요가 전혀 없다?
      │   └─> 완전 독립 저장소 3개로 분리하자.
      │
      └─ 공통 문서, 프롬프트, AI 규칙, 스크립트를 중앙에서 통제하고 싶다?
          └─> 깃 서브모듈(Git Submodule Federation)이 정답이다.
```

거창한 유행에 속지 말자. 1인 개발자에게 가장 위대한 아키텍처는 **"한쪽이 터져도 다른 쪽에 불이 옮겨붙지 않는 완전한 격리"**다.

깃 서브모듈은 낡은 기술이 아니다. 규칙만 명확하다면, 1인 개발자가 다중 브랜드를 가장 깔끔하고 견고하게 굴릴 수 있는 최고의 엔지니어링 방화벽이다.
