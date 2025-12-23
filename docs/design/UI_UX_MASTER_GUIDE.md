# MG-Games UI/UX 마스터 가이드

> **문서 버전**: 1.0.0
> **최종 수정**: 2025-12-19
> **적용 대상**: 52종 전체 게임 (MG-0001 ~ MG-0052)

---

## 1. 개요

### 1.1 목적
본 문서는 MG-Games 52종 게임의 일관된 사용자 경험을 보장하기 위한 UI/UX 디자인 원칙과 가이드라인을 정의합니다.

### 1.2 적용 범위
| 카테고리 | 게임 ID | 게임 수 |
|----------|---------|---------|
| Year 1 Core | MG-0001 ~ MG-0012 | 12종 |
| Year 2 Core | MG-0013 ~ MG-0024 | 12종 |
| Level A JRPG | MG-0025 ~ MG-0036 | 12종 |
| Casual Emerging | MG-0037 ~ MG-0052 | 16종 |

### 1.3 관련 문서
- [SCREEN_ORIENTATION_GUIDE.md](./SCREEN_ORIENTATION_GUIDE.md) - 화면 방향 가이드
- [SAFE_AREA_GUIDE.md](./SAFE_AREA_GUIDE.md) - Safe Area 가이드
- [ACCESSIBILITY_GUIDE.md](./ACCESSIBILITY_GUIDE.md) - 접근성 가이드
- [DEVICE_OPTIMIZATION_GUIDE.md](./DEVICE_OPTIMIZATION_GUIDE.md) - 디바이스 최적화 가이드

---

## 2. 디자인 원칙

### 2.1 핵심 원칙 (FOCUS)

```
F - Functional    : 기능이 명확하고 직관적
O - Optimized     : 성능과 메모리 최적화
C - Consistent    : 일관된 시각 언어와 인터랙션
U - Universal     : 다양한 기기와 사용자 수용
S - Simple        : 불필요한 복잡성 제거
```

### 2.2 사용자 중심 설계

#### 2.2.1 3초 규칙
- 유저가 화면을 본 후 **3초 이내**에 다음 행동을 파악할 수 있어야 함
- 핵심 CTA(Call-to-Action)는 시선 흐름의 끝에 배치

#### 2.2.2 2-탭 규칙
- 모든 주요 기능은 **최대 2번의 탭**으로 도달 가능해야 함
- 예외: 설정, 고객센터 등 저빈도 기능

#### 2.2.3 피츠의 법칙 (Fitts's Law)
```
T = a + b × log₂(D/W + 1)

T: 도달 시간
D: 목표까지 거리
W: 목표 크기
```
- 자주 사용하는 버튼은 **크고 가깝게**
- 위험한 액션(삭제, 초기화)은 **작고 멀게**

---

## 3. 게임 티어별 UI 복잡도

### 3.1 티어 분류

| 티어 | 대상 게임 | HUD 요소 | 최소 버튼 | 정보 계층 |
|------|-----------|----------|-----------|-----------|
| **Tier 1: Simple** | 0001~0012, 0037~0052 | 3~5개 | 48dp | 2단계 |
| **Tier 2: Medium** | 0013~0024 | 5~8개 | 44dp | 3단계 |
| **Tier 3: Complex** | 0025~0036 | 8~15개 | 40dp | 4단계 |

### 3.2 Tier 1: Simple (캐주얼/하이퍼캐주얼)

```
┌─────────────────────────────────────┐
│  [점수: 1,234]          [⚙️]        │  ← 상시 HUD (최소화)
│                                     │
│                                     │
│                                     │
│           게임 영역                  │  ← 최대 공간 확보
│                                     │
│                                     │
│                                     │
│         [PLAY 버튼]                  │  ← 단일 CTA
└─────────────────────────────────────┘
```

**특징:**
- 한손 조작 최적화
- 정보 최소화 (점수, 자원, 시간)
- 대형 터치 영역 (56dp 권장)
- 직관적 제스처 (탭, 스와이프)

### 3.3 Tier 2: Medium (미드코어)

```
┌─────────────────────────────────────┐
│  [레벨] [자원1] [자원2]    [⚙️][📬] │  ← 상시 HUD
├─────────────────────────────────────┤
│                                     │
│           게임 영역                  │
│                                     │
├─────────────────────────────────────┤
│  [스킬1] [스킬2] [스킬3] [궁극기]   │  ← 액션 바
└─────────────────────────────────────┘
```

**특징:**
- 다중 자원 표시
- 스킬/액션 바
- 탭 기반 네비게이션
- 팝업 정보 계층

### 3.4 Tier 3: Complex (Level A JRPG)

```
┌─────────────────────────────────────┐
│  [HP] [MP] [버프] [디버프]  [⚙️][📬]│  ← 상태 바
├─────────────────────────────────────┤
│  ┌─────┐                            │
│  │ 적  │       전투 영역             │  ← 메인 영역
│  └─────┘                            │
│  [콤보: 12]        [브레이크: 75%]  │  ← 컨텍스트 정보
├─────────────────────────────────────┤
│  [아군1] [아군2] [아군3] [아군4]    │  ← 파티 상태
├─────────────────────────────────────┤
│  [스킬] [아이템] [방어] [도망]      │  ← 커맨드 메뉴
└─────────────────────────────────────┘
```

**특징:**
- 다중 게이지 시스템
- 복합 상태 표시
- 다단계 메뉴 구조
- 상세 정보 레이어

---

## 4. 컬러 시스템

### 4.1 시맨틱 컬러

```dart
// 공통 시맨틱 컬러 (mg-common-game/lib/ui/colors.dart)
class MGColors {
  // Primary Actions
  static const Color primaryAction = Color(0xFF4CAF50);    // 녹색 - 긍정적 액션
  static const Color secondaryAction = Color(0xFF2196F3); // 파랑 - 보조 액션

  // Status
  static const Color success = Color(0xFF4CAF50);   // 성공
  static const Color warning = Color(0xFFFF9800);   // 경고
  static const Color error = Color(0xFFF44336);     // 오류
  static const Color info = Color(0xFF2196F3);      // 정보

  // Resources
  static const Color gold = Color(0xFFFFD700);      // 골드/코인
  static const Color gem = Color(0xFF9C27B0);       // 젬/다이아
  static const Color energy = Color(0xFF00BCD4);    // 에너지
  static const Color exp = Color(0xFF8BC34A);       // 경험치

  // Rarity
  static const Color common = Color(0xFF9E9E9E);    // 일반
  static const Color uncommon = Color(0xFF4CAF50);  // 고급
  static const Color rare = Color(0xFF2196F3);      // 희귀
  static const Color epic = Color(0xFF9C27B0);      // 영웅
  static const Color legendary = Color(0xFFFF9800); // 전설
  static const Color mythic = Color(0xFFF44336);    // 신화
}
```

### 4.2 게임별 테마 컬러

| 게임 카테고리 | Primary | Secondary | Accent |
|---------------|---------|-----------|--------|
| Year 1 Core | #4CAF50 | #81C784 | #FFD700 |
| Year 2 Core | #2196F3 | #64B5F6 | #FF9800 |
| Level A JRPG | #9C27B0 | #BA68C8 | #E91E63 |
| 인도 Casual | #FF5722 | #FF8A65 | #FFEB3B |
| 남미 Casual | #E91E63 | #F48FB1 | #4CAF50 |
| 동남아 Casual | #00BCD4 | #4DD0E1 | #FF9800 |
| 아프리카 Casual | #FFC107 | #FFD54F | #4CAF50 |

---

## 5. 타이포그래피

### 5.1 폰트 시스템

```dart
// 기본 폰트 스택
const String fontFamily = 'Pretendard, Roboto, sans-serif';

// Level A JRPG 보조 폰트
const String jrpgDisplayFont = 'Black Han Sans, sans-serif';
```

### 5.2 폰트 크기 스케일

| 용도 | 크기 (sp) | 굵기 | 줄 높이 |
|------|-----------|------|---------|
| Display | 32-48 | Bold | 1.2 |
| H1 (제목) | 24-28 | Bold | 1.3 |
| H2 (부제목) | 20-22 | SemiBold | 1.3 |
| H3 (섹션) | 18 | SemiBold | 1.4 |
| Body (본문) | 14-16 | Regular | 1.5 |
| Caption (캡션) | 12 | Regular | 1.4 |
| Button (버튼) | 14-16 | Medium | 1.0 |

### 5.3 다국어 폰트 설정

```dart
// 지역별 폰트 설정
Map<String, String> regionalFonts = {
  'ko': 'Pretendard',         // 한국어
  'ja': 'Noto Sans JP',       // 일본어
  'zh': 'Noto Sans SC',       // 중국어 간체
  'hi': 'Noto Sans Devanagari', // 힌디어
  'th': 'Noto Sans Thai',     // 태국어
  'ar': 'Noto Sans Arabic',   // 아랍어
};
```

---

## 6. 아이콘 시스템

### 6.1 아이콘 크기

| 용도 | 크기 | 터치 영역 |
|------|------|-----------|
| Navigation | 24dp | 48dp |
| Action Bar | 28dp | 44dp |
| Toolbar | 24dp | 40dp |
| List Item | 20dp | - |
| Badge | 16dp | - |

### 6.2 아이콘 스타일 가이드

```
✓ 일관된 선 굵기 (2dp)
✓ 둥근 모서리 (2dp radius)
✓ 채움/외곽선 구분 (활성/비활성)
✓ 최소 요소로 의미 전달
```

### 6.3 공통 아이콘 세트

```
📦 Core Icons (필수)
├── nav_home.svg
├── nav_shop.svg
├── nav_inventory.svg
├── nav_quest.svg
├── nav_social.svg
├── icon_settings.svg
├── icon_notification.svg
├── icon_mail.svg
├── icon_close.svg
├── icon_back.svg
└── icon_info.svg

💰 Resource Icons (자원)
├── res_gold.svg
├── res_gem.svg
├── res_energy.svg
├── res_key.svg
└── res_ticket.svg

⚔️ Game Icons (게임별)
├── game_attack.svg
├── game_defend.svg
├── game_skill.svg
├── game_item.svg
└── game_special.svg
```

---

## 7. 컴포넌트 라이브러리

### 7.1 버튼

```dart
// Primary Button
MGButton.primary(
  text: '시작하기',
  onPressed: () {},
  size: MGButtonSize.large,  // small, medium, large
);

// Secondary Button
MGButton.secondary(
  text: '취소',
  onPressed: () {},
);

// Icon Button
MGButton.icon(
  icon: Icons.settings,
  onPressed: () {},
);
```

**버튼 크기 스펙:**

| 크기 | 높이 | 최소 너비 | 패딩 | 폰트 |
|------|------|-----------|------|------|
| Small | 32dp | 64dp | 12dp | 12sp |
| Medium | 44dp | 88dp | 16dp | 14sp |
| Large | 56dp | 120dp | 20dp | 16sp |

### 7.2 카드

```dart
MGCard(
  child: content,
  elevation: 2,
  borderRadius: 12,
  padding: EdgeInsets.all(16),
);
```

### 7.3 모달/팝업

```dart
MGModal.show(
  context: context,
  title: '알림',
  content: '내용',
  primaryAction: MGButton.primary(...),
  secondaryAction: MGButton.secondary(...),
);
```

### 7.4 프로그레스 바

```dart
// 기본 프로그레스
MGProgress.linear(
  value: 0.75,
  color: MGColors.exp,
);

// HP 바 (Level A)
MGProgress.hp(
  current: 750,
  max: 1000,
  showLabel: true,
);

// 타이머 프로그레스
MGProgress.timer(
  remaining: Duration(seconds: 30),
  total: Duration(minutes: 1),
);
```

---

## 8. 애니메이션 가이드

### 8.1 표준 듀레이션

| 유형 | 듀레이션 | 이징 |
|------|----------|------|
| Micro (피드백) | 100ms | easeOut |
| Short (트랜지션) | 200ms | easeInOut |
| Medium (화면 전환) | 300ms | easeInOut |
| Long (강조) | 500ms | easeInOutCubic |

### 8.2 표준 애니메이션

```dart
// 페이드 인
MGAnimation.fadeIn(
  child: widget,
  duration: Duration(milliseconds: 200),
);

// 슬라이드 업
MGAnimation.slideUp(
  child: widget,
  offset: 50,
);

// 스케일 팝
MGAnimation.pop(
  child: widget,
  scale: 1.1,
);

// 쉐이크 (오류)
MGAnimation.shake(
  child: widget,
);
```

### 8.3 금지 패턴

```
✗ 3초 이상의 로딩 애니메이션
✗ 동시에 3개 이상 애니메이션
✗ 급격한 색상 변화 (접근성)
✗ 무한 반복 애니메이션 (배터리)
```

---

## 9. 레이아웃 그리드

### 9.1 기본 그리드 시스템

```
Portrait (세로):
┌────────────────────────────────────┐
│ 16dp │      Content       │ 16dp  │  ← 좌우 마진
│──────┼────────────────────┼───────│
│      │  8dp column gap    │       │  ← 컬럼 간격
│      │  4 columns         │       │  ← 4컬럼 그리드
└────────────────────────────────────┘

Landscape (가로):
┌──────────────────────────────────────────────────┐
│ 24dp │         Content              │ 24dp      │
│──────┼──────────────────────────────┼───────────│
│      │  12dp column gap             │           │
│      │  6 columns                   │           │
└──────────────────────────────────────────────────┘
```

### 9.2 간격 시스템

```dart
class MGSpacing {
  static const double xxs = 4;   // 미세 간격
  static const double xs = 8;    // 아이콘 내부
  static const double sm = 12;   // 관련 요소
  static const double md = 16;   // 섹션 내부
  static const double lg = 24;   // 섹션 간
  static const double xl = 32;   // 주요 영역
  static const double xxl = 48;  // 화면 섹션
}
```

---

## 10. 반응형 브레이크포인트

### 10.1 화면 크기 분류

```dart
enum ScreenSize {
  compact,   // < 600dp (대부분의 폰)
  medium,    // 600-840dp (작은 태블릿, 폴더블)
  expanded,  // > 840dp (태블릿)
}

ScreenSize getScreenSize(BuildContext context) {
  final width = MediaQuery.of(context).size.width;
  if (width < 600) return ScreenSize.compact;
  if (width < 840) return ScreenSize.medium;
  return ScreenSize.expanded;
}
```

### 10.2 적응형 레이아웃

| 요소 | Compact | Medium | Expanded |
|------|---------|--------|----------|
| 그리드 컬럼 | 4 | 8 | 12 |
| 좌우 마진 | 16dp | 24dp | 32dp |
| 네비게이션 | Bottom Bar | Rail | Drawer |
| 카드 크기 | Full Width | 2-up | 3-up |

---

## 11. 인터랙션 패턴

### 11.1 제스처 표준

| 제스처 | 용도 | 피드백 |
|--------|------|--------|
| 탭 | 선택, 확인 | 리플 + 햅틱 |
| 롱프레스 | 상세보기, 컨텍스트 | 진동 + 팝업 |
| 스와이프 | 삭제, 네비게이션 | 트랜지션 |
| 드래그 | 이동, 정렬 | 오브젝트 이동 |
| 핀치 | 줌 인/아웃 | 스케일 변화 |
| 더블탭 | 빠른 액션 | 하이라이트 |

### 11.2 피드백 우선순위

```
1. 시각적 피드백 (필수): 색상, 크기, 애니메이션
2. 햅틱 피드백 (권장): 성공, 오류, 중요 액션
3. 사운드 피드백 (옵션): 보상, 레벨업, 특수 이벤트
```

---

## 12. 다크 모드

### 12.1 컬러 매핑

```dart
// 라이트 모드
const lightBackground = Color(0xFFFAFAFA);
const lightSurface = Color(0xFFFFFFFF);
const lightText = Color(0xFF212121);

// 다크 모드
const darkBackground = Color(0xFF121212);
const darkSurface = Color(0xFF1E1E1E);
const darkText = Color(0xFFE0E0E0);
```

### 12.2 다크 모드 원칙

```
✓ 순수 검정(#000000) 피하기 → 사용: #121212
✓ 밝기 대비 4.5:1 이상 유지
✓ 포화도 낮은 컬러 사용
✓ 그림자 대신 elevation 표현
✓ 시스템 설정 자동 감지
```

---

## 13. 오프라인 UI

### 13.1 오프라인 상태 표시

```dart
MGOfflineIndicator(
  showBanner: true,
  onRetry: () => checkConnection(),
);
```

### 13.2 오프라인 가용 기능

| 기능 | 오프라인 지원 | 동기화 필요 |
|------|--------------|-------------|
| 싱글플레이 | ✓ | 점수/진행도 |
| 설정 변경 | ✓ | - |
| 인벤토리 조회 | ✓ (캐시) | - |
| 상점 | ✗ | - |
| 멀티플레이 | ✗ | - |
| 가챠 | ✗ | - |

---

## 14. 로딩 패턴

### 14.1 로딩 유형

```dart
// 스켈레톤 로딩 (목록, 카드)
MGSkeleton.list(itemCount: 5);

// 스피너 (액션 대기)
MGSpinner(size: 24);

// 프로그레스 (다운로드, 설치)
MGLoadingProgress(
  value: 0.65,
  label: '리소스 다운로드 중... 65%',
);

// 풀스크린 (초기 로딩)
MGLoadingScreen(
  logo: 'assets/logo.png',
  tips: ['팁 1', '팁 2'],
);
```

### 14.2 로딩 시간 가이드

| 작업 | 최대 시간 | 초과 시 |
|------|----------|---------|
| 화면 전환 | 300ms | 스피너 표시 |
| API 요청 | 3초 | 프로그레스 + 취소 옵션 |
| 초기 로딩 | 10초 | 팁 표시 + 진행률 |
| 다운로드 | - | 남은 시간 표시 |

---

## 15. 오류 처리 UI

### 15.1 오류 유형별 UI

```dart
// 네트워크 오류
MGError.network(
  onRetry: () => retry(),
);

// 서버 오류
MGError.server(
  code: 500,
  message: '서버 점검 중입니다.',
);

// 입력 오류 (인라인)
MGTextField(
  error: '올바른 이메일을 입력하세요.',
);

// 권한 오류
MGError.permission(
  permission: '저장소',
  onSettings: () => openSettings(),
);
```

### 15.2 오류 메시지 원칙

```
✓ 무엇이 잘못되었는지 설명
✓ 어떻게 해결할 수 있는지 안내
✓ 기술적 용어 피하기
✓ 사과보다 해결책 제시

예시:
✗ "Error 503: Service Unavailable"
✓ "서버 연결에 실패했습니다. 잠시 후 다시 시도해주세요."
```

---

## 16. 성능 가이드라인

### 16.1 프레임 목표

| 게임 유형 | 목표 FPS | 최소 FPS |
|-----------|----------|----------|
| 하이퍼캐주얼 | 60 | 30 |
| 캐주얼 | 60 | 30 |
| 미드코어 | 60 | 30 |
| Level A JRPG | 60 | 30 |
| 액션 게임 | 60 | 45 |

### 16.2 UI 성능 최적화

```dart
// ✓ const 위젯 사용
const Text('고정 텍스트');

// ✓ RepaintBoundary로 분리
RepaintBoundary(
  child: AnimatedWidget(),
);

// ✓ 이미지 캐싱
CachedNetworkImage(
  imageUrl: url,
  placeholder: (_, __) => Shimmer(),
);

// ✗ build()에서 객체 생성 피하기
// ✗ 무분별한 setState() 피하기
```

---

## 17. 체크리스트

### 17.1 UI 리뷰 체크리스트

```
[ ] 터치 영역이 최소 44dp인가?
[ ] 텍스트 대비가 4.5:1 이상인가?
[ ] Safe Area가 적용되었는가?
[ ] 다크 모드에서 정상 표시되는가?
[ ] 로딩 상태가 표시되는가?
[ ] 오류 상태가 처리되었는가?
[ ] 오프라인 상태가 처리되었는가?
[ ] 애니메이션이 60fps로 동작하는가?
[ ] 폰트 크기가 스케일에 맞는가?
[ ] 아이콘이 선명한가?
```

### 17.2 접근성 체크리스트

```
[ ] 색맹 사용자를 위한 대안이 있는가?
[ ] 스크린 리더가 모든 요소를 읽는가?
[ ] 동작 감도 설정이 가능한가?
[ ] 자막/텍스트 대안이 있는가?
[ ] 큰 글씨 설정이 적용되는가?
```

---

## 18. 버전 히스토리

| 버전 | 날짜 | 변경 내용 |
|------|------|-----------|
| 1.0.0 | 2025-12-19 | 초기 문서 작성 |

---

*이 문서는 MG-Games UI/UX 표준을 정의합니다. 모든 게임은 이 가이드라인을 준수해야 합니다.*
