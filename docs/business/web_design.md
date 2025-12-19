
# Monthly Games 웹 서비스 설계서
> 회사 홈페이지, 게임 서비스 페이지, 마케팅용 랜딩 페이지 통합 설계

---

## 1. 전체 구조 개요

### 1.1 목표

- **회사 홈페이지**: 브랜드/신뢰 허브, 회사 소개 및 포트폴리오 제공
- **게임 서비스 페이지**: 각 게임의 공식 정보/업데이트/이벤트 안내
- **마케팅 랜딩 페이지**: 설치/사전예약/이벤트 참여 등 “전환”에 특화된 페이지

### 1.2 도메인/사이트 구성

- 메인 도메인:
  - `monthly.games` – 회사/포트폴리오/블로그/게임 목록
- 서브도메인:
  - `play.monthly.games` – 게임별 웹 플레이어/WebGL 빌드 호스팅 (향후 확장용)
  - `go.monthly.games` – 마케팅 랜딩 전용

> **참고**: 게임 상세 페이지(`/games/[gameId]`)는 메인 도메인에서 제공하여 SEO와 브랜드 일관성을 유지한다.
> `play.monthly.games`는 웹 플레이 가능한 게임의 실제 플레이어 호스팅용으로 예약한다.

### 1.3 다국어 지원 전략

#### 지원 언어 (우선순위)

1. **한국어 (ko)** – 기본 언어
2. **영어 (en)** – 글로벌 기본
3. **일본어 (ja)** – 주요 타깃 시장
4. **중국어 간체 (zh-CN)** – 확장 시장

#### URL 구조

```text
monthly.games/ko/         # 한국어 홈
monthly.games/en/games    # 영어 게임 목록
monthly.games/ja/games/game_0001  # 일본어 게임 상세
go.monthly.games/game_0001_launch_ja  # 일본어 랜딩
```

#### 다국어 콘텐츠 파일 구조

```text
content/
  ├─ company.ko.json
  ├─ company.en.json
  ├─ company.ja.json
  ├─ games/
  │   ├─ game_0001.ko.json
  │   ├─ game_0001.en.json
  │   └─ game_0001.ja.json
  └─ landings/
      ├─ game_0001_launch_kr.json
      └─ game_0001_launch_jp.json
```

#### 구현 방식

- Next.js의 `next-intl` 또는 `next-i18next` 사용
- `Accept-Language` 헤더 기반 자동 감지 + 수동 전환 지원
- hreflang 태그로 검색엔진에 언어별 페이지 관계 명시
- 번역 관리: Crowdin 또는 Lokalise 연동 고려

### 1.4 웹 레포 구조

#### 1) 회사/서비스 사이트 레포 – `mg-web-main`

```text
mg-web-main/
  ├─ apps/web/             # Next.js (Vercel)
  │   ├─ pages/
  │   │   ├─ index.tsx     # 회사 홈페이지
  │   │   ├─ games/
  │   │   │   └─ [gameId].tsx  # 각 게임 서비스 페이지
  │   │   ├─ about.tsx
  │   │   ├─ careers.tsx
  │   │   ├─ support.tsx
  │   │   └─ blog/
  │   ├─ components/
  │   └─ lib/
  ├─ content/
  │   ├─ company.json       # 회사 홈/공통 설정
  │   ├─ games/
  │   │   └─ game_0001.json # 게임별 페이지 설정
  │   └─ blog/              # MDX 또는 JSON 기반 포스트
  └─ ...
```

#### 2) 랜딩 전용 레포 – `mg-web-landing`

```text
mg-web-landing/
  ├─ apps/landing/         # Next.js (Vercel)
  │   ├─ pages/
  │   │   └─ [slug].tsx    # 캠페인별 랜딩
  │   ├─ components/
  │   └─ lib/
  ├─ content/
  │   └─ landings/
  │       ├─ game_0001_launch_kr.json
  │       └─ game_0003_pre_reg_jp.json
  └─ ...
```

---

## 2. 회사 홈페이지 설계 (`monthly.games`)

### 2.1 정보 구조(IA)

상단 네비게이션:

1. **Home** – 메인, 최신 게임/비전/하이라이트
2. **Games** – 게임 리스트 + 각 게임 상세 링크
3. **About** – 회사 소개, 비전, 팀, 문화
4. **Careers** – 채용 정보
5. **Blog/News** – 개발/운영/서비스 소식
6. **Support** – FAQ, 문의/지원 채널

### 2.2 Home 페이지 섹션

1. **Hero**
   - 메시지: "한 달에 한 번, 새로운 플레이."
   - 서브 카피: Monthly Games의 방향성
   - CTA:
     - “게임 둘러보기” (/games)
     - “최신 게임 바로 플레이” (/games/game_0001 등)

2. **Featured Games**
   - 최근 출시/밀고 싶은 게임 1~3개 카드
   - 썸네일, 장르, 한 줄 설명, 플랫폼 버튼

3. **Monthly Lineup 타임라인**
   - game_0001~0012 (Year 1), 이후 게임까지 간단하게 표시
   - “다음 달 예정” 게임 슬롯

4. **회사 비전/철학**
   - 키워드: Speed / Quality / Data / Players
   - “한 달 1게임” 전략 요약

5. **개발/운영 방식**
   - Flutter/Firebase/GCP/AI 활용 간단 소개
   - “작게 자주 만들고, 데이터로 개선한다” 메시지

6. **뉴스/블로그 하이라이트**
   - 최신 포스트 3개

7. **푸터**
   - 회사 정보, 링크, 정책(개인정보처리방침, 이용약관), SNS 등

### 2.3 Home 설정 JSON 예시 (`content/company.json`)

```json
{
  "hero": {
    "title": "한 달에 한 번, 새로운 플레이.",
    "subtitle": "Monthly Games는 퍼즐, 방치형, JRPG를 중심으로 매월 새로운 모바일 게임을 선보입니다.",
    "primary_cta": {
      "label": "게임 둘러보기",
      "href": "/games"
    },
    "secondary_cta": {
      "label": "최신 게임 바로 플레이",
      "href": "/games/game_0001"
    }
  },
  "featured_games": ["game_0001", "game_0003", "game_0004"],
  "show_timeline": true,
  "show_ai_process_section": true
}
```

---

## 3. 게임 서비스 페이지 설계 (`/games/[gameId]`)

### 3.1 공통 템플릿 구조

각 게임 상세 페이지 공통 레이아웃:

1. **Hero**
   - 게임 로고/타이틀
   - 한 줄 설명(USP)
   - 스토어 버튼(플레이스토어/앱스토어)
   - 썸네일/영상 배경

2. **게임 소개 & 특징**
   - 짧은 게임 소개 문단
   - 3~5개의 핵심 특징 카드

3. **스크린샷 / 트레일러**
   - 이미지 갤러리
   - YouTube/스토어 트레일러 embed

4. **게임 시스템 / 코어 루프**
   - 장르 태그
   - 코어 루프 단계 요약(텍스트 or 라이트 다이어그램)
   - 타깃 플레이어 설명

5. **라이브/운영 정보**
   - 현재 버전, 최근 업데이트(날짜, 버전, 주요 변경점)
   - 진행 중인 이벤트 배너 (마케팅 캠페인 JSON과 연동)

6. **FAQ & Support**
   - 자주 묻는 질문
   - 고객센터/버그 신고/커뮤니티 링크

7. **푸터**
   - 정책/지원 OS/지원 기기 등

### 3.2 게임 페이지 JSON 설정 예시 (`content/games/game_0001.json`)

```json
{
  "game_id": "game_0001",
  "title_kr": "심플 타워 디펜스",
  "short_tagline": "타워 배치와 전략으로 웨이브를 막는 모바일 디펜스 게임.",
  "stores": [
    {
      "platform": "google_play",
      "url": "https://play.google.com/...",
      "label": "Google Play에서 받기"
    },
    {
      "platform": "app_store",
      "url": "https://apps.apple.com/...",
      "label": "App Store에서 받기"
    }
  ],
  "hero_media": {
    "type": "video",
    "youtube_id": "XXXXXXXX",
    "fallback_image": "/images/game_0001/hero.jpg"
  },
  "features": [
    {
      "title": "퍼즐로 여는 던전",
      "description": "간단한 퍼즐 플레이로 던전을 열고 전투를 준비하세요."
    },
    {
      "title": "방치로 쌓이는 보상",
      "description": "접속하지 않아도 자원이 쌓이는 방치형 성장 시스템."
    },
    {
      "title": "자동 전투 JRPG",
      "description": "직접 조작하지 않아도 전투는 계속 진행됩니다."
    }
  ],
  "systems": {
    "core_loop": [
      "퍼즐 스테이지 도전",
      "전투 및 보상 획득",
      "영웅/시설 강화",
      "방치 수익 수령",
      "다음 던전/챕터로 진행"
    ],
    "genre_tags": ["puzzle", "idle", "jrpg"],
    "target_players": [
      "짧은 시간에 가볍게 즐기고 싶은 유저",
      "성장/숫자 상승 보는 걸 좋아하는 유저"
    ]
  },
  "live_info": {
    "status": "live",
    "current_version": "1.0.3",
    "last_update": "2025-02-15",
    "recent_patch_summary": [
      "챕터 3 신규 보스 추가",
      "튜토리얼 난이도 완화",
      "광고 보상 밸런스 조정"
    ],
    "active_events": [
      {
        "campaign_id": "game_0001_spring_event",
        "title": "봄맞이 출석 이벤트",
        "period": "2025-03-01 ~ 2025-03-14"
      }
    ]
  },
  "faq": [
    {
      "q": "광고를 봐도 보상이 지급되지 않아요.",
      "a": "앱을 재시작한 뒤에도 문제가 지속되면 설정 > 고객센터에서 로그와 함께 문의해 주세요."
    }
  ],
  "support": {
    "email": "support@monthly.games",
    "discord": "https://discord.gg/...",
    "bug_report_form": "https://forms.gle/..."
  }
}
```

---

## 4. 마케팅 랜딩 페이지 설계 (`go.monthly.games/[slug]`)

### 4.1 랜딩 페이지 타입

1. 런칭/UA 랜딩 – 설치/스토어 방문 유도
2. 사전예약 랜딩 – 이메일/사전등록 유도
3. 이벤트/캠페인 랜딩 – 특정 이벤트 참여/공유 유도

### 4.2 공통 섹션 구조

1. Hero – 한 줄 카피, 비주얼, 메인 CTA
2. 핵심 포인트 – 3~4개 장점/차별점
3. 플레이/하이라이트 영상
4. 소셜 증거 – 평점/리뷰/다운로드 수 등
5. FAQ – 핵심 질문 1~3개
6. CTA 반복 – 상/중/하단 스토어 버튼

### 4.3 랜딩 설정 JSON 예시

```json
{
  "slug": "game_0001_launch_kr",
  "game_id": "game_0001",
  "type": "launch",
  "locale": "ko-KR",
  "hero": {
    "headline": "배치가 곧 승리다.",
    "subheadline": "심플 타워 디펜스, 지금 바로 설치하고 보상을 받아보세요.",
    "primary_cta": {
      "label": "지금 무료로 플레이",
      "target": "store",
      "store_priority": "google_play"
    },
    "background_image": "/landing/game_0001/bg_hero.jpg"
  },
  "selling_points": [
    "간단한 조작, 깊이 있는 타워 배치 전략",
    "웨이브 방어와 보상 루프",
    "다양한 타워/적 조합으로 전술 확장"
  ],
  "video": {
    "youtube_id": "XXXXXXXX",
    "autoplay": true,
    "loop": true,
    "mute": true
  },
  "social_proof": {
    "show": true,
    "rating": 4.6,
    "reviews_count": 1200,
    "quotes": [
      "출퇴근길에 딱 맞는 게임.",
      "과금 압박이 적어서 오래 하게 됩니다."
    ]
  },
  "conversion_goal": {
    "type": "install",
    "kpi": "cv_install_from_landing"
  },
  "tracking": {
    "ga4_event_name": "landing_game_0001_launch_kr_view",
    "campaign_id": "game_0001_launch_kr",
    "source_param": "utm_source",
    "medium_param": "utm_medium"
  }
}
```

---

## 5. 운영/분석 연계

### 5.1 GitHub 이슈/레포 연계

- `mg-web-main`:
  - 회사 홈페이지/게임 서비스 페이지 관련 이슈 관리
- `mg-web-landing`:
  - 캠페인별 랜딩 구현/수정 이슈 관리
- `mg-common-marketing`:
  - 캠페인/크리에이티브 정의, 랜딩 JSON 스키마/템플릿 관리
- `mg-common-analytics`:
  - 웹 이벤트 스키마/BigQuery 쿼리/대시보드 관리

### 5.2 공통 이벤트 설계 (GA4)

- `page_view` + `page_type` (home, game_detail, landing)
- `cta_click` + `cta_type` (install, pre_register, event_join)
- `outbound_click` (스토어/딥링크)
- 게임 상세:
  - `game_page_view` (game_id, source)
  - `game_store_click` (game_id, platform)
- 랜딩:
  - `landing_view` (landing_id, campaign_id, source, medium)
  - `landing_cv` (conversion_type: install/pre_register/event_join)
  - `variant_id` (A/B 테스트용)

이 정의는 `mg-common-analytics/schemas/events.json`에 포함해
웹/게임/마케팅 데이터를 통합 분석할 수 있게 한다.

---

## 6. SEO 및 메타데이터 설정

### 6.1 공통 메타 태그

모든 페이지에 적용되는 기본 메타 설정:

```html
<!-- 기본 메타 -->
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<meta name="theme-color" content="#1a1a2e" />

<!-- SEO 메타 -->
<meta name="robots" content="index, follow" />
<link rel="canonical" href="https://monthly.games/..." />

<!-- 다국어 hreflang -->
<link rel="alternate" hreflang="ko" href="https://monthly.games/ko/..." />
<link rel="alternate" hreflang="en" href="https://monthly.games/en/..." />
<link rel="alternate" hreflang="ja" href="https://monthly.games/ja/..." />
<link rel="alternate" hreflang="x-default" href="https://monthly.games/en/..." />
```

### 6.2 Open Graph / Twitter Card

```html
<!-- Open Graph -->
<meta property="og:type" content="website" />
<meta property="og:site_name" content="Monthly Games" />
<meta property="og:title" content="페이지 제목" />
<meta property="og:description" content="페이지 설명" />
<meta property="og:image" content="https://monthly.games/og/default.png" />
<meta property="og:url" content="https://monthly.games/..." />
<meta property="og:locale" content="ko_KR" />

<!-- Twitter Card -->
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:site" content="@monthlygames" />
<meta name="twitter:title" content="페이지 제목" />
<meta name="twitter:description" content="페이지 설명" />
<meta name="twitter:image" content="https://monthly.games/og/default.png" />
```

### 6.3 OG 이미지 규격

| 용도 | 크기 | 파일명 패턴 |
|------|------|-------------|
| 기본 | 1200×630px | `og/default.png` |
| 게임별 | 1200×630px | `og/games/game_0001.png` |
| 랜딩별 | 1200×630px | `og/landings/{slug}.png` |

### 6.4 Sitemap 및 robots.txt

#### sitemap.xml 구조

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:xhtml="http://www.w3.org/1999/xhtml">
  <url>
    <loc>https://monthly.games/ko/</loc>
    <xhtml:link rel="alternate" hreflang="ko" href="https://monthly.games/ko/" />
    <xhtml:link rel="alternate" hreflang="en" href="https://monthly.games/en/" />
    <changefreq>weekly</changefreq>
    <priority>1.0</priority>
  </url>
  <!-- 게임 페이지들 -->
  <url>
    <loc>https://monthly.games/ko/games/game_0001</loc>
    <changefreq>daily</changefreq>
    <priority>0.9</priority>
  </url>
</urlset>
```

#### robots.txt

```text
User-agent: *
Allow: /

Sitemap: https://monthly.games/sitemap.xml

# 마케팅 랜딩은 별도 sitemap
Sitemap: https://go.monthly.games/sitemap.xml

# 관리자/테스트 경로 차단
Disallow: /admin/
Disallow: /api/
Disallow: /_next/
```

### 6.5 구조화 데이터 (JSON-LD)

#### 회사 정보

```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Monthly Games",
  "url": "https://monthly.games",
  "logo": "https://monthly.games/logo.png",
  "sameAs": [
    "https://twitter.com/monthlygames",
    "https://discord.gg/monthlygames"
  ]
}
```

#### 게임 상세 페이지

```json
{
  "@context": "https://schema.org",
  "@type": "VideoGame",
  "name": "심플 타워 디펜스",
  "description": "타워 배치와 전략으로 웨이브를 막는 모바일 디펜스 게임.",
  "genre": ["Tower Defense", "Strategy"],
  "gamePlatform": ["Android", "iOS"],
  "applicationCategory": "Game",
  "operatingSystem": "Android, iOS",
  "offers": {
    "@type": "Offer",
    "price": "0",
    "priceCurrency": "KRW"
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.6",
    "ratingCount": "1200"
  }
}
```

---

## 7. 이미지 및 미디어 최적화

### 7.1 이미지 최적화 전략

#### Next.js Image 컴포넌트 활용

```tsx
import Image from 'next/image';

<Image
  src="/images/game_0001/hero.jpg"
  alt="심플 타워 디펜스 메인 이미지"
  width={1200}
  height={630}
  priority  // LCP 이미지에 적용
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,..."
/>
```

#### 이미지 포맷 전략

| 용도 | 포맷 | 비고 |
|------|------|------|
| 히어로/배경 | WebP (AVIF fallback) | 자동 변환 via Vercel Image Optimization |
| 아이콘/로고 | SVG | 벡터 유지 |
| 스크린샷 | WebP | 품질 85% |
| OG 이미지 | PNG | 호환성 우선 |

### 7.2 CDN 구성

- **Vercel Edge Network**: 정적 자산 자동 CDN 배포
- **이미지 경로**: `/images/` → Vercel Image Optimization 자동 적용
- **캐시 정책**:
  - 정적 이미지: `Cache-Control: public, max-age=31536000, immutable`
  - 동적 콘텐츠: `Cache-Control: public, s-maxage=60, stale-while-revalidate=300`

### 7.3 반응형 이미지

```tsx
<Image
  src="/images/hero.jpg"
  alt="Hero"
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  fill
  style={{ objectFit: 'cover' }}
/>
```

---

## 8. 에러 페이지 설계

### 8.1 404 페이지 (Not Found)

**목적**: 잘못된 URL 접근 시 사용자를 유효한 페이지로 안내

**구성 요소**:
- 친근한 에러 메시지: "앗, 페이지를 찾을 수 없어요!"
- Monthly Games 캐릭터/일러스트
- 추천 링크:
  - 홈으로 가기
  - 게임 목록 보기
  - 인기 게임 바로가기
- 검색창 (선택)

### 8.2 500 페이지 (Server Error)

**목적**: 서버 오류 시 사용자에게 안내 및 재시도 유도

**구성 요소**:
- 에러 메시지: "잠시 문제가 생겼어요. 곧 복구됩니다!"
- 새로고침 버튼
- 상태 페이지 링크 (status.monthly.games)
- 고객센터 문의 링크

### 8.3 점검 페이지 (Maintenance)

**목적**: 계획된 점검 시 표시

**구성 요소**:
- 점검 안내 메시지
- 예상 완료 시간
- SNS 채널 안내 (실시간 업데이트)

### 8.4 에러 페이지 파일 구조

```text
apps/web/
  ├─ pages/
  │   ├─ 404.tsx
  │   ├─ 500.tsx
  │   └─ _error.tsx
  └─ components/
      └─ ErrorLayout.tsx  # 공통 에러 레이아웃
```

---

## 9. 성능 목표 (Core Web Vitals)

### 9.1 목표 지표

| 지표 | 목표값 | 측정 기준 |
|------|--------|-----------|
| **LCP** (Largest Contentful Paint) | < 2.5s | 75th percentile |
| **INP** (Interaction to Next Paint) | < 200ms | 75th percentile |
| **CLS** (Cumulative Layout Shift) | < 0.1 | 75th percentile |
| **TTFB** (Time to First Byte) | < 800ms | 75th percentile |
| **FCP** (First Contentful Paint) | < 1.8s | 75th percentile |

### 9.2 성능 최적화 전략

#### 빌드 타임 최적화

- **SSG (Static Site Generation)**: 홈, About, 게임 목록 등 정적 페이지
- **ISR (Incremental Static Regeneration)**: 게임 상세 페이지 (revalidate: 60초)
- **SSR**: 실시간 데이터가 필요한 이벤트 페이지

#### 런타임 최적화

- **코드 스플리팅**: 페이지별 자동 분할 + dynamic import
- **폰트 최적화**: `next/font`로 FOUT/FOIT 방지
- **스크립트 로딩**:
  - GA4, 광고 SDK → `strategy="afterInteractive"`
  - 채팅 위젯 → `strategy="lazyOnload"`

#### 모니터링

- **Vercel Analytics**: 실시간 Core Web Vitals 모니터링
- **Google Search Console**: 필드 데이터 확인
- **Lighthouse CI**: PR마다 자동 성능 체크

### 9.3 성능 버짓

```json
{
  "performance_budget": {
    "javascript": "300KB (gzip)",
    "css": "50KB (gzip)",
    "images_per_page": "500KB",
    "total_page_weight": "1MB",
    "requests_per_page": 30
  }
}
```

---

## 10. 추가 시스템 설계

### 10.1 인증 및 회원 시스템

#### 현재 방침

- **웹사이트 자체 회원가입 불필요**: 정보 제공 목적의 웹사이트로 운영
- **게임 내 계정**: Firebase Auth 기반, 게임 앱 내에서만 관리
- **뉴스레터 구독**: 이메일 수집만 (Mailchimp/Resend 연동)

#### 향후 확장 시

```text
통합 계정 시스템 도입 시:
- OAuth 2.0 (Google, Apple, Discord)
- Firebase Auth 웹 SDK 연동
- 게임 간 공유 프로필/업적/보상 연동
```

### 10.2 게임 간 연동 (크로스 프로모션)

#### 구현 방식

```json
{
  "cross_promo": {
    "enabled": true,
    "current_game": "game_0003",
    "promo_targets": [
      {
        "game_id": "game_0001",
        "banner_image": "/promo/game_0001_banner.png",
        "message": "심플 타워 디펜스 플레이어 전용 보너스!",
        "reward_code": "XPROMO_0003_0001"
      }
    ]
  }
}
```

#### 표시 위치

- 게임 상세 페이지 하단 "다른 게임도 즐겨보세요" 섹션
- 랜딩 페이지 푸터
- 이벤트 페이지 사이드바

### 10.3 A/B 테스트 구현

#### 구현 방식

- **도구**: Vercel Edge Config + Flags SDK 또는 GrowthBook
- **적용 범위**: 랜딩 페이지 우선, 이후 게임 상세 페이지 확장

#### 테스트 설정 예시

```json
{
  "ab_tests": {
    "landing_hero_v2": {
      "enabled": true,
      "variants": [
        { "id": "control", "weight": 50 },
        { "id": "variant_a", "weight": 50 }
      ],
      "metrics": ["landing_cv", "cta_click"]
    }
  }
}
```

#### 분석 연동

- `variant_id`를 GA4 이벤트 파라미터로 전송
- BigQuery에서 variant별 전환율 비교 분석

### 10.4 캐싱 전략

#### 렌더링 방식 선택 기준

| 페이지 유형 | 렌더링 | 캐싱 | 이유 |
|-------------|--------|------|------|
| 홈페이지 | ISR | revalidate: 3600 | 하루 몇 번 업데이트 |
| 게임 목록 | ISR | revalidate: 1800 | 신규 게임 추가 시 반영 |
| 게임 상세 | ISR | revalidate: 60 | 라이브 이벤트 반영 필요 |
| 블로그 글 | SSG | 빌드 시 생성 | 발행 후 변경 거의 없음 |
| 랜딩 페이지 | SSG/ISR | revalidate: 300 | 캠페인 중 수정 가능 |
| About/Careers | SSG | 빌드 시 생성 | 거의 변경 없음 |

#### API 응답 캐싱

```typescript
// Next.js API Route 캐싱 예시
export async function GET() {
  return Response.json(data, {
    headers: {
      'Cache-Control': 'public, s-maxage=60, stale-while-revalidate=300',
    },
  });
}
```

---

## 11. 브랜드 스타일 및 톤앤매너 가이드

### 11.1 브랜드 아이덴티티

#### 핵심 가치

| 키워드 | 의미 | 표현 방식 |
|--------|------|-----------|
| **Speed** | 빠른 실행, 민첩한 대응 | 간결한 문장, 즉각적인 피드백 |
| **Playful** | 게임다운 재미, 가벼운 유머 | 친근한 어투, 캐릭터 활용 |
| **Honest** | 투명한 소통, 과장 없는 약속 | 명확한 정보, 솔직한 톤 |
| **Data-Driven** | 근거 있는 개선, 플레이어 중심 | 수치/성과 공유, 피드백 반영 강조 |

#### 브랜드 퍼스널리티

> "Monthly Games는 **열정적인 인디 개발팀 친구** 같은 존재다.
> 매달 새로운 걸 만들어 보여주고, 솔직하게 피드백을 구하며, 함께 성장한다."

### 11.2 컬러 팔레트

#### Primary Colors

| 용도 | 색상 | HEX | 사용처 |
|------|------|-----|--------|
| Primary | Deep Navy | `#1a1a2e` | 배경, 헤더 |
| Primary Accent | Electric Purple | `#6c5ce7` | CTA 버튼, 강조 |
| Secondary | Coral Pink | `#ff6b81` | 알림, 하이라이트 |

#### Neutral Colors

| 용도 | 색상 | HEX |
|------|------|-----|
| Text Primary | Off White | `#f5f5f7` |
| Text Secondary | Light Gray | `#a0a0a0` |
| Border | Dark Gray | `#2d2d44` |
| Background Alt | Charcoal | `#16162a` |

#### Semantic Colors

| 용도 | 색상 | HEX |
|------|------|-----|
| Success | Mint Green | `#00d9a6` |
| Warning | Warm Yellow | `#ffc048` |
| Error | Soft Red | `#ff5252` |
| Info | Sky Blue | `#54a0ff` |

### 11.3 타이포그래피

#### 폰트 패밀리

| 용도 | 한글 | 영문 | Fallback |
|------|------|------|----------|
| 제목 | Pretendard | Pretendard | system-ui, sans-serif |
| 본문 | Pretendard | Pretendard | system-ui, sans-serif |
| 코드/숫자 | - | JetBrains Mono | monospace |

#### 폰트 사이즈 스케일

```css
--text-xs: 0.75rem;    /* 12px - 캡션, 라벨 */
--text-sm: 0.875rem;   /* 14px - 보조 텍스트 */
--text-base: 1rem;     /* 16px - 본문 */
--text-lg: 1.125rem;   /* 18px - 리드 텍스트 */
--text-xl: 1.25rem;    /* 20px - 소제목 */
--text-2xl: 1.5rem;    /* 24px - 섹션 제목 */
--text-3xl: 1.875rem;  /* 30px - 페이지 제목 */
--text-4xl: 2.25rem;   /* 36px - 히어로 서브 */
--text-5xl: 3rem;      /* 48px - 히어로 메인 */
```

### 11.4 톤앤매너 가이드

#### 문체 원칙

| 원칙 | 좋은 예 | 피해야 할 예 |
|------|---------|--------------|
| **간결하게** | "오늘 업데이트했어요" | "금일 업데이트가 진행되었음을 알려드립니다" |
| **친근하게** | "같이 해봐요!" | "참여해 주시기 바랍니다" |
| **명확하게** | "3월 15일 오픈" | "곧 만나요" |
| **솔직하게** | "버그 수정 중이에요" | "더 나은 서비스를 위해 점검 중입니다" |

#### 상황별 톤

| 상황 | 톤 | 예시 |
|------|-----|------|
| **신규 게임 출시** | 설렘 + 자신감 | "드디어 공개! 이번 달 게임, 직접 플레이해 보세요." |
| **업데이트 안내** | 친근 + 구체적 | "1.2 업데이트: 보스 밸런스 조정하고, 버그 5개 잡았어요." |
| **에러/장애** | 솔직 + 책임감 | "서버 문제로 접속이 안 돼요. 30분 내 복구 목표입니다." |
| **이벤트** | 재미 + 명확한 혜택 | "7일 출석하면 10연차 무료! 놓치지 마세요." |
| **FAQ/지원** | 도움 + 공감 | "불편하셨죠? 이렇게 해결할 수 있어요." |

#### 금지 표현

- ❌ "~하시기 바랍니다" → ✅ "~해 주세요" / "~해 보세요"
- ❌ "최고의", "최상의", "완벽한" (과장) → ✅ 구체적인 장점 설명
- ❌ "죄송합니다만" (반복적 사과) → ✅ 해결책 먼저 제시
- ❌ "추후 안내 예정" (모호함) → ✅ 구체적 일정 또는 "확정되면 바로 알려드릴게요"

### 11.5 UI 컴포넌트 스타일

#### 버튼

```css
/* Primary CTA */
.btn-primary {
  background: linear-gradient(135deg, #6c5ce7, #a29bfe);
  color: #ffffff;
  border-radius: 12px;
  padding: 12px 24px;
  font-weight: 600;
  transition: transform 0.2s, box-shadow 0.2s;
}
.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(108, 92, 231, 0.4);
}

/* Secondary */
.btn-secondary {
  background: transparent;
  border: 2px solid #6c5ce7;
  color: #6c5ce7;
  border-radius: 12px;
}
```

#### 카드

```css
.card {
  background: rgba(45, 45, 68, 0.6);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  padding: 24px;
  transition: transform 0.3s, border-color 0.3s;
}
.card:hover {
  transform: translateY(-4px);
  border-color: #6c5ce7;
}
```

#### 애니메이션 원칙

- **Duration**: 200~300ms (빠르고 반응적)
- **Easing**: `cubic-bezier(0.4, 0, 0.2, 1)` (자연스러운 감속)
- **방향**: 아래→위 (상승감), 작은→큰 (확장감)
- **과하지 않게**: 핵심 인터랙션에만 적용

### 11.6 아이콘 및 일러스트

#### 아이콘 스타일

- **스타일**: Outlined, 2px stroke, rounded corners
- **크기**: 16px (인라인), 24px (기본), 32px (강조)
- **라이브러리**: Lucide Icons 또는 Phosphor Icons 권장

#### 일러스트/캐릭터 활용

| 상황 | 캐릭터/일러스트 |
|------|-----------------|
| 에러 페이지 | 당황한 표정의 마스코트 |
| 빈 상태 | 기다리는 포즈의 마스코트 |
| 성공/완료 | 축하하는 마스코트 |
| 로딩 | 달리는/움직이는 마스코트 |

### 11.7 다크/라이트 모드

#### 기본 정책

- **기본값**: 다크 모드 (게임 브랜드 특성)
- **라이트 모드**: 시스템 설정 또는 수동 전환 지원 (Phase 2)

#### 다크 모드 팔레트 (기본)

```css
:root {
  --bg-primary: #1a1a2e;
  --bg-secondary: #16162a;
  --text-primary: #f5f5f7;
  --text-secondary: #a0a0a0;
  --accent: #6c5ce7;
}
```

### 11.8 문구 작성 가이드

#### 헤드라인 작성법

```text
✅ 좋은 예:
- "한 달에 한 번, 새로운 플레이."
- "퍼즐 끝나면, 던전이 열린다."
- "오늘도 보상이 쌓이는 중."

❌ 피해야 할 예:
- "세계 최고의 모바일 게임 회사"
- "혁신적인 게임 경험을 선사합니다"
- "고객님의 성원에 감사드립니다"
```

#### CTA 버튼 문구

| 목적 | 문구 |
|------|------|
| 스토어 이동 | "지금 플레이" / "무료로 시작" |
| 사전예약 | "출시 알림 받기" / "먼저 만나기" |
| 이벤트 참여 | "보상 받기" / "참여하기" |
| 더보기 | "자세히 보기" / "전체 보기" |

### 11.9 이미지/미디어 스타일

#### 스크린샷 가이드

- **해상도**: 최소 1080×1920 (모바일 세로)
- **UI 표시**: 게임 UI가 잘 보이도록 캡처
- **상황**: 액션/하이라이트 순간 포착
- **편집**: 과도한 필터 지양, 실제 게임과 유사하게

#### 영상 가이드

- **길이**: 티저 15~30초, 트레일러 60~90초
- **포맷**: 16:9 (웹), 9:16 (쇼츠/릴스)
- **BGM**: 저작권 프리 또는 자체 제작
- **자막**: 필수 (음소거 시청 대응)
