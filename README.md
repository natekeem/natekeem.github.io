# 나태킴의 커밋로그 (Dev Blog)

> **"AI 블로그 자동화 프로젝트를 차근차근 커밋하는 기록"**
> 1인 개발자의 AI 도구 실전 리뷰, 깃허브 오픈소스 트렌드, 자동화 블로그 운영 회고, 기술 글쓰기 기록을 담는 기술 블로그입니다.

- **공개 주소 (Canonical URL)**: [https://natekeem.com/](https://natekeem.com/)
- **저장소 (GitHub)**: [natekeem/natekeem.github.io](https://github.com/natekeem/natekeem.github.io)
- **작성자 / 표시 이름**: `나태킴` (`natekeem`)
- **엔진 및 테마**: Jekyll 4.x + Chirpy Theme
- **발행 현황**: 총 21편 발행 완료 (2026-10-16 기준)
- **서치콘솔 상태**: 8대 정예 토픽 허브 체제 (`sitemap.xml` 40개 URL 정합 수집 완료)

---

## 📚 주요 카테고리 및 연재 시리즈

1. **AI & Tech**: 최신 AI 에이전트, LLM 툴체인 실전 리뷰 및 **[Git Trending AI]** 주간 큐레이션 (순위 엄격 정렬, 1200×675 실기 터미널 캡처, 즉시 실행 CLI, Before/After 지표, 실무 유스케이스 2선 제공)
2. **Dev Log**: 프롬프트 하네스 트러블슈팅, 사이트 인프라 개선, 서치콘솔/애드센스 문제 해결, 자동화 운영 회고
3. **Web Dev**: Astro/Jekyll 테마 튜닝, 반응형 CSS, 폰트 및 에디토리얼 비주얼 시스템 구축
4. **Project Autoblog**: Project AI Autoblog 포트폴리오 거버넌스 및 운영 기록

---

## 🛠️ 로컬 개발 및 빌드 (Local Development)

```bash
# 의존성 설치
bundle install

# 로컬 개발 서버 실행 (포트 4000)
bundle exec jekyll serve --livereload

# 배포 빌드 테스트
JEKYLL_ENV=production bundle exec jekyll build
```

---

## 📜 콘텐츠 운영 및 시각화 규범 (Editorial Standards)

- **인간 승인 필수 (Human Gate)**: 모든 글은 AI 초안 후 사람의 명시적 승인을 거쳐 발행됩니다.
- **실측 데이터 및 솔직한 목소리**: 허위 벤치마크를 배제하고, 실제 CLI 실행 결과, 트렌드시프트 실측치, 브라우저 캡처를 증빙하며 개발자 실무자 구어체로 작성합니다.
- **표지 이미지 탈정형화 (Anti-Sterile Cover)**: 진부한 노트북/모니터 앞 책상 스톡 사진을 지양하고, 생생한 감정 묘사(머리 감싸기, 페이스팜, 환호), 실제 작업 설비 현장, 또는 다크테크 랭킹 타이포그래피 포스터를 적용합니다.
- **본문 시각화 의무화**: 글만 가득한 텍스트 벽을 방지하기 위해 최소 2개 이상의 설명용 시각 자료(UI/터미널 캡처, 아키텍처 다이어그램 등)를 배치합니다.
- **외부 링크**: 외부 레퍼런스나 깃허브 링크는 `target="_blank" rel="noopener noreferrer"`를 적용합니다.
