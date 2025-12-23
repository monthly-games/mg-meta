# MG-Games Firebase CLI 자동화 가이드

> **문서 버전**: 1.0.0
> **최종 수정일**: 2025-12-19
> **대상**: 52종 게임 CLI 기반 자동 셋업

---

## 1. 사전 준비

### 1.1 Firebase CLI 설치
```bash
# npm으로 설치
npm install -g firebase-tools

# 버전 확인
firebase --version
```

### 1.2 로그인
```bash
# 브라우저 기반 로그인
firebase login

# CI 환경용 토큰 생성
firebase login:ci
# 출력된 토큰을 CI/CD 시크릿에 저장
```

### 1.3 Google Cloud CLI 설치 (BigQuery, Cloud Run용)
```bash
# Windows (PowerShell)
(New-Object Net.WebClient).DownloadFile("https://dl.google.com/dl/cloudsdk/channels/rapid/GoogleCloudSDKInstaller.exe", "$env:Temp\GoogleCloudSDKInstaller.exe")
& $env:Temp\GoogleCloudSDKInstaller.exe

# 로그인
gcloud auth login
gcloud config set project mg-games-prod
```

---

## 2. 단일 프로젝트 생성

### 2.1 프로젝트 구조
```
┌─────────────────────────────────────────────────────────────┐
│                mg-games-prod (단일 프로젝트)                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  104개 앱 (52 Android + 52 iOS)                             │
│  ├── com.mggames.game0001 ~ game0052 (Android)             │
│  └── com.mggames.game0001 ~ game0052 (iOS)                 │
│                                                             │
│  공유 서비스                                                 │
│  ├── Analytics      → app_info.id로 게임 구분              │
│  ├── Remote Config  → app.id 조건으로 게임별 설정           │
│  ├── Crashlytics    → 앱별 자동 분리                       │
│  ├── Cloud Functions→ 52개 게임 공통 API                   │
│  └── BigQuery       → 단일 데이터셋, 크로스 게임 분석       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 프로젝트 생성
```bash
# 프로덕션 프로젝트 (52개 게임 통합)
firebase projects:create mg-games-prod --display-name "MG-Games Production"

# 개발/스테이징 (선택사항)
firebase projects:create mg-games-dev --display-name "MG-Games Dev"
```

### 2.3 프로젝트 확인
```bash
firebase projects:list
firebase use mg-games-prod
```

---

## 3. 52개 앱 일괄 등록

### 3.1 앱 등록 스크립트

```bash
#!/bin/bash
# scripts/register_firebase_apps.sh
# 52개 게임 Android/iOS 앱 일괄 등록

PROJECT_ID="mg-games-prod"

# 게임 정보 배열 (게임ID:패키지명:앱이름)
declare -a GAMES=(
  "0001:com.mggames.game0001:Slime Jump"
  "0002:com.mggames.game0002:Block Puzzle"
  "0003:com.mggames.game0003:Color Match"
  # ... 0004~0051
  "0052:com.mggames.game0052:Safari Quest"
)

echo "=== MG-Games Firebase App Registration ==="
echo "Project: $PROJECT_ID"
echo "Total games: ${#GAMES[@]}"
echo ""

for game in "${GAMES[@]}"; do
  IFS=':' read -r game_id package_name app_name <<< "$game"

  echo "Registering MG-$game_id: $app_name"

  # Android 앱 등록
  firebase apps:create android \
    --project "$PROJECT_ID" \
    --package-name "$package_name" \
    --display-name "MG-$game_id Android"

  # iOS 앱 등록
  firebase apps:create ios \
    --project "$PROJECT_ID" \
    --bundle-id "$package_name" \
    --display-name "MG-$game_id iOS"

  echo "✓ MG-$game_id registered"
  echo ""
done

echo "=== Registration Complete ==="
```

### 3.2 앱 ID 목록 추출
```bash
# 모든 앱 목록 (JSON)
firebase apps:list --project mg-games-prod --json > firebase_apps.json

# Android 앱만
firebase apps:list android --project mg-games-prod

# iOS 앱만
firebase apps:list ios --project mg-games-prod
```

### 3.3 설정 파일 자동 다운로드

```bash
#!/bin/bash
# scripts/download_firebase_configs.sh
# google-services.json / GoogleService-Info.plist 일괄 다운로드

PROJECT_ID="mg-games-prod"
REPOS_DIR="d:/mg-games/repos"

# 앱 목록 가져오기
apps=$(firebase apps:list --project "$PROJECT_ID" --json)

# Android 앱 설정 다운로드
echo "$apps" | jq -r '.result[] | select(.platform == "ANDROID") | .appId + ":" + .displayName' | while IFS=':' read -r app_id display_name; do
  # 게임 ID 추출 (MG-0001 -> 0001)
  game_id=$(echo "$display_name" | grep -oP '\d{4}')

  if [ -n "$game_id" ]; then
    output_dir="$REPOS_DIR/mg-game-$game_id/game/android/app"
    mkdir -p "$output_dir"

    firebase apps:sdkconfig android "$app_id" \
      --project "$PROJECT_ID" \
      --out "$output_dir/google-services.json"

    echo "✓ Downloaded: mg-game-$game_id/google-services.json"
  fi
done

# iOS 앱 설정 다운로드
echo "$apps" | jq -r '.result[] | select(.platform == "IOS") | .appId + ":" + .displayName' | while IFS=':' read -r app_id display_name; do
  game_id=$(echo "$display_name" | grep -oP '\d{4}')

  if [ -n "$game_id" ]; then
    output_dir="$REPOS_DIR/mg-game-$game_id/game/ios/Runner"
    mkdir -p "$output_dir"

    firebase apps:sdkconfig ios "$app_id" \
      --project "$PROJECT_ID" \
      --out "$output_dir/GoogleService-Info.plist"

    echo "✓ Downloaded: mg-game-$game_id/GoogleService-Info.plist"
  fi
done
```

---

## 4. Firebase 서비스 활성화

### 4.1 서비스별 활성화 명령

```bash
PROJECT_ID="mg-games-prod"

# Analytics는 프로젝트 생성 시 자동 활성화됨

# Crashlytics 활성화
firebase crashlytics:enable --project "$PROJECT_ID"

# Performance Monitoring 활성화
firebase perf:enable --project "$PROJECT_ID"

# App Distribution 활성화
firebase appdistribution:enable --project "$PROJECT_ID"
```

### 4.2 Firestore 설정 (Level A 게임용)
```bash
# Firestore 데이터베이스 생성
firebase firestore:databases:create \
  --project "$PROJECT_ID" \
  --location asia-northeast3

# 보안 규칙 배포
firebase deploy --only firestore:rules --project "$PROJECT_ID"
```

### 4.3 Cloud Functions 배포
```bash
# Functions 배포
firebase deploy --only functions --project "$PROJECT_ID"

# 특정 함수만 배포
firebase deploy --only functions:verifyReceipt,functions:dailyReport --project "$PROJECT_ID"
```

---

## 5. Remote Config CLI 자동화

### 5.1 현재 설정 내보내기
```bash
firebase remoteconfig:get \
  --project mg-games-prod \
  --output remote_config_backup.json
```

### 5.2 설정 가져오기/배포
```bash
# JSON 파일에서 설정 배포
firebase remoteconfig:publish remote_config.json \
  --project mg-games-prod
```

### 5.3 Remote Config 템플릿 (JSON)

```json
{
  "parameters": {
    "daily_reward_coins": {
      "defaultValue": { "value": "100" },
      "description": "일일 보상 코인",
      "valueType": "NUMBER"
    },
    "energy_max": {
      "defaultValue": { "value": "5" },
      "description": "최대 에너지",
      "valueType": "NUMBER"
    },
    "energy_regen_minutes": {
      "defaultValue": { "value": "20" },
      "description": "에너지 충전 시간 (분)",
      "valueType": "NUMBER"
    },
    "interstitial_cooldown_sec": {
      "defaultValue": { "value": "180" },
      "description": "인터스티셜 쿨다운 (초)",
      "valueType": "NUMBER"
    },
    "rewarded_daily_limit": {
      "defaultValue": { "value": "10" },
      "description": "일일 리워드 광고 제한",
      "valueType": "NUMBER"
    },
    "maintenance_mode": {
      "defaultValue": { "value": "false" },
      "description": "점검 모드",
      "valueType": "BOOLEAN"
    },
    "maintenance_message": {
      "defaultValue": { "value": "서버 점검 중입니다." },
      "description": "점검 메시지",
      "valueType": "STRING"
    },
    "current_event_id": {
      "defaultValue": { "value": "" },
      "description": "현재 이벤트 ID",
      "valueType": "STRING"
    },
    "feature_social_enabled": {
      "defaultValue": { "value": "true" },
      "description": "소셜 기능 활성화",
      "valueType": "BOOLEAN"
    },
    "ab_test_enabled": {
      "defaultValue": { "value": "false" },
      "description": "A/B 테스트 활성화",
      "valueType": "BOOLEAN"
    }
  },
  "conditions": [
    {
      "name": "Level_A_Games",
      "expression": "app.id.matches('com\\.mggames\\.game00(2[5-9]|3[0-6])')"
    },
    {
      "name": "Region_India",
      "expression": "app.id.matches('com\\.mggames\\.game00(3[7-9]|40)')"
    },
    {
      "name": "Region_LATAM",
      "expression": "app.id.matches('com\\.mggames\\.game004[1-4]')"
    },
    {
      "name": "Region_SEA",
      "expression": "app.id.matches('com\\.mggames\\.game004[5-8]')"
    },
    {
      "name": "Region_Africa",
      "expression": "app.id.matches('com\\.mggames\\.game00(49|5[0-2])')"
    }
  ],
  "parameterGroups": {
    "Economy": {
      "description": "경제 관련 설정",
      "parameters": {
        "daily_reward_coins": {},
        "energy_max": {},
        "energy_regen_minutes": {}
      }
    },
    "Ads": {
      "description": "광고 관련 설정",
      "parameters": {
        "interstitial_cooldown_sec": {},
        "rewarded_daily_limit": {}
      }
    },
    "System": {
      "description": "시스템 설정",
      "parameters": {
        "maintenance_mode": {},
        "maintenance_message": {}
      }
    }
  },
  "version": {
    "versionNumber": "1"
  }
}
```

### 5.4 조건부 값 설정

```json
{
  "parameters": {
    "energy_max": {
      "defaultValue": { "value": "5" },
      "conditionalValues": {
        "Level_A_Games": { "value": "120" }
      }
    },
    "gacha_pity_threshold": {
      "defaultValue": { "value": "0" },
      "conditionalValues": {
        "Level_A_Games": { "value": "90" }
      }
    },
    "starter_pack_price_usd": {
      "defaultValue": { "value": "1.99" },
      "conditionalValues": {
        "Region_India": { "value": "0.99" },
        "Region_LATAM": { "value": "0.99" },
        "Region_SEA": { "value": "0.99" },
        "Region_Africa": { "value": "0.99" }
      }
    }
  }
}
```

---

## 6. BigQuery 연동 CLI

### 6.1 BigQuery Export 활성화
```bash
# Firebase 프로젝트의 BigQuery 연동 (Google Cloud Console에서)
gcloud services enable bigquery.googleapis.com \
  --project mg-games-prod

# BigQuery 데이터셋 생성
bq mk --location=asia-northeast3 \
  --dataset mg-games-prod:analytics_export
```

### 6.2 BigQuery 쿼리 실행
```bash
# DAU 쿼리
bq query --use_legacy_sql=false '
SELECT
  event_date,
  app_info.id as game_id,
  COUNT(DISTINCT user_pseudo_id) as dau
FROM `mg-games-prod.analytics_*.events_*`
WHERE event_name = "session_start"
  AND _TABLE_SUFFIX >= FORMAT_DATE("%Y%m%d", DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY))
GROUP BY 1, 2
ORDER BY 1 DESC, 3 DESC
'
```

### 6.3 스케줄된 쿼리 생성
```bash
# 일일 리포트 스케줄 쿼리
bq mk --transfer_config \
  --target_dataset=analytics_reports \
  --display_name="Daily KPI Report" \
  --schedule="every day 09:00" \
  --data_source=scheduled_query \
  --params='{
    "query": "SELECT ... FROM analytics_*.events_* ...",
    "destination_table_name_template": "daily_kpi_{run_date}",
    "write_disposition": "WRITE_TRUNCATE"
  }'
```

---

## 7. 일괄 자동화 스크립트

### 7.1 전체 셋업 스크립트

```bash
#!/bin/bash
# scripts/setup_firebase_complete.sh
# MG-Games 52종 Firebase 전체 자동 셋업

set -e

PROJECT_ID="${1:-mg-games-prod}"
REPOS_DIR="${2:-d:/mg-games/repos}"

echo "=========================================="
echo "MG-Games Firebase Complete Setup"
echo "Project: $PROJECT_ID"
echo "Repos: $REPOS_DIR"
echo "=========================================="

# 1. 프로젝트 확인/생성
echo ""
echo "[1/7] Checking project..."
if ! firebase projects:list | grep -q "$PROJECT_ID"; then
  echo "Creating project: $PROJECT_ID"
  firebase projects:create "$PROJECT_ID" --display-name "MG-Games Production"
else
  echo "Project exists: $PROJECT_ID"
fi

# 2. 52개 앱 등록
echo ""
echo "[2/7] Registering apps..."
for i in $(seq -f "%04g" 1 52); do
  package="com.mggames.game$i"

  # Android
  if ! firebase apps:list android --project "$PROJECT_ID" | grep -q "$package"; then
    firebase apps:create android \
      --project "$PROJECT_ID" \
      --package-name "$package" \
      --display-name "MG-$i Android"
  fi

  # iOS
  if ! firebase apps:list ios --project "$PROJECT_ID" | grep -q "$package"; then
    firebase apps:create ios \
      --project "$PROJECT_ID" \
      --bundle-id "$package" \
      --display-name "MG-$i iOS"
  fi

  echo "✓ MG-$i registered"
done

# 3. 설정 파일 다운로드
echo ""
echo "[3/7] Downloading config files..."
for i in $(seq -f "%04g" 1 52); do
  game_dir="$REPOS_DIR/mg-game-$i/game"

  if [ -d "$game_dir" ]; then
    # Android
    mkdir -p "$game_dir/android/app"
    app_id=$(firebase apps:list android --project "$PROJECT_ID" --json | \
      jq -r ".result[] | select(.displayName | contains(\"MG-$i\")) | .appId" | head -1)

    if [ -n "$app_id" ]; then
      firebase apps:sdkconfig android "$app_id" \
        --project "$PROJECT_ID" \
        --out "$game_dir/android/app/google-services.json"
    fi

    # iOS
    mkdir -p "$game_dir/ios/Runner"
    app_id=$(firebase apps:list ios --project "$PROJECT_ID" --json | \
      jq -r ".result[] | select(.displayName | contains(\"MG-$i\")) | .appId" | head -1)

    if [ -n "$app_id" ]; then
      firebase apps:sdkconfig ios "$app_id" \
        --project "$PROJECT_ID" \
        --out "$game_dir/ios/Runner/GoogleService-Info.plist"
    fi

    echo "✓ MG-$i configs downloaded"
  fi
done

# 4. Remote Config 배포
echo ""
echo "[4/7] Deploying Remote Config..."
if [ -f "remote_config.json" ]; then
  firebase remoteconfig:publish remote_config.json --project "$PROJECT_ID"
  echo "✓ Remote Config deployed"
else
  echo "⚠ remote_config.json not found, skipping"
fi

# 5. Firestore 규칙 배포
echo ""
echo "[5/7] Deploying Firestore rules..."
if [ -f "firestore.rules" ]; then
  firebase deploy --only firestore:rules --project "$PROJECT_ID"
  echo "✓ Firestore rules deployed"
else
  echo "⚠ firestore.rules not found, skipping"
fi

# 6. Cloud Functions 배포
echo ""
echo "[6/7] Deploying Cloud Functions..."
if [ -d "functions" ]; then
  cd functions && npm ci && cd ..
  firebase deploy --only functions --project "$PROJECT_ID"
  echo "✓ Cloud Functions deployed"
else
  echo "⚠ functions directory not found, skipping"
fi

# 7. 결과 요약
echo ""
echo "[7/7] Setup Summary"
echo "=========================================="
firebase apps:list --project "$PROJECT_ID"
echo ""
echo "✅ Firebase setup complete!"
echo ""
echo "Next steps:"
echo "1. Enable BigQuery Export in Firebase Console"
echo "2. Set up billing (Blaze plan) for Cloud Functions"
echo "3. Configure FCM server key for push notifications"
echo "4. Add SHA-1 fingerprints for Android apps"
```

### 7.2 PowerShell 버전 (Windows)

```powershell
# scripts/setup_firebase_complete.ps1
# MG-Games 52종 Firebase 전체 자동 셋업 (Windows)

param(
    [string]$ProjectId = "mg-games-prod",
    [string]$ReposDir = "d:\mg-games\repos"
)

Write-Host "==========================================" -ForegroundColor Cyan
Write-Host "MG-Games Firebase Complete Setup" -ForegroundColor Cyan
Write-Host "Project: $ProjectId"
Write-Host "Repos: $ReposDir"
Write-Host "==========================================" -ForegroundColor Cyan

# 1. 52개 앱 등록
Write-Host "`n[1/5] Registering apps..." -ForegroundColor Yellow

for ($i = 1; $i -le 52; $i++) {
    $gameId = $i.ToString("D4")
    $package = "com.mggames.game$gameId"

    # Android
    firebase apps:create android `
        --project $ProjectId `
        --package-name $package `
        --display-name "MG-$gameId Android" 2>$null

    # iOS
    firebase apps:create ios `
        --project $ProjectId `
        --bundle-id $package `
        --display-name "MG-$gameId iOS" 2>$null

    Write-Host "✓ MG-$gameId registered" -ForegroundColor Green
}

# 2. 설정 파일 다운로드
Write-Host "`n[2/5] Downloading config files..." -ForegroundColor Yellow

$appsJson = firebase apps:list --project $ProjectId --json | ConvertFrom-Json

for ($i = 1; $i -le 52; $i++) {
    $gameId = $i.ToString("D4")
    $gameDir = Join-Path $ReposDir "mg-game-$gameId\game"

    if (Test-Path $gameDir) {
        # Android
        $androidDir = Join-Path $gameDir "android\app"
        New-Item -ItemType Directory -Force -Path $androidDir | Out-Null

        $androidApp = $appsJson.result | Where-Object {
            $_.platform -eq "ANDROID" -and $_.displayName -like "*MG-$gameId*"
        } | Select-Object -First 1

        if ($androidApp) {
            firebase apps:sdkconfig android $androidApp.appId `
                --project $ProjectId `
                --out "$androidDir\google-services.json"
        }

        # iOS
        $iosDir = Join-Path $gameDir "ios\Runner"
        New-Item -ItemType Directory -Force -Path $iosDir | Out-Null

        $iosApp = $appsJson.result | Where-Object {
            $_.platform -eq "IOS" -and $_.displayName -like "*MG-$gameId*"
        } | Select-Object -First 1

        if ($iosApp) {
            firebase apps:sdkconfig ios $iosApp.appId `
                --project $ProjectId `
                --out "$iosDir\GoogleService-Info.plist"
        }

        Write-Host "✓ MG-$gameId configs downloaded" -ForegroundColor Green
    }
}

# 3. Remote Config 배포
Write-Host "`n[3/5] Deploying Remote Config..." -ForegroundColor Yellow
if (Test-Path "remote_config.json") {
    firebase remoteconfig:publish remote_config.json --project $ProjectId
    Write-Host "✓ Remote Config deployed" -ForegroundColor Green
}

# 4. Firestore 규칙 배포
Write-Host "`n[4/5] Deploying Firestore rules..." -ForegroundColor Yellow
if (Test-Path "firestore.rules") {
    firebase deploy --only firestore:rules --project $ProjectId
    Write-Host "✓ Firestore rules deployed" -ForegroundColor Green
}

# 5. Cloud Functions 배포
Write-Host "`n[5/5] Deploying Cloud Functions..." -ForegroundColor Yellow
if (Test-Path "functions") {
    Push-Location functions
    npm ci
    Pop-Location
    firebase deploy --only functions --project $ProjectId
    Write-Host "✓ Cloud Functions deployed" -ForegroundColor Green
}

Write-Host "`n==========================================" -ForegroundColor Cyan
Write-Host "✅ Firebase setup complete!" -ForegroundColor Green
Write-Host "==========================================" -ForegroundColor Cyan
```

---

## 8. CI/CD 통합

### 8.1 GitHub Actions에서 Firebase 사용

```yaml
# .github/workflows/firebase-deploy.yml
name: Firebase Deploy

on:
  push:
    branches: [main]
    paths:
      - 'firebase/**'
      - 'functions/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install Firebase CLI
        run: npm install -g firebase-tools

      - name: Install Functions dependencies
        run: cd functions && npm ci

      - name: Deploy to Firebase
        env:
          FIREBASE_TOKEN: ${{ secrets.FIREBASE_TOKEN }}
        run: |
          firebase deploy --only functions,firestore:rules \
            --project mg-games-prod \
            --token "$FIREBASE_TOKEN"
```

### 8.2 환경별 배포

```yaml
# .github/workflows/firebase-deploy-env.yml
name: Firebase Deploy (Environment)

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        type: choice
        options:
          - dev
          - staging
          - prod

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ github.event.inputs.environment }}
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install Firebase CLI
        run: npm install -g firebase-tools

      - name: Set Project ID
        id: project
        run: |
          case "${{ github.event.inputs.environment }}" in
            dev) echo "project_id=mg-games-dev" >> $GITHUB_OUTPUT ;;
            staging) echo "project_id=mg-games-staging" >> $GITHUB_OUTPUT ;;
            prod) echo "project_id=mg-games-prod" >> $GITHUB_OUTPUT ;;
          esac

      - name: Deploy
        env:
          FIREBASE_TOKEN: ${{ secrets.FIREBASE_TOKEN }}
        run: |
          firebase deploy \
            --project ${{ steps.project.outputs.project_id }} \
            --token "$FIREBASE_TOKEN"
```

---

## 9. 유용한 CLI 명령어 모음

### 프로젝트 관리
```bash
firebase projects:list                    # 프로젝트 목록
firebase projects:create <id>             # 프로젝트 생성
firebase use <project-id>                 # 프로젝트 선택
firebase use --add                        # 프로젝트 별칭 추가
```

### 앱 관리
```bash
firebase apps:list                        # 모든 앱 목록
firebase apps:list android                # Android 앱만
firebase apps:list ios                    # iOS 앱만
firebase apps:create android              # Android 앱 생성
firebase apps:create ios                  # iOS 앱 생성
firebase apps:sdkconfig android <app-id>  # 설정 파일 다운로드
```

### Remote Config
```bash
firebase remoteconfig:get                 # 현재 설정 보기
firebase remoteconfig:get --output f.json # 파일로 저장
firebase remoteconfig:publish config.json # 설정 배포
firebase remoteconfig:rollback            # 이전 버전으로 롤백
firebase remoteconfig:versions:list       # 버전 히스토리
```

### Hosting
```bash
firebase hosting:sites:list               # 사이트 목록
firebase hosting:sites:create <site>      # 사이트 생성
firebase deploy --only hosting            # 호스팅 배포
firebase hosting:channel:deploy preview   # 프리뷰 채널 배포
```

### Functions
```bash
firebase functions:list                   # 함수 목록
firebase functions:log                    # 로그 보기
firebase functions:log --only <fn-name>   # 특정 함수 로그
firebase functions:delete <fn-name>       # 함수 삭제
firebase functions:shell                  # 로컬 테스트 셸
```

### Firestore
```bash
firebase firestore:indexes                # 인덱스 보기
firebase firestore:delete <path>          # 문서 삭제
firebase emulators:start --only firestore # 에뮬레이터 시작
```

### App Distribution
```bash
firebase appdistribution:distribute app.apk \
  --app <app-id> \
  --groups "qa,beta" \
  --release-notes "Bug fixes"
```

---

## 10. 체크리스트

### CLI 셋업
- [ ] Firebase CLI 설치
- [ ] `firebase login` 완료
- [ ] CI용 토큰 생성 (`firebase login:ci`)
- [ ] gcloud CLI 설치 (BigQuery용)

### 프로젝트 셋업
- [ ] prod/staging/dev 프로젝트 생성
- [ ] 52개 Android 앱 등록
- [ ] 52개 iOS 앱 등록
- [ ] 설정 파일 다운로드 완료

### 서비스 배포
- [ ] Remote Config 배포
- [ ] Firestore 규칙 배포
- [ ] Cloud Functions 배포
- [ ] Hosting 배포 (필요 시)

### CI/CD 통합
- [ ] GitHub Secrets에 FIREBASE_TOKEN 설정
- [ ] 배포 워크플로우 구성
- [ ] 환경별 배포 테스트

---

*문서 작성: 2025-12-19*
*MG-Games DevOps팀*
