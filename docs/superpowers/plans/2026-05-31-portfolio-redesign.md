# 포트폴리오 리디자인 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 허유경의 다크/Three.js 콘셉트 포폴을, 화이트톤 젠틀몬스터 스타일 에디토리얼 단일 페이지(`index.html`)로 전면 재구축하고 GitHub Pages로 배포한다.

**Architecture:** CDN-only 단일 정적 HTML. 상단 영상 히어로 → INTRO → WORK(7개 프로젝트, JS 데이터 배열로 인덱스 리스트 + 풀스크린 상세 패널 렌더) → ABOUT → CONTACT 순 단일 페이지 스크롤. 모션은 Lenis(관성 스크롤) + GSAP/ScrollTrigger(스크롤 연동 리빌·카운트업·내비 전환)로 통일.

**Tech Stack:** HTML + Tailwind CSS(Play CDN) + GSAP 3.12 + ScrollTrigger + Lenis 1.x, Google Fonts(Urbanist) + Pretendard(CDN). 빌드 없음. 배포: GitHub Pages.

**검증 방식:** 단위 테스트 없음(기존 repo 관례). 각 태스크는 `python3 -m http.server 8000` 으로 `http://localhost:8000/index.html` 을 열어 **시각 확인 + 콘솔 에러 0** 으로 검증한다.

**참고:** 프로필 사진·프로젝트 썸네일은 사용자가 추후 제공. 들어갈 자리는 `/assets/` 경로의 placeholder로 구축하고, 교체 지점을 주석으로 표시한다.

---

## File Structure

| 파일 | 책임 |
|------|------|
| `index.html` (신규) | 사이트 전체 — head(Tailwind config·폰트·CDN), body 마크업 5개 섹션 + 내비 + 상세 패널 컨테이너, 하단 단일 `<script>`(Lenis/GSAP init, PROJECTS 데이터, 렌더·모션 로직) |
| `Soft_pink_flowers_and_grass_sw.mp4` (기존) | 히어로 배경 영상 |
| `/assets/` (신규 폴더) | 프로젝트 썸네일·프로필 사진 placeholder(추후 사용자 자산으로 교체) |
| `portfolio.html` (기존) | 구버전 — 작업 완료 후 삭제 |
| `CLAUDE.md` (기존) | 디자인 시스템 섹션을 화이트톤 기준으로 갱신(마지막 태스크) |

단일 HTML 제약(CLAUDE.md)을 유지한다. WORK 7개 프로젝트의 반복 마크업은 하단 `<script>`의 `PROJECTS` 데이터 배열 하나에서 인덱스 리스트와 상세 패널을 함께 렌더해 DRY를 지킨다.

---

## Task 1: 스캐폴드 — 새 index.html + CDN + 화이트톤 토큰 + 스크롤 엔진 + git

**Files:**
- Create: `index.html`
- Create: `assets/.gitkeep`

- [ ] **Step 1: git 저장소 초기화 (Pages 배포 대비)**

Run:
```bash
cd /Users/yugang/XENONIX/project/IICOMBINED
git init
printf "node_modules/\n.DS_Store\n" > .gitignore
mkdir -p assets && touch assets/.gitkeep
```
Expected: `Initialized empty Git repository ...`

- [ ] **Step 2: index.html 작성 (head + 비어있는 body + 스크롤 엔진 부트스트랩)**

Create `index.html`:
```html
<!DOCTYPE html>
<html lang="ko" class="scroll-smooth">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>YUGYEONG HEO · AI · Full-Stack Developer</title>
<meta name="description" content="허유경 — AI · 풀스택 개발자 포트폴리오" />

<!-- Fonts -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link href="https://fonts.googleapis.com/css2?family=Urbanist:ital,wght@0,300;0,400;0,500;0,700;0,900;1,300;1,400&display=swap" rel="stylesheet" />
<link rel="stylesheet" as="style" crossorigin href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.css" />

<!-- Tailwind Play CDN -->
<script src="https://cdn.tailwindcss.com"></script>
<script>
  tailwind.config = {
    theme: {
      extend: {
        colors: {
          paper: '#F5F3EF',   // 웜 화이트 기본 배경
          snow:  '#FFFFFF',   // 순백 교차 섹션
          ink:   '#0A0A0A',   // 잉크 블랙 텍스트
          ash:   '#6B6B6B',   // 보조 회색
          line:  '#E2DED7',   // 헤어라인
        },
        fontFamily: {
          display: ['Urbanist', 'sans-serif'],
          kr: ['Pretendard', 'sans-serif'],
        },
      },
    },
  };
</script>

<!-- Motion: GSAP + ScrollTrigger + Lenis -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>
<script src="https://unpkg.com/lenis@1.1.14/dist/lenis.min.js"></script>

<style>
  html, body { margin: 0; padding: 0; }
  body { background: #F5F3EF; color: #0A0A0A; font-family: 'Pretendard', sans-serif; -webkit-font-smoothing: antialiased; }
  /* reveal 기본 상태 */
  .reveal { opacity: 0; transform: translateY(28px); }
  .label { font-size: 10px; letter-spacing: 0.4em; text-transform: uppercase; color: #6B6B6B; }
</style>
</head>
<body class="font-kr">

  <main id="main">
    <!-- 섹션은 이후 태스크에서 추가 -->
    <section class="h-screen flex items-center justify-center">
      <span class="label">SCAFFOLD OK</span>
    </section>
  </main>

  <!-- ============= SCRIPT ============= -->
  <script>
  (() => {
    'use strict';
    gsap.registerPlugin(ScrollTrigger);

    /* ---------- Lenis 관성 스크롤 + ScrollTrigger 연동 ---------- */
    const lenis = new Lenis({ duration: 1.1, smoothWheel: true });
    lenis.on('scroll', ScrollTrigger.update);
    gsap.ticker.add((time) => lenis.raf(time * 1000));
    gsap.ticker.lagSmoothing(0);

    /* ---------- 공용 reveal 유틸 ---------- */
    window.__reveal = () => {
      gsap.utils.toArray('.reveal').forEach((el) => {
        gsap.to(el, {
          opacity: 1, y: 0, duration: 1, ease: 'power3.out',
          scrollTrigger: { trigger: el, start: 'top 85%', once: true },
          delay: parseFloat(el.dataset.delay || 0),
        });
      });
    };
    window.__reveal();
  })();
  </script>
</body>
</html>
```

- [ ] **Step 3: 브라우저 검증**

Run: `python3 -m http.server 8000` → open `http://localhost:8000/index.html`
Expected: 웜 화이트 배경에 "SCAFFOLD OK" 중앙 표시. DevTools 콘솔 **에러 0**. `window.Lenis`, `window.gsap`, `window.ScrollTrigger` 가 정의됨(콘솔에서 확인).

- [ ] **Step 4: Commit**

```bash
git add index.html assets/.gitkeep .gitignore
git commit -m "feat: scaffold white-tone portfolio shell with Lenis+GSAP"
```

---

## Task 2: 고정 상단 내비 + 스크롤 진행 바 + 부드러운 앵커 스크롤

**Files:**
- Modify: `index.html` (body 상단에 `<header>` 추가, script에 내비 로직 추가)

- [ ] **Step 1: 내비 마크업 추가** — `<main id="main">` 바로 위에 삽입:

```html
<!-- 스크롤 진행 바 -->
<div id="progress" class="fixed top-0 left-0 h-px bg-ink z-[60]" style="width:0%"></div>

<!-- 고정 상단 내비 -->
<header id="nav" class="fixed top-0 inset-x-0 z-50 transition-all duration-500">
  <div class="flex items-center justify-between px-6 md:px-12 py-5">
    <a href="#hero" class="nav-link font-display font-700 tracking-[0.02em] text-lg">HEO YUGYEONG</a>
    <nav class="flex items-center gap-6 md:gap-10 text-xs tracking-[0.25em] uppercase font-display">
      <a href="#work" class="nav-link hover:opacity-60 transition-opacity">Work</a>
      <a href="#about" class="nav-link hover:opacity-60 transition-opacity">About</a>
      <a href="#contact" class="nav-link hover:opacity-60 transition-opacity">Contact</a>
    </nav>
  </div>
</header>
```

- [ ] **Step 2: 내비 색 전환용 CSS 추가** — `<style>` 블록에 추가:

```css
/* 영상 위: 흰 글씨 / 스크롤 후: 검정 글씨 + 흰 배경 */
#nav .nav-link { color: #FFFFFF; }
#nav.solid { background: rgba(245,243,239,0.85); backdrop-filter: blur(12px); border-bottom: 1px solid #E2DED7; }
#nav.solid .nav-link { color: #0A0A0A; }
```

- [ ] **Step 3: 내비 전환 + 진행 바 + 앵커 스크롤 로직 추가** — script IIFE 안 `__reveal()` 호출 다음에:

```javascript
    /* ---------- 내비: 히어로 지나면 solid 전환 ---------- */
    ScrollTrigger.create({
      start: 'top -80', end: 99999,
      onUpdate: (self) => {
        const nav = document.getElementById('nav');
        nav.classList.toggle('solid', self.scroll() > window.innerHeight * 0.85);
      },
    });

    /* ---------- 스크롤 진행 바 ---------- */
    const bar = document.getElementById('progress');
    lenis.on('scroll', ({ scroll, limit }) => {
      bar.style.width = (limit ? (scroll / limit) * 100 : 0) + '%';
    });

    /* ---------- 앵커 클릭 → Lenis 부드러운 스크롤 ---------- */
    document.querySelectorAll('a[href^="#"]').forEach((a) => {
      a.addEventListener('click', (e) => {
        const id = a.getAttribute('href');
        if (id.length > 1) { e.preventDefault(); lenis.scrollTo(id, { offset: 0 }); }
      });
    });
```

- [ ] **Step 4: 브라우저 검증**

reload → Expected: 내비가 상단 고정. (현재 히어로 없으니) 스크롤 시 배경이 흰색+블러로 바뀌고 글씨가 검정으로 전환, 상단 진행 바가 늘어남. 콘솔 에러 0.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: sticky nav with scroll transition, progress bar, smooth anchors"
```

---

## Task 3: HERO — 풀스크린 영상 + 워드마크 오버레이 + 스크롤 모션

**Files:**
- Modify: `index.html` (placeholder 섹션을 hero로 교체, script에 hero 모션 추가)

- [ ] **Step 1: hero 마크업으로 교체** — `<main>` 안의 placeholder `<section>` 을 다음으로 교체:

```html
<!-- ① HERO -->
<section id="hero" class="relative h-[100svh] w-full overflow-hidden">
  <video id="hero-video" class="absolute inset-0 w-full h-full object-cover"
         autoplay muted loop playsinline>
    <source src="Soft_pink_flowers_and_grass_sw.mp4" type="video/mp4" />
  </video>
  <div class="absolute inset-0 bg-black/20"></div>

  <div id="hero-copy" class="relative h-full flex flex-col items-center justify-center text-center text-white px-6">
    <div class="label text-white/80 mb-6">AI · Full-Stack Developer</div>
    <h1 class="font-display font-300 tracking-[-0.02em] leading-[0.9] text-[16vw] md:text-[11vw]">
      YUGYEONG<br /><span class="font-700">HEO</span>
    </h1>
  </div>

  <div class="absolute bottom-8 inset-x-0 flex flex-col items-center text-white/80 text-[10px] tracking-[0.4em] uppercase">
    <span class="mb-2">Scroll</span>
    <span class="block w-px h-10 bg-white/50 animate-pulse"></span>
  </div>
</section>
```

- [ ] **Step 2: hero 스크롤 모션 추가** — script의 앵커 로직 다음에:

```javascript
    /* ---------- HERO: 스크롤 시 영상 줌 + 카피 페이드 ---------- */
    gsap.to('#hero-video', {
      scale: 1.15, ease: 'none',
      scrollTrigger: { trigger: '#hero', start: 'top top', end: 'bottom top', scrub: true },
    });
    gsap.to('#hero-copy', {
      opacity: 0, y: -60, ease: 'none',
      scrollTrigger: { trigger: '#hero', start: 'top top', end: 'bottom top', scrub: true },
    });
```

- [ ] **Step 3: 브라우저 검증**

reload → Expected: 핑크꽃 영상이 풀스크린 자동재생(무음·루프). 중앙에 `YUGYEONG HEO` 흰 워드마크. 스크롤하면 영상이 살짝 확대되고 카피가 위로 페이드. 내비는 흰 글씨로 영상 위에 떠 있음. 콘솔 에러 0.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: fullscreen video hero with scroll-driven zoom and fade"
```

---

## Task 4: INTRO — 선언문 타이포 + 줄 단위 리빌

**Files:**
- Modify: `index.html` (hero `</section>` 다음에 intro 추가)

- [ ] **Step 1: intro 마크업 추가** — hero 섹션 닫는 태그 다음:

```html
<!-- ② INTRO -->
<section id="intro" class="bg-paper px-6 md:px-12 py-32 md:py-48">
  <div class="max-w-5xl mx-auto">
    <div class="label reveal mb-10">Introduction</div>
    <p class="font-display font-300 leading-[1.15] tracking-[-0.01em] text-3xl md:text-5xl lg:text-6xl text-ink">
      <span class="reveal inline-block">AI 기술 트렌드를 빠르게 습득하고,</span><br />
      <span class="reveal inline-block" data-delay="0.08">새로운 기술 스택과 개발 방법론에</span><br />
      <span class="reveal inline-block" data-delay="0.16">적극적으로 도전하며 끊임없이 성장하는</span><br />
      <span class="reveal inline-block font-700" data-delay="0.24">신입 개발자 허유경입니다.</span>
    </p>
  </div>
</section>
```

- [ ] **Step 2: 브라우저 검증**

reload → 스크롤해서 intro 진입 시 4줄이 아래에서 떠오르며 순차(스태거) 페이드인. 마지막 줄 굵게. 콘솔 에러 0.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: intro statement section with staggered reveal"
```

---

## Task 5: WORK 데이터 모델 + 인덱스 리스트

**Files:**
- Modify: `index.html` (intro 다음에 work 섹션 컨테이너 추가, script에 PROJECTS 데이터 + 리스트 렌더 추가)

- [ ] **Step 1: work 섹션 컨테이너 마크업 추가** — intro `</section>` 다음:

```html
<!-- ③ WORK -->
<section id="work" class="bg-snow px-6 md:px-12 py-32 md:py-48 border-t border-line">
  <div class="max-w-6xl mx-auto">
    <div class="flex items-end justify-between mb-16 md:mb-24">
      <h2 class="reveal font-display font-300 text-5xl md:text-7xl tracking-[-0.02em] text-ink">Selected<br /><span class="font-700">Work</span></h2>
      <div class="reveal label text-right">07 Projects<br />2024 — 2026</div>
    </div>
    <ul id="work-list" class="border-t border-line"></ul>
  </div>
</section>
```

- [ ] **Step 2: PROJECTS 데이터 배열 추가** — script IIFE 맨 위(`gsap.registerPlugin` 다음)에:

```javascript
    /* ---------- 프로젝트 데이터 (포폴 MD 기준) ---------- */
    const PROJECTS = [
      {
        no: '01', title: 'en.sedaily.com', kr: '글로벌 뉴스 AEO·SEO 최적화 플랫폼',
        role: 'Full-Stack · AI 번역 · AEO/SEO', year: '2025',
        tags: ['Next.js', 'FastAPI', 'AWS Lambda', 'Claude'],
        summary: 'LLM 검색 엔진에서 직접 추천되는 구조를 구축한 글로벌 뉴스 미디어 최적화 플랫폼. 키워드 기반 분류 시스템으로 활성 사용자·페이지뷰 급증, 전 세계 10개국+ 유입.',
        metrics: [
          { value: 9.03, suffix: '만', label: '총 클릭수' },
          { value: 2092, suffix: '만', label: '총 노출수' },
          { value: 42.2, suffix: '%', label: 'ChatGPT 유입' },
        ],
        bullets: [
          'AI 번역 시스템 구축 및 다국어 콘텐츠 파이프라인',
          'AEO·SEO 최적화 로직으로 Organic Search 유입 확보',
          'AWS Lambda 기반 확장성 있는 글로벌 서비스 구현',
          'AdSense 광고 최적화 및 수익화 기반 마련',
        ],
        stack: 'TypeScript · Python · Next.js · FastAPI · React · Tailwind · AWS(EC2·Lambda·S3·DynamoDB) · Claude Opus',
        link: 'https://en.sedaily.com',
      },
      {
        no: '02', title: 'AI KEY', kr: '한국 상장사·재계 AI 분석 프리미엄 서비스',
        role: 'Full-Stack · AI 파이프라인 · 인프라', year: '2026',
        tags: ['Next.js 14', 'AWS', 'DynamoDB', 'ElasticSearch'],
        summary: '한국 상장사·재계 그룹·산업 섹터를 AI가 분석·추적하는 영문 프리미엄 서비스. 글로벌 투자자용 인텔리전스 플랫폼.',
        metrics: [
          { value: 102, suffix: '개', label: '대기업집단' },
          { value: 85, suffix: '+', label: '상장사 분석' },
          { value: 99.9, suffix: '%', label: '데이터 가용성' },
        ],
        bullets: [
          'Next.js 14 기반 회사·그룹·섹터·시장 등 14개 분석 페이지 구축',
          'DART 전자공시 자동 수집 파이프라인 — 공시 수집·재무 추출·실적 예측',
          'KFTC 동향 감시 API 및 그룹/회사 히스토리 아카이브 구현',
          '팔로우·구독 기반 개인화 뉴스레터 자동 발송 에이전트 개발',
          'NextAuth 인증 + AWS Lambda 서버리스 백엔드 설계',
        ],
        stack: 'TypeScript · Python · Next.js 14(SSR) · DynamoDB · ElasticSearch · Redis · AWS(Lambda·API GW·CloudFront) · Claude(Bedrock)',
        link: 'https://en.sedaily.com/key',
      },
      {
        no: '03', title: 'AI 카드뉴스 CMS', kr: '영문 기사 → 한글 인스타 카드뉴스 자동 변환',
        role: 'Full-Stack · AI 프롬프트 · 인프라 (단독)', year: '2026',
        tags: ['Next.js 15', 'Bedrock', 'satori', 'resvg'],
        summary: '영문 기사 URL을 한글 인스타그램 카드뉴스로 자동 변환하는 풀스택 CMS. 데이터 수집부터 비주얼·카피라이팅까지 전 과정 자동화.',
        metrics: [
          { value: 90, suffix: '%↓', label: '수작업 시간 단축' },
          { value: 8, suffix: '종', label: '디자인 템플릿' },
          { value: 1080, suffix: 'px', label: '카드 렌더 해상도' },
        ],
        bullets: [
          'AWS Bedrock Claude 기반 기사 분석·번역·카드 콘텐츠 생성 파이프라인',
          'satori·resvg 기반 1080×1350 인스타 카드 PNG 렌더링 엔진 직접 개발',
          '뷰티·머니·썸네일 등 8종 템플릿 시스템 + 편집 UI',
          'en.sedaily 데이터 연동 키워드 검색 → 클릭 한 번 카드 생성',
          'Gemini TTS 보이스오버 기능 구현',
        ],
        stack: 'TypeScript · Next.js 15 · React 19 · SST · OpenNext · DynamoDB · AWS(Lambda·CloudFront·S3·Bedrock) · satori/resvg/sharp · Gemini TTS',
        link: 'https://www.instagram.com/look.edits.mag/',
      },
      {
        no: '04', title: 'AI NEXUS', kr: '기자용 올인원 AI 어시스턴트 (서울경제 X AWS)',
        role: 'AWS 비즈니스 로직 · 다중 AI 모델 통합', year: '2025',
        tags: ['React', 'AWS Lambda', 'Serverless', 'Claude'],
        summary: '현직 기자들의 뉴스 생산성 향상을 위한 올인원 AI 어시스턴트. 서울경제신문 내부 시스템으로 실제 도입·운영 중.',
        metrics: [
          { value: 110, suffix: 'M+', label: '누적 입력 토큰' },
          { value: 4, suffix: '명', label: '개발 인원' },
        ],
        bullets: [
          '뉴스 제목 생성·본문 교열 등 기자 맞춤형 챗봇 서비스',
          '다중 AI 모델 통합 및 실시간 스트리밍 처리',
          '서울경제신문 내부 데스크 업무에 실제 도입',
          'B2B 구독형 AI 솔루션 확장 전략 수립·추진',
        ],
        stack: 'JavaScript · Python · Node.js · React · Vite · Tailwind · Serverless Framework · AWS(Lambda·CloudFront·S3·Cognito·DynamoDB) · Claude',
        link: '',
      },
      {
        no: '05', title: 'Hover AI', kr: '시각장애인 터치스크린 사용 도우미 (CV + TTS)',
        role: '팀장 · 데이터 · YOLO11 · OpenCV', year: '2025',
        tags: ['YOLO11', 'EasyOCR', 'PyTorch', 'OpenCV'],
        award: 'SKT FLY AI 7기 프로젝트 부문 최우수상 · SK AI SUMMIT 2025',
        summary: '손가락 위치와 버튼 텍스트를 인식해 음성 안내하는 시각장애인용 가전 조작 도우미. 5명 협업, 팀장 담당.',
        metrics: [
          { value: 91, suffix: '%', label: 'OCR 정확도 (19%→91%)' },
          { value: 82, suffix: '%', label: 'Confidence (37%→82%)' },
          { value: 134010, suffix: '장', label: '수집·증강 이미지' },
        ],
        bullets: [
          'YOLO11 파인튜닝: fingertip 클래스 추가로 손가락 끝 탐지',
          'EasyOCR 가전 특화 파인튜닝: 정확도 19% → 91%',
          '조작 모드: 손가락–버튼 텍스트 매칭 후 TTS 안내',
          '보기 모드: LED 검출 + 텍스트 매칭으로 현재 동작 음성 안내',
        ],
        stack: 'Python · YOLO11 · PyTorch · OpenCV · Roboflow · Ultralytics · EasyOCR · TTS API · HuggingFace Spaces',
        link: 'https://huggingface.co/spaces/yugangee/Hover_AI',
      },
      {
        no: '06', title: '축구 분석 중계 시스템', kr: 'Computer Vision + RAG 해설 자막',
        role: 'Frontend · RAG 자막 · YOLOv8 · 배포', year: '2024–25',
        tags: ['YOLOv8', 'DeepSORT', 'BM25', 'FAISS'],
        award: '교내 AI융합학부 IT 경진대회 최우수상',
        summary: '선수·공·심판을 추적하고 RAG로 자연스러운 중계 해설 자막을 생성하는 축구 분석 시스템. 4명 협업.',
        metrics: [
          { value: 4, suffix: '클래스', label: '객체 탐지 단위' },
          { value: 0.6, suffix: 'α', label: 'Hybrid 가중치' },
        ],
        bullets: [
          'YOLOv8 객체 탐지 + DeepSORT ID 지속 추적',
          'K-Means 색상 군집화로 팀 자동 분류',
          '유클리드 거리 기반 공 소유자 판별',
          'BM25 + FAISS 하이브리드 검색 → OpenAI로 해설 자막 생성',
        ],
        stack: 'Python · JavaScript(React) · FastAPI · YOLOv8 · OpenCV · DeepSORT · Sentence-BERT · BM25 · FAISS · HuggingFace Spaces',
        link: 'https://huggingface.co/spaces/yugangee/football_analysis',
      },
      {
        no: '07', title: 'Fashion Jiok', kr: '사용자 스타일 기반 매칭 애플리케이션',
        role: 'Backend · AI (Agent + 분류 모델)', year: '2025',
        tags: ['React Native', 'Express', 'MySQL', 'CLIP'],
        summary: 'AI Agent 대화 추천과 CLIP 기반 스타일 분류를 결합한 스타일 매칭 앱. 3명 협업, 백엔드·AI 담당.',
        metrics: [
          { value: 78, suffix: '%', label: '스타일 분류 정확도' },
          { value: 0.84, suffix: 'F1', label: 'Lovely 클래스 최고' },
        ],
        bullets: [
          'Node.js 백엔드 서버 + MySQL DB 설계',
          'Gemini API 기반 대화 추천 Agent (첫 멘트·맥락 답변)',
          'EfficientNet/CLIP 기반 여성 스타일 분류 모델 개발 (78%)',
        ],
        stack: 'JavaScript(Node.js·React Native/Expo) · Python · Express.js · Sequelize · MySQL · Gemini · EfficientNet · CLIP · PyTorch',
        link: 'https://github.com/Fashion-Jiok/fashion-jiok',
      },
    ];
```

- [ ] **Step 3: 인덱스 리스트 렌더 로직 추가** — PROJECTS 정의 다음, Lenis init 전에:

```javascript
    /* ---------- WORK 인덱스 리스트 렌더 ---------- */
    const workList = document.getElementById('work-list');
    workList.innerHTML = PROJECTS.map((p, i) => `
      <li class="work-row reveal border-b border-line group cursor-pointer" data-index="${i}">
        <div class="flex items-center gap-6 py-7 md:py-9">
          <span class="label w-10 shrink-0">${p.no}</span>
          <div class="flex-1 min-w-0">
            <div class="font-display font-500 text-2xl md:text-4xl tracking-[-0.01em] text-ink group-hover:translate-x-2 transition-transform duration-500">${p.title}</div>
            <div class="font-kr text-sm text-ash mt-1">${p.kr}</div>
          </div>
          <span class="hidden md:block label shrink-0">${p.role}</span>
          <span class="label w-14 text-right shrink-0">${p.year}</span>
          <span class="shrink-0 text-ink group-hover:rotate-45 transition-transform duration-500">↗</span>
        </div>
      </li>`).join('');
```

- [ ] **Step 4: 브라우저 검증**

reload → WORK 섹션에 7개 프로젝트가 번호·제목·한글설명·역할·연도 행으로 나열. 행 호버 시 제목이 살짝 오른쪽 이동 + ↗ 화살표 45° 회전. 행이 스크롤 진입 시 리빌. 콘솔 에러 0.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: work section data model and editorial index list"
```

---

## Task 6: WORK 상세 — 풀스크린 패널 (열기/닫기 + 콘텐츠 렌더)

**Files:**
- Modify: `index.html` (body 끝 `<script>` 위에 패널 컨테이너 추가, script에 패널 로직 추가)

- [ ] **Step 1: 패널 컨테이너 마크업 추가** — `</main>` 다음, `<!-- SCRIPT -->` 위:

```html
<!-- WORK 상세 풀스크린 패널 -->
<div id="panel" class="fixed inset-0 z-[70] bg-paper translate-y-full invisible overflow-y-auto" aria-hidden="true">
  <button id="panel-close" class="fixed top-5 right-6 md:right-12 z-[71] text-sm tracking-[0.25em] uppercase font-display hover:opacity-60">Close ✕</button>
  <div id="panel-body" class="min-h-screen px-6 md:px-12 py-24 md:py-32"></div>
</div>
```

- [ ] **Step 2: 패널 렌더/열기/닫기 로직 추가** — 인덱스 리스트 렌더 로직 다음:

```javascript
    /* ---------- WORK 상세 패널 ---------- */
    const panel = document.getElementById('panel');
    const panelBody = document.getElementById('panel-body');

    function renderPanel(p) {
      const metricsHtml = p.metrics.map((m) => `
        <div class="border-t border-line pt-4">
          <div class="count font-display font-700 text-4xl md:text-5xl text-ink" data-value="${m.value}" data-suffix="${m.suffix}">0</div>
          <div class="label mt-2">${m.label}</div>
        </div>`).join('');
      const bulletsHtml = p.bullets.map((b) => `<li class="flex gap-3 text-ink/80 leading-relaxed"><span class="text-ash">—</span><span>${b}</span></li>`).join('');
      const awardHtml = p.award ? `<div class="inline-block label border border-ink px-3 py-2 mt-4">★ ${p.award}</div>` : '';
      const linkHtml = p.link ? `<a href="${p.link}" target="_blank" rel="noopener" class="inline-flex items-center gap-2 mt-8 text-sm tracking-[0.2em] uppercase font-display border-b border-ink pb-1 hover:opacity-60">View Live ↗</a>` : '';
      return `
        <div class="max-w-5xl mx-auto">
          <div class="label mb-4">${p.no} / ${p.year}</div>
          <h2 class="font-display font-300 text-4xl md:text-6xl tracking-[-0.02em] text-ink">${p.title}</h2>
          <p class="font-kr text-lg text-ash mt-3">${p.kr}</p>
          ${awardHtml}
          <!-- IMAGE PLACEHOLDER: 추후 assets/${p.no}.jpg 로 교체 -->
          <div class="my-12 aspect-[16/9] w-full bg-line/60 flex items-center justify-center label">IMAGE · ${p.title}</div>
          <p class="font-kr text-lg md:text-xl leading-relaxed text-ink/90 max-w-3xl">${p.summary}</p>
          <div class="grid grid-cols-2 md:grid-cols-3 gap-6 my-12">${metricsHtml}</div>
          <h3 class="label mb-5">Role & Contribution</h3>
          <ul class="space-y-3 mb-12">${bulletsHtml}</ul>
          <h3 class="label mb-3">Tech Stack</h3>
          <p class="font-kr text-sm text-ash leading-relaxed">${p.stack}</p>
          ${linkHtml}
        </div>`;
    }

    function openPanel(i) {
      panelBody.innerHTML = renderPanel(PROJECTS[i]);
      panel.classList.remove('invisible');
      panel.setAttribute('aria-hidden', 'false');
      lenis.stop();
      gsap.to(panel, { y: '0%', duration: 0.7, ease: 'power3.out' });
      // 카운트업
      panelBody.querySelectorAll('.count').forEach((el) => {
        const val = parseFloat(el.dataset.value);
        const suffix = el.dataset.suffix || '';
        const isInt = Number.isInteger(val);
        gsap.fromTo(el, { innerText: 0 }, {
          innerText: val, duration: 1.4, ease: 'power2.out', delay: 0.3,
          snap: { innerText: isInt ? 1 : 0.1 },
          onUpdate: function () {
            const n = parseFloat(el.innerText);
            el.innerText = (isInt ? Math.round(n).toLocaleString() : n.toFixed(1)) + suffix;
          },
        });
      });
    }

    function closePanel() {
      gsap.to(panel, { y: '100%', duration: 0.5, ease: 'power3.in', onComplete: () => {
        panel.classList.add('invisible');
        panel.setAttribute('aria-hidden', 'true');
        panelBody.innerHTML = '';
        lenis.start();
      }});
    }

    workList.querySelectorAll('.work-row').forEach((row) => {
      row.addEventListener('click', () => openPanel(parseInt(row.dataset.index, 10)));
    });
    document.getElementById('panel-close').addEventListener('click', closePanel);
    document.addEventListener('keydown', (e) => { if (e.key === 'Escape' && panel.getAttribute('aria-hidden') === 'false') closePanel(); });
```

- [ ] **Step 3: 브라우저 검증**

reload → WORK 행 클릭 시 풀스크린 패널이 아래에서 슬라이드업. 제목·한글·(수상작은 어워드 배지)·이미지 placeholder·요약·지표 **카운트업 애니메이션**·역할 불릿·기술스택·(있으면)View Live 링크 표시. Close 버튼 또는 ESC로 닫으면 슬라이드다운, 배경 스크롤 잠금/해제 정상. 7개 모두 확인. 콘솔 에러 0.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: fullscreen project detail panel with metric count-up"
```

---

## Task 7: ABOUT — 프로필·학력·활동·자격증·기술스택

**Files:**
- Modify: `index.html` (work 섹션 다음에 about 추가)

- [ ] **Step 1: about 마크업 추가** — work `</section>` 다음:

```html
<!-- ④ ABOUT -->
<section id="about" class="bg-paper px-6 md:px-12 py-32 md:py-48 border-t border-line">
  <div class="max-w-6xl mx-auto grid md:grid-cols-12 gap-10 md:gap-16">
    <!-- 프로필 사진 placeholder: 추후 assets/profile.jpg 로 교체 -->
    <div class="md:col-span-4 reveal">
      <div class="aspect-[3/4] w-full bg-line/60 flex items-center justify-center label">PROFILE PHOTO</div>
      <div class="mt-6 font-display font-700 text-2xl text-ink">허유경</div>
      <div class="label mt-1">Heo Yugyeong · 2001.07.17</div>
    </div>

    <div class="md:col-span-8 space-y-12">
      <div class="reveal">
        <div class="label mb-3">About</div>
        <p class="font-display font-300 text-2xl md:text-4xl leading-[1.2] tracking-[-0.01em] text-ink">
          AI와 풀스택을 가로지르며, 데이터 수집부터 모델·프론트엔드·인프라까지<br class="hidden md:block" /> 직접 만들어 굴리는 신입 개발자.
        </p>
      </div>

      <div class="reveal grid sm:grid-cols-2 gap-8">
        <div>
          <div class="label mb-4 border-b border-line pb-2">Education</div>
          <p class="text-ink leading-relaxed">성신여자대학교 (2021–2026.02)<br /><span class="text-ash">주전공 AI학과 · 복수전공 경제학과</span></p>
        </div>
        <div>
          <div class="label mb-4 border-b border-line pb-2">Activities</div>
          <ul class="text-ink/80 space-y-1 text-sm leading-relaxed">
            <li>BDAA 3기 (2022)</li>
            <li>이데일리 W페스타 · 아태영리더스 포럼 서포터즈 (2023)</li>
            <li>사이드나우 2기 (2024)</li>
            <li>SKT FLY AI 7기 · SK AI SUMMIT 2025</li>
          </ul>
        </div>
      </div>

      <div class="reveal">
        <div class="label mb-4 border-b border-line pb-2">Certifications & Awards</div>
        <ul class="text-ink/80 grid sm:grid-cols-2 gap-x-8 gap-y-1 text-sm">
          <li>SQLD (2025.06)</li>
          <li>TOEIC 800 (2025.10)</li>
          <li>OPIc IM2 (2025.07)</li>
          <li class="text-ink font-500">★ SKT FLY AI 7기 프로젝트 최우수상</li>
          <li class="text-ink font-500">★ 교내 IT경진대회 최우수상</li>
        </ul>
      </div>

      <div class="reveal">
        <div class="label mb-4 border-b border-line pb-2">Tech Stack</div>
        <div id="skills" class="flex flex-wrap gap-2"></div>
      </div>
    </div>
  </div>
</section>
```

- [ ] **Step 2: 스킬 칩 렌더 추가** — script의 패널 로직 다음:

```javascript
    /* ---------- ABOUT 스킬 칩 ---------- */
    const SKILLS = ['Next.js','React','React Native','TypeScript','JavaScript','Python','Node.js','FastAPI','Express','Tailwind CSS','AWS Lambda','DynamoDB','S3','EC2','Bedrock','Claude','Gemini','YOLO','OpenCV','PyTorch','RAG','MySQL'];
    const skills = document.getElementById('skills');
    skills.innerHTML = SKILLS.map((s) => `<span class="border border-line text-ink/80 text-sm px-3 py-1.5 rounded-full hover:border-ink hover:text-ink transition-colors">${s}</span>`).join('');
```

- [ ] **Step 3: 브라우저 검증**

reload → ABOUT: 좌측 프로필 사진 placeholder + 이름, 우측 About 문장·Education·Activities·Certifications/Awards(수상 2건 강조)·Tech Stack 칩들. 칩 호버 시 테두리 진해짐. 스크롤 리빌 동작. 콘솔 에러 0.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: about section with profile, education, awards, skill chips"
```

---

## Task 8: CONTACT + 푸터 + 테크 키워드 마퀴

**Files:**
- Modify: `index.html` (about 다음에 contact 추가, `<style>`에 마퀴 CSS)

- [ ] **Step 1: 마퀴 CSS 추가** — `<style>` 블록에:

```css
.marquee { overflow: hidden; white-space: nowrap; }
.marquee-track { display: inline-flex; gap: 3rem; animation: marquee 30s linear infinite; }
@keyframes marquee { from { transform: translateX(0); } to { transform: translateX(-50%); } }
```

- [ ] **Step 2: contact 마크업 추가** — about `</section>` 다음:

```html
<!-- 테크 키워드 마퀴 -->
<div class="marquee bg-snow border-y border-line py-6">
  <div class="marquee-track font-display font-300 text-2xl md:text-4xl text-ink/30 tracking-tight">
    <span>AI Engineering</span><span>·</span><span>Full-Stack</span><span>·</span><span>Computer Vision</span><span>·</span><span>RAG</span><span>·</span><span>AWS Serverless</span><span>·</span><span>LLM Pipelines</span><span>·</span>
    <span>AI Engineering</span><span>·</span><span>Full-Stack</span><span>·</span><span>Computer Vision</span><span>·</span><span>RAG</span><span>·</span><span>AWS Serverless</span><span>·</span><span>LLM Pipelines</span><span>·</span>
  </div>
</div>

<!-- ⑤ CONTACT -->
<section id="contact" class="bg-paper px-6 md:px-12 py-32 md:py-48">
  <div class="max-w-6xl mx-auto">
    <div class="label reveal mb-8">Contact</div>
    <h2 class="reveal font-display font-300 text-5xl md:text-8xl tracking-[-0.03em] leading-[0.95] text-ink mb-12">
      Let's build<br /><span class="font-700">something.</span>
    </h2>
    <a href="mailto:dbruddl56@naver.com" class="reveal inline-block font-display font-500 text-2xl md:text-4xl text-ink border-b-2 border-ink pb-1 hover:opacity-60 transition-opacity">dbruddl56@naver.com</a>
    <div class="reveal mt-12 flex flex-wrap gap-x-10 gap-y-4 text-sm tracking-[0.15em] uppercase font-display">
      <a href="https://www.linkedin.com/in/yugyeongheo/ko/" target="_blank" rel="noopener" class="hover:opacity-60">LinkedIn ↗</a>
      <a href="https://github.com/yugangee" target="_blank" rel="noopener" class="hover:opacity-60">GitHub ↗</a>
      <a href="https://huggingface.co/yugangee" target="_blank" rel="noopener" class="hover:opacity-60">HuggingFace ↗</a>
      <a href="https://blog.naver.com/dbruddl56" target="_blank" rel="noopener" class="hover:opacity-60">Blog ↗</a>
      <span class="text-ash">Tel. 010-7337-5827</span>
    </div>

    <footer class="mt-32 pt-8 border-t border-line flex flex-col md:flex-row justify-between gap-3 label">
      <span>© 2026 Heo Yugyeong</span>
      <span>AI · Full-Stack Developer Portfolio</span>
      <span>Seoul, KR</span>
    </footer>
  </div>
</section>
```

- [ ] **Step 3: 브라우저 검증**

reload → 마퀴 키워드가 좌측으로 끊김 없이 흐름. CONTACT: 대형 "Let's build something." + 이메일 링크(밑줄) + LinkedIn/GitHub/HuggingFace/Blog/전화 + 푸터. 링크 새 탭 동작. 콘솔 에러 0.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: contact section, footer, and tech keyword marquee"
```

---

## Task 9: 로드 인트로 + 정리 + CLAUDE.md 갱신 + Pages 배포 안내

**Files:**
- Modify: `index.html` (로드 인트로)
- Delete: `portfolio.html`
- Modify: `CLAUDE.md`

- [ ] **Step 1: 로드 인트로 마크업 추가** — `<body>` 바로 다음 첫 요소로:

```html
<!-- 로드 인트로 -->
<div id="loader" class="fixed inset-0 z-[100] bg-paper flex items-center justify-center">
  <span class="font-display font-700 text-2xl tracking-[0.1em] text-ink opacity-0" id="loader-mark">HEO YUGYEONG</span>
</div>
```

- [ ] **Step 2: 로더 애니메이션 추가** — script IIFE 맨 끝(닫기 전):

```javascript
    /* ---------- 로드 인트로 ---------- */
    const tl = gsap.timeline();
    tl.to('#loader-mark', { opacity: 1, duration: 0.6, ease: 'power2.out' })
      .to('#loader-mark', { opacity: 0, duration: 0.4, delay: 0.5 })
      .to('#loader', { yPercent: -100, duration: 0.7, ease: 'power3.inOut', onComplete: () => {
        document.getElementById('loader').style.display = 'none';
      }});
```

- [ ] **Step 3: 구버전 삭제 + 검증**

Run:
```bash
git rm portfolio.html
python3 -m http.server 8000
```
open `http://localhost:8000/` (index.html 이 기본 로드되는지) → Expected: 로더가 떴다 사라진 뒤 전체 사이트가 정상 동작. 처음부터 끝까지 스크롤: 히어로 영상 → 인트로 → 7개 work(패널 열기/닫기) → about → 마퀴 → contact. 모바일 폭(개발자도구 반응형)에서도 레이아웃 깨짐 없음. 콘솔 에러 0.

- [ ] **Step 4: CLAUDE.md 디자인 시스템 갱신**

`CLAUDE.md` 의 "What this repo is", "Design system (do not drift)", "Architecture" 섹션을 새 화이트톤 단일 페이지 기준으로 교체한다. 핵심 교체 내용:
- 파일명: `portfolio.html` → `index.html`
- 색: 다크/네온 → 웜 화이트(`#F5F3EF`)·잉크(`#0A0A0A`)·모노크롬, 액센트 없음
- 스택: Three.js 제거 → Lenis + GSAP/ScrollTrigger
- 콘텐츠: 콘셉트 → 허유경 실제 프로젝트 7개(`PROJECTS` 데이터 배열)
- 배포: GitHub Pages

- [ ] **Step 5: 최종 커밋**

```bash
git add -A
git commit -m "feat: load intro, remove legacy portfolio.html, update CLAUDE.md"
```

- [ ] **Step 6: GitHub Pages 배포 (사용자와 함께)**

```bash
# GitHub에 빈 repo 생성 후
git branch -M main
git remote add origin https://github.com/yugangee/<repo>.git
git push -u origin main
# GitHub repo → Settings → Pages → Branch: main / root → Save
# 게시 URL: https://yugangee.github.io/<repo>/
```
Expected: 게시 URL에서 영상 포함 사이트 정상 로드.

---

## Self-Review

**1. Spec coverage:**
- ① HERO → Task 3 ✓ / ② INTRO → Task 4 ✓ / ③ WORK 인덱스 → Task 5 ✓ / ③ WORK 상세 패널 → Task 6 ✓ / ④ ABOUT → Task 7 ✓ / ⑤ CONTACT+푸터 → Task 8 ✓
- 내비·진행바·앵커 → Task 2 ✓ / 모션(Lenis·ScrollTrigger·리빌) → Task 1·전 태스크 ✓ / 카운트업 → Task 6 ✓ / 마퀴 → Task 8 ✓ / 로드 인트로 → Task 9 ✓
- 화이트톤 토큰·폰트 → Task 1 ✓ / 단일 HTML·CDN → 전체 ✓ / GitHub Pages → Task 9 ✓
- 사진 placeholder(추후 교체) → Task 6(프로젝트)·Task 7(프로필) ✓
- 중간 섹션 사진+모션 분산(사용자 추가 요구) → 각 섹션 reveal + 패널/about 이미지 placeholder로 자리 확보 ✓

**2. Placeholder scan:** 이미지 영역은 "추후 사용자 자산 교체"로 스펙에 명시된 의도적 placeholder(주석으로 교체 지점 표시). 코드 스텝은 모두 실제 동작 코드 포함, "TODO/TBD" 없음.

**3. Type consistency:** `PROJECTS` 객체 키(`no,title,kr,role,year,tags,summary,metrics{value,suffix,label},bullets,stack,award?,link`)를 Task 5(리스트)·Task 6(패널·카운트업)에서 동일하게 참조. `openPanel/closePanel/renderPanel`, `lenis`, `workList`, `panel/panelBody` 식별자 일관. `.count[data-value][data-suffix]` 카운트업 대상 일치.

**미해결(스펙 8과 동일):** 프로필/썸네일 이미지·영문 폰트 최종 선택은 사용자 입력 대기 — placeholder 및 Urbanist 기본값으로 진행, 자산 도착 시 교체.
