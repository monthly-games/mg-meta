# MG-Games 호스팅 아키텍처 가이드

> **문서 버전**: 1.0.0
> **최종 수정일**: 2025-12-19

---

## 1. 호스팅 구조 개요

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        MG-Games Hosting Architecture                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │                         Cloudflare DNS                           │   │
│   │                       mg-games.com Zone                          │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                    │                    │                    │          │
│                    ▼                    ▼                    ▼          │
│   ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐      │
│   │     Vercel      │   │  GitHub Pages   │   │  Firebase/GCP   │      │
│   │                 │   │                 │   │                 │      │
│   │ 회사 홈페이지    │   │ 게임 랜딩페이지  │   │ API & Backend   │      │
│   │                 │   │ (52개)          │   │                 │      │
│   │ mg-games.com    │   │ game-0001.      │   │ api.mg-games.   │      │
│   │                 │   │ mg-games.com    │   │ com             │      │
│   └─────────────────┘   └─────────────────┘   └─────────────────┘      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. 회사 홈페이지 (Vercel)

### 2.1 레포지토리 구조
```
repos/mg-website/
├── src/
│   ├── app/
│   │   ├── page.tsx              # 메인 페이지
│   │   ├── games/
│   │   │   └── page.tsx          # 게임 목록
│   │   ├── about/
│   │   │   └── page.tsx          # 회사 소개
│   │   └── contact/
│   │       └── page.tsx          # 문의
│   ├── components/
│   └── styles/
├── public/
│   ├── images/
│   │   └── games/               # 게임 썸네일
│   └── locales/                 # 다국어
├── package.json
├── next.config.js
└── vercel.json
```

### 2.2 Vercel 설정

#### vercel.json
```json
{
  "framework": "nextjs",
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "regions": ["icn1", "sin1"],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        }
      ]
    }
  ],
  "rewrites": [
    {
      "source": "/api/:path*",
      "destination": "https://api.mg-games.com/:path*"
    }
  ]
}
```

#### 환경 변수
```
# Vercel Dashboard > Settings > Environment Variables

NEXT_PUBLIC_FIREBASE_API_KEY=xxx
NEXT_PUBLIC_GA_ID=G-XXXXXXXX
NEXT_PUBLIC_API_URL=https://api.mg-games.com
```

### 2.3 도메인 설정

#### Vercel Domains
```
Vercel Dashboard > Project > Settings > Domains

- mg-games.com (Primary)
- www.mg-games.com (Redirect to mg-games.com)
```

#### DNS 레코드 (Cloudflare)
```
Type    Name    Content              Proxy
A       @       76.76.19.19         Proxied
CNAME   www     cname.vercel-dns.com Proxied
```

---

## 3. 게임 랜딩페이지 (GitHub Pages)

### 3.1 구조 옵션

#### Option A: 개별 레포 (52개)
```
repos/
├── mg-game-0001-landing/
├── mg-game-0002-landing/
└── ...
```
- **장점**: 독립적 배포, 게임팀별 관리
- **단점**: 관리 복잡

#### Option B: 단일 레포 + 서브디렉토리 (권장)
```
repos/mg-landings/
├── games/
│   ├── 0001/
│   │   ├── index.html
│   │   ├── assets/
│   │   └── locales/
│   ├── 0002/
│   └── ...
├── shared/
│   ├── css/
│   ├── js/
│   └── templates/
└── .github/
    └── workflows/
        └── deploy.yml
```
- **장점**: 중앙 관리, 공유 리소스
- **단점**: 단일 레포 크기 증가

### 3.2 GitHub Pages 설정

#### 레포지토리 설정
```
GitHub > mg-landings > Settings > Pages

- Source: GitHub Actions
- Custom domain: games.mg-games.com
- Enforce HTTPS: ✓
```

#### GitHub Actions 워크플로우
```yaml
# .github/workflows/deploy.yml
name: Deploy Landing Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pages: write
      id-token: write

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Build
        run: |
          npm ci
          npm run build

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: './dist'

      - name: Deploy to GitHub Pages
        uses: actions/deploy-pages@v4
```

### 3.3 랜딩페이지 템플릿

#### 디렉토리 구조
```
games/0001/
├── index.html
├── assets/
│   ├── hero.webp           # 1200x630
│   ├── screenshots/
│   │   ├── 01.webp         # 1080x1920
│   │   ├── 02.webp
│   │   └── 03.webp
│   ├── icon.png            # 512x512
│   └── logo.svg
├── locales/
│   ├── en.json
│   ├── ko.json
│   ├── es.json
│   └── pt.json
└── meta.json
```

#### meta.json 스키마
```json
{
  "gameId": "game_0001",
  "title": {
    "en": "Slime Jump",
    "ko": "슬라임 점프"
  },
  "description": {
    "en": "Jump through colorful worlds!",
    "ko": "다채로운 세계를 점프하세요!"
  },
  "genre": ["casual", "arcade"],
  "stores": {
    "android": "https://play.google.com/store/apps/details?id=com.mggames.game0001",
    "ios": "https://apps.apple.com/app/id123456789"
  },
  "socialLinks": {
    "facebook": "https://facebook.com/mggames",
    "instagram": "https://instagram.com/mggames"
  },
  "privacyPolicy": "/legal/privacy",
  "termsOfService": "/legal/terms"
}
```

#### index.html 템플릿
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Slime Jump - MG Games</title>

  <!-- SEO -->
  <meta name="description" content="Jump through colorful worlds!">
  <meta name="keywords" content="casual game, mobile game, arcade">

  <!-- Open Graph -->
  <meta property="og:title" content="Slime Jump">
  <meta property="og:description" content="Jump through colorful worlds!">
  <meta property="og:image" content="https://games.mg-games.com/0001/assets/hero.webp">
  <meta property="og:url" content="https://games.mg-games.com/0001/">
  <meta property="og:type" content="website">

  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:title" content="Slime Jump">
  <meta name="twitter:description" content="Jump through colorful worlds!">
  <meta name="twitter:image" content="https://games.mg-games.com/0001/assets/hero.webp">

  <!-- App Links -->
  <meta property="al:android:package" content="com.mggames.game0001">
  <meta property="al:android:app_name" content="Slime Jump">
  <meta property="al:ios:app_store_id" content="123456789">
  <meta property="al:ios:app_name" content="Slime Jump">

  <!-- Styles -->
  <link rel="stylesheet" href="/shared/css/landing.css">

  <!-- Analytics -->
  <script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXX"></script>
  <script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());
    gtag('config', 'G-XXXXXXXX');
  </script>
</head>
<body>
  <header class="hero">
    <img src="assets/hero.webp" alt="Slime Jump" class="hero-image">
    <div class="hero-content">
      <img src="assets/logo.svg" alt="Slime Jump Logo" class="logo">
      <h1>Slime Jump</h1>
      <p class="tagline">Jump through colorful worlds!</p>
      <div class="store-badges">
        <a href="https://play.google.com/store/apps/details?id=com.mggames.game0001"
           class="store-badge" target="_blank">
          <img src="/shared/images/google-play-badge.png" alt="Get it on Google Play">
        </a>
        <a href="https://apps.apple.com/app/id123456789"
           class="store-badge" target="_blank">
          <img src="/shared/images/app-store-badge.svg" alt="Download on the App Store">
        </a>
      </div>
    </div>
  </header>

  <section class="screenshots">
    <h2>Screenshots</h2>
    <div class="screenshot-gallery">
      <img src="assets/screenshots/01.webp" alt="Screenshot 1">
      <img src="assets/screenshots/02.webp" alt="Screenshot 2">
      <img src="assets/screenshots/03.webp" alt="Screenshot 3">
    </div>
  </section>

  <section class="features">
    <h2>Features</h2>
    <ul>
      <li>100+ colorful levels</li>
      <li>Easy one-touch controls</li>
      <li>Collect cute slime characters</li>
      <li>Challenge your friends</li>
    </ul>
  </section>

  <footer>
    <nav class="legal-links">
      <a href="/legal/privacy">Privacy Policy</a>
      <a href="/legal/terms">Terms of Service</a>
    </nav>
    <p>&copy; 2025 MG Games. All rights reserved.</p>
  </footer>

  <script src="/shared/js/landing.js"></script>
</body>
</html>
```

### 3.4 URL 구조 옵션

#### Option A: 서브도메인
```
game-0001.mg-games.com
game-0002.mg-games.com
...
```
- DNS 설정: 와일드카드 CNAME
- GitHub Pages: 각 레포에 커스텀 도메인

#### Option B: 서브디렉토리 (권장)
```
games.mg-games.com/0001/
games.mg-games.com/0002/
...
```
- 단일 GitHub Pages 배포
- 관리 용이

#### Option C: 메인 도메인 경로
```
mg-games.com/games/0001/
mg-games.com/games/0002/
...
```
- Vercel에서 처리 (rewrite)
- SEO 통합

### 3.5 DNS 설정 (games.mg-games.com)

#### Cloudflare
```
Type    Name    Content                              Proxy
CNAME   games   mg-games.github.io                   Proxied
```

#### GitHub Pages Custom Domain
```
# repos/mg-landings/CNAME
games.mg-games.com
```

---

## 4. API 서버 (Firebase/GCP)

### 4.1 Cloud Run 설정

#### Dockerfile
```dockerfile
FROM node:20-slim

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

ENV PORT=8080
EXPOSE 8080

CMD ["node", "dist/server.js"]
```

#### cloudbuild.yaml
```yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/mg-api:$COMMIT_SHA', '.']

  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/mg-api:$COMMIT_SHA']

  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: gcloud
    args:
      - 'run'
      - 'deploy'
      - 'mg-api'
      - '--image=gcr.io/$PROJECT_ID/mg-api:$COMMIT_SHA'
      - '--region=asia-northeast3'
      - '--platform=managed'
```

### 4.2 커스텀 도메인 매핑

#### Cloud Run 도메인 매핑
```bash
gcloud run domain-mappings create \
  --service=mg-api \
  --domain=api.mg-games.com \
  --region=asia-northeast3
```

#### DNS 레코드
```
Type    Name    Content              Proxy
CNAME   api     ghs.googlehosted.com Proxied (SSL Only)

# 또는 A 레코드 (Google 제공)
A       api     216.239.32.21        DNS Only
```

---

## 5. CDN 설정 (Cloud Storage)

### 5.1 버킷 구조
```
gs://mg-games-cdn/
├── games/
│   ├── 0001/
│   │   ├── icons/
│   │   ├── banners/
│   │   └── assets/
│   └── ...
├── shared/
│   ├── fonts/
│   └── images/
└── marketing/
    ├── creatives/
    └── screenshots/
```

### 5.2 Cloud CDN 설정
```bash
# 백엔드 버킷 생성
gcloud compute backend-buckets create mg-cdn-backend \
  --gcs-bucket-name=mg-games-cdn \
  --enable-cdn

# URL 맵 생성
gcloud compute url-maps create mg-cdn-map \
  --default-backend-bucket=mg-cdn-backend

# SSL 인증서
gcloud compute ssl-certificates create mg-cdn-cert \
  --domains=cdn.mg-games.com

# HTTPS 프록시
gcloud compute target-https-proxies create mg-cdn-proxy \
  --url-map=mg-cdn-map \
  --ssl-certificates=mg-cdn-cert

# 전달 규칙
gcloud compute forwarding-rules create mg-cdn-rule \
  --global \
  --target-https-proxy=mg-cdn-proxy \
  --ports=443
```

### 5.3 CORS 설정
```json
[
  {
    "origin": ["https://mg-games.com", "https://games.mg-games.com"],
    "method": ["GET", "HEAD"],
    "responseHeader": ["Content-Type", "Content-Length"],
    "maxAgeSeconds": 86400
  }
]
```

```bash
gsutil cors set cors.json gs://mg-games-cdn
```

---

## 6. 전체 DNS 레코드 요약

### Cloudflare DNS Zone: mg-games.com
```
Type    Name     Content                      Proxy    TTL
A       @        76.76.19.19                  Yes      Auto
CNAME   www      cname.vercel-dns.com         Yes      Auto
CNAME   games    mg-games.github.io           Yes      Auto
CNAME   api      ghs.googlehosted.com         DNS      Auto
CNAME   cdn      c.storage.googleapis.com     Yes      Auto

# 이메일 (선택사항)
MX      @        mx.example.com               DNS      Auto
TXT     @        v=spf1 include:_spf.google.com ~all
```

---

## 7. CI/CD 파이프라인

### 7.1 회사 홈페이지 (Vercel)
```
Push to main → Vercel Auto Deploy → Preview → Production
```

### 7.2 랜딩페이지 (GitHub Pages)
```
Push to main → GitHub Actions → Build → Deploy to GitHub Pages
```

### 7.3 API (Cloud Run)
```
Push to main → Cloud Build → Docker Build → Deploy to Cloud Run
```

### 7.4 통합 배포 상태
```yaml
# .github/workflows/status.yml
name: Deployment Status

on:
  workflow_run:
    workflows: ["Deploy Website", "Deploy Landings", "Deploy API"]
    types: [completed]

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Notify Slack
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "Deployment completed: ${{ github.workflow }}",
              "blocks": [...]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

---

## 8. 체크리스트

### 회사 홈페이지 (Vercel)
- [ ] Vercel 프로젝트 생성
- [ ] GitHub 레포 연결
- [ ] 커스텀 도메인 설정
- [ ] 환경 변수 설정
- [ ] SSL 확인

### 게임 랜딩페이지 (GitHub Pages)
- [ ] 레포지토리 생성 (mg-landings)
- [ ] GitHub Actions 워크플로우
- [ ] 커스텀 도메인 (games.mg-games.com)
- [ ] 52개 게임 템플릿 생성
- [ ] SSL 확인

### API 서버 (Cloud Run)
- [ ] Cloud Run 서비스 생성
- [ ] 도메인 매핑
- [ ] SSL 인증서
- [ ] Cloud Build 트리거

### CDN (Cloud Storage)
- [ ] 버킷 생성
- [ ] Cloud CDN 설정
- [ ] CORS 설정
- [ ] 커스텀 도메인

### DNS (Cloudflare)
- [ ] DNS 레코드 설정
- [ ] SSL/TLS 설정
- [ ] 캐싱 규칙
- [ ] 방화벽 규칙

---

*문서 작성: 2025-12-19*
*MG-Games 인프라팀*
