# MG-Games Firebase Console 셋업 가이드

> **문서 버전**: 1.0.0
> **최종 수정일**: 2025-12-19
> **대상**: 52종 게임 (MG-0001 ~ MG-0052)

---

## 📋 목차
1. [인프라 아키텍처 개요](#1-인프라-아키텍처-개요)
2. [Firebase 프로젝트 구조](#2-firebase-프로젝트-구조)
3. [Firebase Console 셋업 절차](#3-firebase-console-셋업-절차)
4. [서비스별 설정](#4-서비스별-설정)
5. [환경 분리 전략](#5-환경-분리-전략)
6. [보안 및 권한 설정](#6-보안-및-권한-설정)
7. [모니터링 및 알림](#7-모니터링-및-알림)
8. [비용 최적화](#8-비용-최적화)

---

## 1. 인프라 아키텍처 개요

### 1.1 전체 구조
```
┌────────────────────────────────────────────────────────────────────┐
│                      MG-Games Infrastructure                        │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐ │
│  │   Vercel     │  │ GitHub Pages │  │      Firebase           │ │
│  │              │  │              │  │                          │ │
│  │ 회사 홈페이지  │  │ 게임별 랜딩   │  │ • Analytics             │ │
│  │ mg-games.com │  │ (52개 페이지) │  │ • Remote Config         │ │
│  │              │  │              │  │ • Cloud Functions        │ │
│  └──────────────┘  └──────────────┘  │ • Crashlytics           │ │
│                                       │ • BigQuery Export       │ │
│                                       │ • Cloud Messaging       │ │
│                                       │ • App Distribution      │ │
│                                       └──────────────────────────┘ │
│                                                   │                │
│                                                   ▼                │
│                                       ┌──────────────────────────┐ │
│                                       │     Google Cloud         │ │
│                                       │                          │ │
│                                       │ • BigQuery (Analytics)   │ │
│                                       │ • Cloud Run (API)        │ │
│                                       │ • Cloud Storage          │ │
│                                       └──────────────────────────┘ │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 1.2 도메인 구조
| 용도 | 도메인 | 호스팅 |
|------|--------|--------|
| 회사 홈페이지 | `mg-games.com` | Vercel |
| 게임 랜딩페이지 | `mg-games.com/games/{game_id}` 또는 `{game_id}.mg-games.com` | GitHub Pages |
| API 서버 | `api.mg-games.com` | Cloud Run |
| CDN (에셋) | `cdn.mg-games.com` | Cloud Storage + CDN |

---

## 2. Firebase 프로젝트 구조

### 2.1 단일 프로젝트 구성 (권장)
```
Firebase 프로젝트: mg-games-prod
│
├── Apps (104개 = 52게임 × 2플랫폼)
│   ├── Android Apps
│   │   ├── com.mggames.game0001
│   │   ├── com.mggames.game0002
│   │   └── ... (52개)
│   └── iOS Apps
│       ├── com.mggames.game0001
│       ├── com.mggames.game0002
│       └── ... (52개)
│
├── 공유 서비스
│   ├── Analytics (게임별 app_id로 구분)
│   ├── Remote Config (조건으로 게임별 설정)
│   ├── Cloud Functions (공통 API)
│   ├── Crashlytics (앱별 자동 분리)
│   └── BigQuery Export (app_info.id로 쿼리)
│
└── 환경별 프로젝트 (선택)
    ├── mg-games-staging
    └── mg-games-dev
```

### 2.2 단일 프로젝트 장점

| 항목 | 장점 |
|------|------|
| **관리** | 하나의 콘솔에서 52개 게임 통합 관리 |
| **비용** | Blaze 플랜 1개만 필요, 무료 할당량 공유 |
| **분석** | 크로스 게임 분석 용이 (BigQuery 단일 데이터셋) |
| **배포** | Cloud Functions 한 번 배포로 전체 적용 |
| **설정** | Remote Config 조건으로 게임별 분기 |

### 2.3 게임 구분 방법

```
┌─────────────────────────────────────────────────────────────┐
│                   mg-games-prod 프로젝트                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Analytics         → app_info.id 로 게임 구분               │
│  Remote Config     → app.id 조건으로 게임별 값 설정          │
│  Crashlytics       → 앱별 자동 분리 (별도 설정 불필요)        │
│  Cloud Functions   → request.app_id 로 게임 식별            │
│  BigQuery          → app_info.id 컬럼으로 필터링            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Firebase Console 셋업 절차

### 3.1 프로젝트 생성

#### Step 1: Firebase Console 접속
```
https://console.firebase.google.com/
```

#### Step 2: 프로젝트 생성
1. "프로젝트 추가" 클릭
2. 프로젝트 이름: `mg-games-prod`
3. Google Analytics 활성화: **예**
4. Analytics 계정: 새로 생성 또는 기존 계정 선택
5. 위치: `asia-northeast3` (서울) 또는 `us-central1`

#### Step 3: Blaze 플랜 업그레이드
```
프로젝트 설정 > 사용량 및 결제 > Blaze 플랜으로 업그레이드
```
**필수**: Cloud Functions, BigQuery Export 사용을 위해 필요

### 3.2 52개 게임 앱 등록

#### CLI로 일괄 등록 (권장)
```bash
#!/bin/bash
# 52개 게임 Android/iOS 앱 일괄 등록

PROJECT_ID="mg-games-prod"

for i in $(seq -f "%04g" 1 52); do
  echo "Registering MG-$i..."

  # Android 앱
  firebase apps:create android \
    --project "$PROJECT_ID" \
    --package-name "com.mggames.game$i" \
    --display-name "MG-$i Android"

  # iOS 앱
  firebase apps:create ios \
    --project "$PROJECT_ID" \
    --bundle-id "com.mggames.game$i" \
    --display-name "MG-$i iOS"
done

echo "✅ 104개 앱 등록 완료 (52 Android + 52 iOS)"
```

#### 설정 파일 일괄 다운로드
```bash
#!/bin/bash
# google-services.json / GoogleService-Info.plist 다운로드

PROJECT_ID="mg-games-prod"
REPOS_DIR="d:/mg-games/repos"

# 앱 목록 가져오기
apps=$(firebase apps:list --project "$PROJECT_ID" --json)

for i in $(seq -f "%04g" 1 52); do
  game_dir="$REPOS_DIR/mg-game-$i/game"

  # Android 설정
  app_id=$(echo "$apps" | jq -r ".result[] | select(.displayName==\"MG-$i Android\") | .appId")
  if [ -n "$app_id" ]; then
    mkdir -p "$game_dir/android/app"
    firebase apps:sdkconfig android "$app_id" \
      --project "$PROJECT_ID" \
      --out "$game_dir/android/app/google-services.json"
  fi

  # iOS 설정
  app_id=$(echo "$apps" | jq -r ".result[] | select(.displayName==\"MG-$i iOS\") | .appId")
  if [ -n "$app_id" ]; then
    mkdir -p "$game_dir/ios/Runner"
    firebase apps:sdkconfig ios "$app_id" \
      --project "$PROJECT_ID" \
      --out "$game_dir/ios/Runner/GoogleService-Info.plist"
  fi

  echo "✓ MG-$i configs downloaded"
done
```

#### Console에서 수동 등록 (필요 시)
```
1. 프로젝트 개요 > 앱 추가 > Android/iOS 선택
2. 패키지/번들 ID: com.mggames.game0001
3. 앱 닉네임: MG-0001 Android (또는 iOS)
4. 설정 파일 다운로드
```

### 3.3 디렉토리 구조
```
repos/mg-game-0001/
├── game/
│   ├── android/
│   │   └── app/
│   │       └── google-services.json
│   └── ios/
│       └── Runner/
│           └── GoogleService-Info.plist
└── firebase/
    ├── firebase.json
    └── .firebaserc
```

---

## 4. 서비스별 설정

### 4.1 Firebase Analytics

#### 설정 위치
```
Firebase Console > Analytics > 설정
```

#### 필수 설정
1. **데이터 보관 기간**: 14개월 (최대)
2. **Google 신호 데이터**: 활성화
3. **BigQuery 연결**: 활성화

#### 커스텀 이벤트 등록
```
Analytics > 이벤트 > 이벤트 만들기
```

**필수 커스텀 이벤트**:
| 이벤트 | 설명 | 매개변수 |
|--------|------|----------|
| `level_complete` | 레벨 완료 | level, score, stars |
| `currency_earned` | 재화 획득 | currency_type, amount, source |
| `currency_spent` | 재화 소비 | currency_type, amount, item |
| `ad_reward_claimed` | 광고 보상 | placement, reward_type |
| `gacha_pull` | 가챠 (Level A) | banner_id, pull_count, pity |

#### 사용자 속성 등록
```
Analytics > 사용자 > 사용자 정의 정의
```

| 속성 | 설명 |
|------|------|
| `game_id` | 게임 식별자 (game_0001) |
| `user_level` | 유저 레벨 |
| `total_spend_usd` | 총 결제 금액 |
| `is_payer` | 결제 유저 여부 |
| `ab_test_group` | A/B 테스트 그룹 |

### 4.2 Remote Config

#### 설정 위치
```
Firebase Console > Remote Config
```

#### 초기 파라미터 설정
```json
{
  "daily_reward_coins": 100,
  "energy_max": 5,
  "energy_regen_minutes": 20,
  "interstitial_cooldown_sec": 180,
  "rewarded_daily_limit": 10,
  "maintenance_mode": false,
  "current_event_id": "",
  "ab_test_enabled": false
}
```

#### 게임 그룹별 조건 설정 (단일 프로젝트에서 52개 게임 구분)

```
┌─────────────────────────────────────────────────────────────┐
│  Remote Config 조건으로 52개 게임 설정 분기                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  조건명: Level_A_Games                                       │
│  조건: app.id.matches('com\\.mggames\\.game00(2[5-9]|3[0-6])')│
│  → MG-0025 ~ MG-0036 (Level A JRPG 12종)                    │
│                                                             │
│  조건명: Region_India                                        │
│  조건: app.id.matches('com\\.mggames\\.game00(3[7-9]|40)')   │
│  → MG-0037 ~ MG-0040 (인도 4종)                             │
│                                                             │
│  조건명: Region_LATAM                                        │
│  조건: app.id.matches('com\\.mggames\\.game004[1-4]')        │
│  → MG-0041 ~ MG-0044 (남미 4종)                             │
│                                                             │
│  조건명: Region_SEA                                          │
│  조건: app.id.matches('com\\.mggames\\.game004[5-8]')        │
│  → MG-0045 ~ MG-0048 (동남아 4종)                           │
│                                                             │
│  조건명: Region_Africa                                       │
│  조건: app.id.matches('com\\.mggames\\.game00(49|5[0-2])')   │
│  → MG-0049 ~ MG-0052 (아프리카 4종)                         │
│                                                             │
│  기본값: MG-0001 ~ MG-0024 (Year 1-2 캐주얼 24종)            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**조건별 파라미터 값**:
| 파라미터 | 기본값 | Level_A | Emerging Markets |
|----------|--------|---------|------------------|
| energy_max | 5 | 120 | 5 |
| energy_regen_minutes | 20 | 5 | 15 |
| starter_pack_price_usd | 1.99 | 4.99 | 0.99 |
| gacha_pity_threshold | 0 | 90 | 0 |
| interstitial_cooldown_sec | 180 | 300 | 120 |

#### CLI로 Remote Config 배포
```bash
# 설정 배포
firebase remoteconfig:publish remote_config.json --project mg-games-prod

# 현재 설정 백업
firebase remoteconfig:get --project mg-games-prod -o backup.json

# 버전 히스토리
firebase remoteconfig:versions:list --project mg-games-prod
```

### 4.3 Cloud Functions

#### 필수 함수
```
functions/
├── src/
│   ├── iap/
│   │   └── verifyReceipt.ts      # 영수증 검증
│   ├── analytics/
│   │   └── dailyReport.ts        # 일일 리포트
│   ├── events/
│   │   └── eventScheduler.ts     # 이벤트 스케줄러
│   └── admin/
│       └── configUpdate.ts       # 설정 업데이트
└── package.json
```

#### 배포
```bash
firebase deploy --only functions
```

#### 환경 변수 설정
```bash
firebase functions:config:set \
  slack.webhook_url="https://hooks.slack.com/..." \
  admin.api_key="secret-key"
```

### 4.4 Crashlytics

#### 설정 위치
```
Firebase Console > Crashlytics
```

#### Flutter 통합
```yaml
# pubspec.yaml
dependencies:
  firebase_crashlytics: ^3.4.0

# main.dart
await FirebaseCrashlytics.instance.setCrashlyticsCollectionEnabled(true);
FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterError;
```

#### 커스텀 키 설정
```dart
FirebaseCrashlytics.instance.setCustomKey('game_id', 'game_0001');
FirebaseCrashlytics.instance.setCustomKey('user_level', userLevel);
FirebaseCrashlytics.instance.setCustomKey('device_tier', deviceTier);
```

### 4.5 BigQuery Export

#### 설정 위치
```
Firebase Console > 프로젝트 설정 > 통합 > BigQuery
```

#### 활성화 절차
1. BigQuery 연결 활성화
2. 데이터 세트 위치: `asia-northeast3` (서울)
3. 내보내기 유형: **스트리밍** (실시간)

#### 생성되는 테이블
```sql
-- Analytics 이벤트 테이블
analytics_<property_id>.events_*

-- 스키마 예시
SELECT
  event_name,
  event_timestamp,
  user_pseudo_id,
  device.category,
  geo.country,
  app_info.id as game_id,
  -- 이벤트 파라미터
  (SELECT value.int_value FROM UNNEST(event_params) WHERE key = 'level') as level,
  (SELECT value.int_value FROM UNNEST(event_params) WHERE key = 'score') as score
FROM `mg-games-prod.analytics_*.events_*`
WHERE event_name = 'level_complete'
```

### 4.6 Cloud Messaging (FCM)

#### 설정 위치
```
Firebase Console > Cloud Messaging
```

#### 서버 키 획득
```
프로젝트 설정 > 클라우드 메시징 > 서버 키
```

#### 토픽 구조
```
/topics/game_0001               # 게임별 전체
/topics/game_0001_event         # 게임별 이벤트
/topics/all_games               # 전체 게임
/topics/level_a_games           # Level A 게임
/topics/casual_games            # 캐주얼 게임
/topics/region_india            # 지역별
/topics/region_latam
/topics/region_sea
/topics/region_africa
```

### 4.7 App Distribution (테스트 배포)

#### 설정 위치
```
Firebase Console > App Distribution
```

#### 테스터 그룹
| 그룹 | 설명 |
|------|------|
| `internal` | 내부 테스터 |
| `qa` | QA 팀 |
| `beta` | 베타 테스터 |
| `regional_india` | 인도 테스터 |
| `regional_latam` | 남미 테스터 |

#### CI/CD 통합
```yaml
# .github/workflows/deploy-test.yml
- name: Upload to Firebase App Distribution
  uses: wzieba/Firebase-Distribution-Github-Action@v1
  with:
    appId: ${{ secrets.FIREBASE_APP_ID }}
    token: ${{ secrets.FIREBASE_TOKEN }}
    groups: qa
    file: build/app/outputs/flutter-apk/app-release.apk
```

---

## 5. 환경 분리 전략

### 5.1 환경별 프로젝트
| 환경 | 프로젝트 ID | 용도 |
|------|------------|------|
| Production | `mg-games-prod` | 실서비스 |
| Staging | `mg-games-staging` | 출시 전 테스트 |
| Development | `mg-games-dev` | 개발 |

### 5.2 Flutter 환경 설정

#### 디렉토리 구조
```
game/
├── lib/
│   ├── config/
│   │   ├── env_config.dart
│   │   └── firebase_options_*.dart
│   └── main_*.dart
├── android/
│   └── app/
│       ├── google-services.json          # prod
│       ├── src/
│       │   ├── dev/google-services.json
│       │   └── staging/google-services.json
└── ios/
    └── Runner/
        ├── GoogleService-Info.plist      # prod
        └── Firebase/
            ├── dev/GoogleService-Info.plist
            └── staging/GoogleService-Info.plist
```

#### 빌드 명령
```bash
# Development
flutter run --dart-define=ENVIRONMENT=dev

# Staging
flutter run --dart-define=ENVIRONMENT=staging

# Production
flutter build apk --dart-define=ENVIRONMENT=prod
```

### 5.3 환경별 Firebase 초기화
```dart
// lib/config/firebase_init.dart
Future<void> initializeFirebase() async {
  final env = const String.fromEnvironment('ENVIRONMENT', defaultValue: 'prod');

  FirebaseOptions options;
  switch (env) {
    case 'dev':
      options = FirebaseOptionsDev.currentPlatform;
      break;
    case 'staging':
      options = FirebaseOptionsStaging.currentPlatform;
      break;
    default:
      options = FirebaseOptionsProd.currentPlatform;
  }

  await Firebase.initializeApp(options: options);
}
```

---

## 6. 보안 및 권한 설정

### 6.1 IAM 권한 구조

#### 역할 정의
| 역할 | 권한 | 대상 |
|------|------|------|
| `Owner` | 모든 권한 | 프로젝트 관리자 |
| `Editor` | 대부분 수정 | 개발팀 리드 |
| `Viewer` | 읽기 전용 | 분석팀 |
| `Firebase Admin` | Firebase 서비스 | 백엔드 개발자 |
| `Analytics Viewer` | Analytics 읽기 | 마케팅팀 |

#### 서비스 계정
```
Firebase Console > 프로젝트 설정 > 서비스 계정
```

| 계정 | 용도 | 파일명 |
|------|------|--------|
| `firebase-adminsdk` | 백엔드 API | `service-account-prod.json` |
| `github-actions` | CI/CD | `ci-service-account.json` |
| `bigquery-export` | BigQuery 접근 | (자동 생성) |

### 6.2 API 키 제한

#### Android 키 제한
```
Google Cloud Console > API 및 서비스 > 사용자 인증 정보
> Android 키 선택 > 애플리케이션 제한사항

- Android 앱으로 제한
- 패키지 이름: com.mggames.game* (와일드카드 불가, 개별 등록)
- SHA-1 인증서 지문 추가
```

#### iOS 키 제한
```
- iOS 앱으로 제한
- 번들 ID: com.mggames.game*
```

### 6.3 보안 규칙 (Firestore - Level A용)
```javascript
// firestore.rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // 사용자 데이터
    match /users/{userId} {
      allow read, write: if request.auth.uid == userId;
    }

    // 길드 데이터 (Level A)
    match /guilds/{guildId} {
      allow read: if request.auth != null;
      allow write: if isGuildMember(guildId);
    }

    // 랭킹 (읽기 전용)
    match /rankings/{gameId}/{document=**} {
      allow read: if true;
      allow write: if false; // Cloud Functions만 가능
    }

    function isGuildMember(guildId) {
      return exists(/databases/$(database)/documents/guilds/$(guildId)/members/$(request.auth.uid));
    }
  }
}
```

---

## 7. 모니터링 및 알림

### 7.1 알림 설정

#### Crashlytics 알림
```
Crashlytics > 설정 > 알림

- 새로운 비정상 종료 문제: Slack 알림
- 회귀 문제: 이메일 + Slack
- 속도 알림 (비정상 종료율 > 1%): 즉시 알림
```

#### 성능 모니터링 알림
```
Performance > 설정 > 알림

- 앱 시작 시간 > 5초
- HTTP 요청 성공률 < 95%
- 사용자 지정 추적 임계값
```

### 7.2 Slack 통합

#### Webhook 설정
```javascript
// functions/src/notifications/slack.ts
const SLACK_WEBHOOK = functions.config().slack.webhook_url;

export async function sendSlackAlert(message: string, channel: string) {
  await fetch(SLACK_WEBHOOK, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      channel,
      text: message,
      username: 'MG-Games Bot',
      icon_emoji: ':video_game:'
    })
  });
}
```

#### 채널 구조
| 채널 | 알림 유형 |
|------|----------|
| `#mg-alerts-critical` | 크래시, 서버 장애 |
| `#mg-alerts-revenue` | 매출 이상 |
| `#mg-alerts-qa` | QA 결과 |
| `#mg-daily-report` | 일일 리포트 |

### 7.3 대시보드

#### Firebase Console 대시보드
- 프로젝트 개요: 전체 현황
- Analytics Dashboard: 유저 행동
- Crashlytics: 안정성

#### 외부 대시보드 (Looker Studio)
```
BigQuery 데이터 기반 대시보드
- 게임별 DAU/MAU
- 매출 현황
- 코호트 리텐션
- 광고 수익
```

---

## 8. 비용 최적화

### 8.1 예상 비용 (52개 게임 기준)

| 서비스 | 무료 한도 | 예상 월 비용 |
|--------|----------|-------------|
| Analytics | 무료 | $0 |
| Remote Config | 무료 | $0 |
| Crashlytics | 무료 | $0 |
| Cloud Functions | 200만 호출/월 | $50~200 |
| BigQuery | 10GB 처리/월 | $50~300 |
| Cloud Storage | 5GB | $10~50 |
| Cloud Messaging | 무료 | $0 |

**예상 총 비용**: $150~600/월 (트래픽에 따라 변동)

### 8.2 비용 절감 전략

#### BigQuery 최적화
```sql
-- 파티션 테이블 사용
CREATE TABLE analytics.events_daily
PARTITION BY DATE(event_timestamp)
AS SELECT * FROM ...

-- 필요한 컬럼만 선택
SELECT event_name, user_id, event_params
FROM ... -- * 대신 특정 컬럼
```

#### Cloud Functions 최적화
```javascript
// 콜드 스타트 최소화
exports.api = functions
  .runWith({ minInstances: 1 })
  .https.onRequest(app);

// 메모리 최적화
exports.lightFunction = functions
  .runWith({ memory: '128MB' })
  .pubsub.topic('...')
```

#### 예산 알림 설정
```
Google Cloud Console > 결제 > 예산 및 알림

- 월 예산: $500
- 알림 임계값: 50%, 80%, 100%
- 이메일 + Slack 알림
```

---

## 9. 체크리스트

### 초기 셋업
- [ ] Firebase 프로젝트 생성 (prod/staging/dev)
- [ ] Blaze 플랜 업그레이드
- [ ] 52개 Android 앱 등록
- [ ] 52개 iOS 앱 등록
- [ ] Analytics 활성화
- [ ] BigQuery 연동
- [ ] Remote Config 초기값 설정

### 서비스 설정
- [ ] Crashlytics 통합
- [ ] Cloud Functions 배포
- [ ] Cloud Messaging 설정
- [ ] App Distribution 테스터 그룹

### 보안
- [ ] IAM 역할 설정
- [ ] 서비스 계정 생성
- [ ] API 키 제한
- [ ] 보안 규칙 배포

### 모니터링
- [ ] Slack 웹훅 설정
- [ ] 알림 규칙 구성
- [ ] 예산 알림 설정
- [ ] Looker Studio 대시보드

---

## 부록: 유용한 명령어

### Firebase CLI
```bash
# 로그인
firebase login

# 프로젝트 목록
firebase projects:list

# 앱 목록
firebase apps:list --project mg-games-prod

# Remote Config 가져오기
firebase remoteconfig:get --project mg-games-prod

# Functions 로그
firebase functions:log --project mg-games-prod
```

### gcloud CLI
```bash
# BigQuery 쿼리
bq query --use_legacy_sql=false \
  'SELECT * FROM `mg-games-prod.analytics_*.events_*` LIMIT 10'

# 서비스 계정 키 생성
gcloud iam service-accounts keys create key.json \
  --iam-account=firebase-adminsdk@mg-games-prod.iam.gserviceaccount.com
```

---

*문서 작성: 2025-12-19*
*MG-Games 인프라팀*
