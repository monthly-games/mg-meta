# MG-Games 접근성(Accessibility) 가이드

> **문서 버전**: 1.0.0
> **최종 수정**: 2025-12-19
> **적용 대상**: 52종 전체 게임 (MG-0001 ~ MG-0052)

---

## 1. 개요

### 1.1 목적
모든 사용자가 게임을 즐길 수 있도록 접근성 기능을 정의합니다. 시각, 청각, 운동 능력에 제한이 있는 사용자를 포함합니다.

### 1.2 접근성 원칙 (POUR)

| 원칙 | 설명 | 적용 예시 |
|------|------|-----------|
| **P**erceivable (인지 가능) | 정보를 인식할 수 있어야 함 | 색맹 모드, 자막 |
| **O**perable (조작 가능) | 조작할 수 있어야 함 | 터치 영역 조절, 조작 단순화 |
| **U**nderstandable (이해 가능) | 이해할 수 있어야 함 | 명확한 피드백, 가이드 |
| **R**obust (견고함) | 다양한 기기에서 동작 | 스크린 리더 지원 |

### 1.3 적용 수준

| 수준 | 설명 | 대상 게임 |
|------|------|-----------|
| **A** (필수) | 최소 접근성 요구사항 | 전체 52종 |
| **AA** (권장) | 향상된 접근성 | Level A (0025~0036) |
| **AAA** (최고) | 최고 수준 접근성 | 선택적 적용 |

---

## 2. 시각 접근성

### 2.1 색맹 모드

#### 2.1.1 색맹 유형

| 유형 | 비율 | 영향받는 색상 |
|------|------|---------------|
| 적록 색맹 (Deuteranopia) | 6% (남성) | 빨강/녹색 구분 어려움 |
| 적색맹 (Protanopia) | 1% | 빨강 인식 어려움 |
| 청황 색맹 (Tritanopia) | 0.01% | 파랑/노랑 구분 어려움 |

#### 2.1.2 색맹 대응 팔레트

```dart
/// 일반 컬러 vs 색맹 대응 컬러
class MGAccessibleColors {
  // 성공 (녹색 → 청록)
  static const Color successNormal = Color(0xFF4CAF50);   // 녹색
  static const Color successColorblind = Color(0xFF00ACC1); // 청록

  // 오류 (빨강 → 주황/패턴)
  static const Color errorNormal = Color(0xFFF44336);     // 빨강
  static const Color errorColorblind = Color(0xFFFF6D00);  // 주황

  // 희귀도 (색상 + 패턴/아이콘)
  static const Map<String, RarityStyle> rarityStyles = {
    'common': RarityStyle(color: Color(0xFF9E9E9E), icon: '○'),
    'uncommon': RarityStyle(color: Color(0xFF00BCD4), icon: '◇'),
    'rare': RarityStyle(color: Color(0xFF2196F3), icon: '☆'),
    'epic': RarityStyle(color: Color(0xFF9C27B0), icon: '◆'),
    'legendary': RarityStyle(color: Color(0xFFFF9800), icon: '★'),
  };
}
```

#### 2.1.3 색상 외 보조 표시

```
원칙: 색상만으로 정보를 전달하지 않음

✗ 잘못된 예시:
  - 빨간 버튼 = 위험, 녹색 버튼 = 안전 (색상만 사용)

✓ 올바른 예시:
  - 빨간 버튼 + "삭제" 텍스트 + ⚠️ 아이콘
  - 녹색 버튼 + "확인" 텍스트 + ✓ 아이콘
```

#### 2.1.4 색맹 모드 적용

```dart
// 설정에서 색맹 모드 활성화
class AccessibilitySettings {
  bool colorBlindMode = false;
  ColorBlindType colorBlindType = ColorBlindType.deuteranopia;
}

// 테마에 적용
ThemeData getTheme(AccessibilitySettings settings) {
  if (settings.colorBlindMode) {
    return ThemeData(
      colorScheme: ColorScheme.fromSeed(
        seedColor: MGAccessibleColors.successColorblind,
        error: MGAccessibleColors.errorColorblind,
      ),
    );
  }
  return defaultTheme;
}
```

### 2.2 고대비 모드

#### 2.2.1 대비 비율 요구사항

| 요소 | 최소 대비 (A) | 권장 대비 (AA) | 최고 대비 (AAA) |
|------|---------------|----------------|-----------------|
| 일반 텍스트 | 4.5:1 | 4.5:1 | 7:1 |
| 큰 텍스트 (18sp+) | 3:1 | 3:1 | 4.5:1 |
| 아이콘/그래픽 | 3:1 | 3:1 | - |
| 버튼/입력 필드 | 3:1 | 3:1 | - |

#### 2.2.2 고대비 팔레트

```dart
class MGHighContrastColors {
  // 배경
  static const Color background = Color(0xFF000000);

  // 텍스트
  static const Color textPrimary = Color(0xFFFFFFFF);
  static const Color textSecondary = Color(0xFFFFFF00);

  // UI 요소
  static const Color buttonPrimary = Color(0xFFFFFFFF);
  static const Color buttonSecondary = Color(0xFF00FFFF);
  static const Color border = Color(0xFFFFFFFF);

  // 상태
  static const Color success = Color(0xFF00FF00);
  static const Color error = Color(0xFFFF0000);
  static const Color warning = Color(0xFFFFFF00);
}
```

### 2.3 텍스트 크기 조절

#### 2.3.1 크기 스케일

```dart
// 시스템 설정 연동
double getScaledFontSize(BuildContext context, double baseSize) {
  final textScaleFactor = MediaQuery.of(context).textScaleFactor;
  return baseSize * textScaleFactor;
}

// 사용자 정의 스케일
enum TextScaleOption {
  small(0.85),
  normal(1.0),
  large(1.15),
  extraLarge(1.3),
  huge(1.5);

  final double factor;
  const TextScaleOption(this.factor);
}
```

#### 2.3.2 최소 텍스트 크기

| 용도 | 최소 크기 | 권장 크기 |
|------|-----------|-----------|
| 본문 | 12sp | 14sp |
| 버튼 | 12sp | 14sp |
| 캡션 | 10sp | 12sp |
| HUD | 11sp | 13sp |

### 2.4 화면 읽기 (스크린 리더)

#### 2.4.1 Semantics 적용

```dart
// 모든 인터랙티브 요소에 Semantics 적용
Semantics(
  label: '시작 버튼, 게임을 시작합니다',
  button: true,
  child: StartButton(),
)

// 이미지에 설명 추가
Semantics(
  label: '골드 아이콘, 현재 보유량 1,234',
  child: GoldIcon(),
)

// 상태 변화 알림
Semantics(
  liveRegion: true,  // 변경 시 자동 읽기
  child: ScoreDisplay(),
)
```

#### 2.4.2 탐색 순서

```dart
// 논리적 탐색 순서 지정
FocusTraversalGroup(
  policy: OrderedTraversalPolicy(
    order: [
      NumericFocusOrder(1),  // 점수
      NumericFocusOrder(2),  // 설정
      NumericFocusOrder(3),  // 게임 영역
      NumericFocusOrder(4),  // 시작 버튼
    ],
  ),
  child: GameScreen(),
)
```

### 2.5 화면 확대

```dart
// 확대 제스처 지원
InteractiveViewer(
  minScale: 1.0,
  maxScale: 3.0,
  child: GameCanvas(),
)

// 확대 시 UI 재배치
class ZoomableGame extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        final isZoomed = constraints.maxWidth > 1200;
        return isZoomed
            ? ZoomedLayout()
            : NormalLayout();
      },
    );
  }
}
```

---

## 3. 청각 접근성

### 3.1 자막 (Subtitles)

#### 3.1.1 자막 스타일

```dart
class SubtitleStyle {
  static const TextStyle defaultStyle = TextStyle(
    fontSize: 16,
    fontWeight: FontWeight.w500,
    color: Colors.white,
    backgroundColor: Color(0xCC000000),  // 80% 불투명
    height: 1.5,
  );

  // 화자 구분
  static TextStyle getSpeakerStyle(String speaker) {
    final color = _getSpeakerColor(speaker);
    return defaultStyle.copyWith(color: color);
  }
}
```

#### 3.1.2 자막 위치

```
┌─────────────────────────────────────┐
│                                     │
│            게임 영역                 │
│                                     │
│                                     │
├─────────────────────────────────────┤
│  [화자명]                            │
│  "대사 내용입니다."                   │  ← 하단 Safe Area 위
└─────────────────────────────────────┘
```

### 3.2 시각적 효과음 표시

#### 3.2.1 효과음 아이콘

```
| 효과음 | 시각적 표시 |
|--------|-------------|
| 폭발   | 💥 + 화면 흔들림 |
| 경고   | ⚠️ + 테두리 빨간색 |
| 보상   | ✨ + 파티클 |
| 타격   | 💢 + 히트 이펙트 |
| 배경음악 | 🎵 + 파형 표시 |
```

#### 3.2.2 시각적 피드백 코드

```dart
class VisualSoundFeedback extends StatelessWidget {
  final SoundEvent event;

  @override
  Widget build(BuildContext context) {
    return AnimatedContainer(
      duration: Duration(milliseconds: 200),
      decoration: BoxDecoration(
        border: event.isAlert
            ? Border.all(color: Colors.red, width: 3)
            : null,
      ),
      child: Stack(
        children: [
          GameContent(),
          if (event.isPlaying)
            Positioned(
              top: 16,
              right: 16,
              child: SoundIndicator(icon: event.icon),
            ),
        ],
      ),
    );
  }
}
```

### 3.3 진동 피드백

```dart
class HapticFeedbackSettings {
  bool enabled = true;
  double intensity = 1.0;  // 0.0 ~ 1.0

  void feedback(HapticType type) {
    if (!enabled) return;

    switch (type) {
      case HapticType.light:
        HapticFeedback.lightImpact();
        break;
      case HapticType.medium:
        HapticFeedback.mediumImpact();
        break;
      case HapticType.heavy:
        HapticFeedback.heavyImpact();
        break;
      case HapticType.success:
        HapticFeedback.heavyImpact();
        await Future.delayed(Duration(milliseconds: 100));
        HapticFeedback.lightImpact();
        break;
      case HapticType.error:
        HapticFeedback.vibrate();
        break;
    }
  }
}
```

---

## 4. 운동 접근성

### 4.1 터치 영역 조절

#### 4.1.1 최소 터치 영역

| 수준 | 최소 크기 | 용도 |
|------|-----------|------|
| 기본 | 44dp × 44dp | 일반 버튼 |
| 확대 | 56dp × 56dp | 접근성 모드 |
| 최대 | 72dp × 72dp | 심한 운동 장애 |

#### 4.1.2 터치 영역 확대 옵션

```dart
class TouchSettings {
  TouchTargetSize size = TouchTargetSize.normal;

  double getMinimumSize() {
    return switch (size) {
      TouchTargetSize.normal => 44,
      TouchTargetSize.large => 56,
      TouchTargetSize.extraLarge => 72,
    };
  }
}

// 적용
Widget buildButton(TouchSettings settings) {
  return SizedBox(
    width: max(buttonWidth, settings.getMinimumSize()),
    height: max(buttonHeight, settings.getMinimumSize()),
    child: Button(),
  );
}
```

### 4.2 조작 단순화

#### 4.2.1 한손 모드

```
기본 레이아웃:              한손 모드:
┌─────────────────┐        ┌─────────────────┐
│ [◀][▶]   [A][B] │        │                 │
│                 │        │                 │
│                 │   →    │                 │
│                 │        │    [◀][▶]       │
│                 │        │    [A] [B]      │
└─────────────────┘        └─────────────────┘
```

#### 4.2.2 자동 조작 옵션

```dart
class AutoAssistSettings {
  bool autoAim = false;       // 자동 조준
  bool autoMove = false;      // 자동 이동
  bool autoCombo = false;     // 자동 콤보
  bool holdToRepeat = true;   // 길게 누르면 반복
  double holdDelay = 500;     // 홀드 인식 시간 (ms)
}
```

### 4.3 타이밍 조절

#### 4.3.1 QTE/타이밍 완화

```dart
class TimingSettings {
  TimingDifficulty difficulty = TimingDifficulty.normal;

  Duration getQteWindow() {
    return switch (difficulty) {
      TimingDifficulty.easy => Duration(milliseconds: 2000),
      TimingDifficulty.normal => Duration(milliseconds: 1000),
      TimingDifficulty.hard => Duration(milliseconds: 500),
    };
  }

  double getTimingTolerance() {
    return switch (difficulty) {
      TimingDifficulty.easy => 0.3,    // ±30%
      TimingDifficulty.normal => 0.15, // ±15%
      TimingDifficulty.hard => 0.05,   // ±5%
    };
  }
}
```

#### 4.3.2 일시 정지 허용

```dart
// 언제든 일시 정지 가능
class PauseHandler {
  bool canPause = true;  // 항상 true로 유지

  void pause() {
    if (canPause) {
      gameState.pause();
      showPauseMenu();
    }
  }
}
```

### 4.4 제스처 대안

#### 4.4.1 복잡한 제스처 대안

| 복잡한 제스처 | 대안 1 | 대안 2 |
|---------------|--------|--------|
| 핀치 줌 | +/- 버튼 | 슬라이더 |
| 멀티터치 | 순차 입력 | 토글 모드 |
| 스와이프 | 화살표 버튼 | 탭으로 방향 |
| 드래그 앤 드롭 | 선택 후 배치 | 키보드 방향키 |

#### 4.4.2 외부 입력 지원

```dart
// 게임패드 지원
class GamepadSupport {
  bool enabled = true;

  Map<GamepadButton, GameAction> buttonMapping = {
    GamepadButton.a: GameAction.confirm,
    GamepadButton.b: GameAction.cancel,
    GamepadButton.x: GameAction.skill1,
    GamepadButton.y: GameAction.skill2,
    GamepadButton.lb: GameAction.prevTab,
    GamepadButton.rb: GameAction.nextTab,
  };
}

// 키보드 지원 (PC/Chromebook)
class KeyboardSupport {
  bool enabled = true;

  Map<LogicalKeyboardKey, GameAction> keyMapping = {
    LogicalKeyboardKey.space: GameAction.confirm,
    LogicalKeyboardKey.escape: GameAction.cancel,
    LogicalKeyboardKey.arrowUp: GameAction.moveUp,
    LogicalKeyboardKey.arrowDown: GameAction.moveDown,
    // ...
  };
}
```

---

## 5. 인지 접근성

### 5.1 튜토리얼 옵션

```dart
class TutorialSettings {
  TutorialLevel level = TutorialLevel.normal;
  bool showHints = true;
  bool repeatTutorial = true;  // 언제든 다시 볼 수 있음
}

enum TutorialLevel {
  minimal,   // 최소 설명
  normal,    // 기본 설명
  detailed,  // 상세 설명
  guided,    // 단계별 안내
}
```

### 5.2 UI 단순화 옵션

```dart
class UISimplificationSettings {
  bool showAllInfo = true;      // 모든 정보 표시
  bool hideAnimations = false;  // 애니메이션 숨기기
  bool largerText = false;      // 큰 텍스트
  bool highlightInteractive = true;  // 상호작용 요소 강조
}
```

### 5.3 읽기 지원

```dart
// 텍스트 읽어주기 (TTS)
class TextToSpeechSupport {
  bool enabled = false;
  double speed = 1.0;
  String language = 'ko';

  void speak(String text) async {
    if (!enabled) return;
    await tts.speak(text);
  }
}

// 주요 UI에 적용
GestureDetector(
  onLongPress: () => tts.speak('시작 버튼입니다. 탭하면 게임이 시작됩니다.'),
  child: StartButton(),
)
```

---

## 6. 게임별 접근성 기능

### 6.1 Level A (수준 A) 필수 기능

**모든 52종 게임에 적용:**

```dart
class LevelAAccessibility {
  // 필수 기능
  static const List<AccessibilityFeature> required = [
    AccessibilityFeature.colorBlindMode,
    AccessibilityFeature.subtitles,
    AccessibilityFeature.touchTargetSize44,
    AccessibilityFeature.pauseAnytime,
    AccessibilityFeature.textScale,
  ];
}
```

### 6.2 Level AA (수준 AA) 권장 기능

**Level A JRPG (0025~0036)에 적용:**

```dart
class LevelAAAccessibility {
  static const List<AccessibilityFeature> recommended = [
    // Level A 포함
    ...LevelAAccessibility.required,

    // 추가 기능
    AccessibilityFeature.highContrastMode,
    AccessibilityFeature.hapticFeedback,
    AccessibilityFeature.autoAssist,
    AccessibilityFeature.timingAdjust,
    AccessibilityFeature.screenReaderFull,
  ];
}
```

### 6.3 게임 유형별 특수 접근성

| 게임 유형 | 특수 접근성 기능 |
|-----------|------------------|
| 퍼즐 (매치-3) | 색맹 모드 필수, 패턴 구분 |
| 리듬 게임 | 타이밍 조절, 시각 피드백 |
| 액션 게임 | 자동 조준, 난이도 조절 |
| RPG 전투 | 자동 전투, 턴 시간 무제한 |
| 러너 게임 | 반응 시간 조절, 장애물 예고 |

---

## 7. 접근성 설정 UI

### 7.1 설정 화면 구조

```
설정 > 접근성
├── 시각
│   ├── 색맹 모드 [ON/OFF]
│   │   └── 유형 선택 [적록/적색/청황]
│   ├── 고대비 모드 [ON/OFF]
│   ├── 텍스트 크기 [작게/보통/크게/매우 크게]
│   └── 화면 읽기 [ON/OFF]
├── 청각
│   ├── 자막 [ON/OFF]
│   │   ├── 크기 [작게/보통/크게]
│   │   └── 배경 [투명/반투명/불투명]
│   ├── 시각적 효과음 [ON/OFF]
│   └── 진동 피드백 [ON/OFF]
├── 조작
│   ├── 터치 영역 [보통/크게/매우 크게]
│   ├── 한손 모드 [OFF/왼손/오른손]
│   ├── 자동 조작 [ON/OFF]
│   └── 타이밍 조절 [쉬움/보통/어려움]
└── 기타
    ├── 튜토리얼 레벨 [최소/기본/상세]
    └── UI 단순화 [ON/OFF]
```

### 7.2 설정 화면 코드

```dart
class AccessibilitySettingsScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('접근성 설정')),
      body: ListView(
        children: [
          // 시각 섹션
          SettingsSection(
            title: '시각',
            children: [
              SwitchTile(
                title: '색맹 모드',
                subtitle: '색상 구분이 어려운 경우 활성화',
                value: settings.colorBlindMode,
                onChanged: (v) => settings.colorBlindMode = v,
              ),
              if (settings.colorBlindMode)
                DropdownTile(
                  title: '색맹 유형',
                  value: settings.colorBlindType,
                  options: ColorBlindType.values,
                ),
              // ...
            ],
          ),
          // 청각 섹션
          // 조작 섹션
          // 기타 섹션
        ],
      ),
    );
  }
}
```

---

## 8. 테스트 체크리스트

### 8.1 시각 접근성 테스트

```
[ ] 색맹 시뮬레이터로 UI 확인 (Coblis, Color Oracle)
[ ] 대비 검사기로 텍스트 대비 확인 (4.5:1 이상)
[ ] 스크린 리더(TalkBack/VoiceOver)로 탐색 가능
[ ] 200% 확대 시 UI 깨지지 않음
[ ] 색상 외 보조 표시 확인 (아이콘, 패턴)
```

### 8.2 청각 접근성 테스트

```
[ ] 자막 표시 확인
[ ] 음소거 상태에서 모든 정보 인지 가능
[ ] 시각적 효과음 표시 확인
[ ] 진동 피드백 동작 확인
```

### 8.3 운동 접근성 테스트

```
[ ] 44dp 이상 터치 영역 확인
[ ] 한손으로 모든 기능 조작 가능
[ ] 일시 정지 언제든 가능
[ ] 타이밍 조절 옵션 동작 확인
[ ] 외부 입력 장치 지원 확인
```

---

## 9. 관련 리소스

### 9.1 도구

- [Coblis 색맹 시뮬레이터](https://www.color-blindness.com/coblis-color-blindness-simulator/)
- [WebAIM 대비 검사기](https://webaim.org/resources/contrastchecker/)
- [Flutter Accessibility](https://docs.flutter.dev/accessibility-and-localization/accessibility)

### 9.2 가이드라인

- [WCAG 2.1](https://www.w3.org/WAI/WCAG21/quickref/)
- [Apple Accessibility](https://developer.apple.com/accessibility/)
- [Android Accessibility](https://developer.android.com/guide/topics/ui/accessibility)

---

## 10. 버전 히스토리

| 버전 | 날짜 | 변경 내용 |
|------|------|-----------|
| 1.0.0 | 2025-12-19 | 초기 문서 작성 |

---

*이 문서는 MG-Games 52종 게임의 접근성 기능을 정의합니다.*
