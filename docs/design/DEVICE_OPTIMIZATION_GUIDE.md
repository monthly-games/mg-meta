# MG-Games 디바이스 최적화 가이드

> **문서 버전**: 1.0.0
> **최종 수정**: 2025-12-19
> **적용 대상**: 52종 전체 게임 (MG-0001 ~ MG-0052)

---

## 1. 개요

### 1.1 목적
다양한 사양의 기기에서 최적의 성능과 사용자 경험을 제공하기 위한 디바이스 최적화 가이드라인을 정의합니다.

### 1.2 타겟 기기 스펙

| 티어 | RAM | SoC 예시 | 대상 시장 |
|------|-----|----------|-----------|
| **Low-end** | 2-3GB | Snapdragon 400, Helio G35 | 인도, 아프리카, 동남아 |
| **Mid-range** | 4-6GB | Snapdragon 600, Dimensity 700 | 글로벌 메인스트림 |
| **High-end** | 8GB+ | Snapdragon 8 Gen, A-series | 한국, 일본, 북미, EU |

### 1.3 게임 카테고리별 타겟

| 카테고리 | 타겟 티어 | APK 크기 | 메모리 |
|----------|-----------|----------|--------|
| Year 1-2 Core | Mid-range | 80-150MB | 200-400MB |
| Level A JRPG | Mid/High | 150-500MB | 400-800MB |
| Emerging (인도) | Low-end | 50-80MB | 100-200MB |
| Emerging (남미) | Low/Mid | 50-90MB | 100-250MB |
| Emerging (동남아) | Low/Mid | 45-85MB | 100-200MB |
| Emerging (아프리카) | Low-end | 50-75MB | 80-150MB |

---

## 2. APK/IPA 크기 최적화

### 2.1 크기 제한

| 게임 카테고리 | 최대 크기 | 권장 크기 |
|---------------|-----------|-----------|
| 하이퍼캐주얼 | 50MB | 30MB |
| 캐주얼 | 100MB | 60MB |
| 미드코어 | 150MB | 100MB |
| Level A | 500MB | 300MB |
| Emerging 마켓 | 80MB | 50MB |

### 2.2 크기 최적화 전략

#### 2.2.1 이미지 최적화

```dart
// pubspec.yaml
flutter:
  assets:
    - assets/images/

// 이미지 포맷 선택
// PNG: UI 아이콘, 투명 필요
// WebP: 일반 이미지 (30-50% 절약)
// ASTC/ETC2: 3D 텍스처

// 해상도별 에셋
assets/
├── images/
│   ├── 1.0x/  # mdpi (160dpi)
│   ├── 1.5x/  # hdpi (240dpi)
│   ├── 2.0x/  # xhdpi (320dpi)
│   ├── 3.0x/  # xxhdpi (480dpi)
│   └── 4.0x/  # xxxhdpi (640dpi)
```

#### 2.2.2 오디오 최적화

```yaml
# 포맷 및 비트레이트
음악 (BGM):
  포맷: OGG (Android), AAC (iOS)
  비트레이트: 96-128kbps
  샘플레이트: 44.1kHz
  채널: 스테레오

효과음 (SFX):
  포맷: OGG (Android), CAF (iOS)
  비트레이트: 64-96kbps
  샘플레이트: 22.05kHz
  채널: 모노
```

#### 2.2.3 코드 최적화

```yaml
# build.yaml - 트리 쉐이킹
flutter build apk --release --split-per-abi --obfuscate --split-debug-info=./debug-info

# Proguard/R8 (Android)
-keep class com.mggames.** { *; }
-optimizationpasses 5
-allowaccessmodification

# App Thinning (iOS)
# Xcode: Enable Bitcode, Strip Debug Symbols
```

### 2.3 Asset Delivery

#### 2.3.1 Play Asset Delivery (Android)

```kotlin
// 설치 시 다운로드 (install-time)
// 필수 에셋

// 빠른 팔로우 (fast-follow)
// 레벨 1-10 에셋

// 주문형 (on-demand)
// 추가 콘텐츠, 언어팩
```

```dart
// Flutter에서 PAD 사용
import 'package:play_asset_delivery/play_asset_delivery.dart';

Future<void> loadAssetPack(String packName) async {
  final assetPack = await PlayAssetDelivery.fetch(packName);
  if (assetPack.status == AssetPackStatus.completed) {
    // 에셋 사용
  }
}
```

#### 2.3.2 On-Demand Resources (iOS)

```swift
// 리소스 태그 설정
// Xcode > Build Phases > Copy Bundle Resources

// 다운로드
let request = NSBundleResourceRequest(tags: ["level_pack_1"])
request.beginAccessingResources { error in
    if error == nil {
        // 리소스 사용
    }
}
```

---

## 3. 메모리 최적화

### 3.1 메모리 제한

| 티어 | 앱 메모리 | 텍스처 메모리 | 오디오 버퍼 |
|------|-----------|---------------|-------------|
| Low-end | 150MB | 50MB | 10MB |
| Mid-range | 300MB | 100MB | 20MB |
| High-end | 500MB | 200MB | 30MB |

### 3.2 이미지 메모리 관리

```dart
// 이미지 캐시 제한
class ImageCacheConfig {
  // 캐시 크기 제한
  static void configure() {
    PaintingBinding.instance.imageCache.maximumSize = 100;
    PaintingBinding.instance.imageCache.maximumSizeBytes = 50 * 1024 * 1024; // 50MB
  }

  // 수동 캐시 정리
  static void clearCache() {
    PaintingBinding.instance.imageCache.clear();
    PaintingBinding.instance.imageCache.clearLiveImages();
  }
}
```

```dart
// 화면 벗어난 이미지 해제
class DisposableImage extends StatefulWidget {
  final String imageUrl;

  @override
  State<DisposableImage> createState() => _DisposableImageState();
}

class _DisposableImageState extends State<DisposableImage> {
  @override
  void dispose() {
    // 이미지 캐시에서 제거
    imageCache.evict(widget.imageUrl);
    super.dispose();
  }
}
```

### 3.3 오브젝트 풀링

```dart
/// 게임 오브젝트 풀
class ObjectPool<T> {
  final T Function() _factory;
  final void Function(T) _reset;
  final List<T> _pool = [];

  ObjectPool({
    required T Function() factory,
    required void Function(T) reset,
    int initialSize = 10,
  }) : _factory = factory, _reset = reset {
    for (var i = 0; i < initialSize; i++) {
      _pool.add(_factory());
    }
  }

  T acquire() {
    if (_pool.isNotEmpty) {
      return _pool.removeLast();
    }
    return _factory();
  }

  void release(T object) {
    _reset(object);
    _pool.add(object);
  }
}

// 사용 예시: 파티클 풀
final particlePool = ObjectPool<Particle>(
  factory: () => Particle(),
  reset: (p) => p.reset(),
  initialSize: 100,
);
```

### 3.4 메모리 모니터링

```dart
// 메모리 사용량 체크
import 'dart:developer' as developer;

class MemoryMonitor {
  static void logMemoryUsage() {
    final info = developer.Service.getInfo();
    print('Memory: ${ProcessInfo.currentRss ~/ 1024 ~/ 1024}MB');
  }

  static void checkMemoryPressure(VoidCallback onLowMemory) {
    // 저사양 기기에서 메모리 부족 시 호출
    WidgetsBinding.instance.addObserver(
      LifecycleEventHandler(
        onMemoryPressure: onLowMemory,
      ),
    );
  }
}
```

---

## 4. CPU/GPU 최적화

### 4.1 프레임 목표

| 게임 유형 | 목표 FPS | 최소 FPS | 프레임 버짓 |
|-----------|----------|----------|-------------|
| 캐주얼/퍼즐 | 60 | 30 | 16.67ms |
| 액션/러너 | 60 | 45 | 16.67ms |
| 전투 RPG | 60 | 30 | 16.67ms |
| 복잡한 UI | 60 | 30 | 16.67ms |

### 4.2 렌더링 최적화

#### 4.2.1 위젯 리빌드 최소화

```dart
// ❌ 잘못된 예시: 전체 리빌드
class BadExample extends StatefulWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('점수: $score'),  // 점수 변경 시 전체 리빌드
        ExpensiveWidget(),
      ],
    );
  }
}

// ✅ 올바른 예시: 부분 리빌드
class GoodExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 점수만 리빌드
        ValueListenableBuilder<int>(
          valueListenable: scoreNotifier,
          builder: (_, score, __) => Text('점수: $score'),
        ),
        const ExpensiveWidget(),  // const로 리빌드 방지
      ],
    );
  }
}
```

#### 4.2.2 RepaintBoundary 사용

```dart
// 복잡한 애니메이션 영역 분리
RepaintBoundary(
  child: AnimatedGameCanvas(),
)

// 정적 UI 분리
RepaintBoundary(
  child: StaticBackground(),
)
```

#### 4.2.3 이미지 렌더링 최적화

```dart
// 이미지 캐싱
Image.asset(
  'assets/image.png',
  cacheWidth: 200,
  cacheHeight: 200,
  filterQuality: FilterQuality.medium,
)

// 대용량 이미지 디코딩
Future<ui.Image> loadOptimizedImage(String path) async {
  final data = await rootBundle.load(path);
  final codec = await ui.instantiateImageCodec(
    data.buffer.asUint8List(),
    targetWidth: 512,  // 최대 크기 제한
    targetHeight: 512,
  );
  final frame = await codec.getNextFrame();
  return frame.image;
}
```

### 4.3 애니메이션 최적화

```dart
// Ticker 기반 애니메이션 (효율적)
class OptimizedAnimation extends StatefulWidget {
  @override
  State<OptimizedAnimation> createState() => _OptimizedAnimationState();
}

class _OptimizedAnimationState extends State<OptimizedAnimation>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,  // VSync로 프레임 동기화
      duration: Duration(milliseconds: 300),
    );
  }
}

// 비활성화 시 애니메이션 정지
@override
void didChangeAppLifecycleState(AppLifecycleState state) {
  if (state == AppLifecycleState.paused) {
    _controller.stop();
  } else if (state == AppLifecycleState.resumed) {
    _controller.forward();
  }
}
```

---

## 5. 배터리 최적화

### 5.1 배터리 소모 요인

| 요인 | 영향도 | 최적화 방법 |
|------|--------|-------------|
| GPU 사용 | 높음 | 프레임 제한, 셰이더 최적화 |
| 네트워크 | 높음 | 배치 요청, 캐싱 |
| 위치 서비스 | 높음 | 필요 시에만 사용 |
| 진동 | 중간 | 빈도 제한 |
| 화면 밝기 | 중간 | 시스템 설정 존중 |

### 5.2 백그라운드 최적화

```dart
// 백그라운드 진입 시
@override
void didChangeAppLifecycleState(AppLifecycleState state) {
  switch (state) {
    case AppLifecycleState.paused:
      // 애니메이션 정지
      _stopAnimations();
      // 네트워크 요청 일시 중단
      _pauseNetworkRequests();
      // 오디오 정지
      _pauseAudio();
      break;

    case AppLifecycleState.resumed:
      // 재개
      _resumeGame();
      break;

    case AppLifecycleState.inactive:
      // 준비 상태
      break;

    case AppLifecycleState.detached:
      // 정리
      _cleanup();
      break;
  }
}
```

### 5.3 프레임 제한

```dart
// 배터리 절약 모드
class BatterySaverMode {
  bool enabled = false;

  int get targetFps => enabled ? 30 : 60;

  void apply() {
    if (enabled) {
      // 애니메이션 속도 절반
      // 파티클 수 감소
      // 백그라운드 요소 단순화
    }
  }
}
```

---

## 6. 네트워크 최적화

### 6.1 데이터 사용량 제한

| 연결 유형 | 전략 |
|-----------|------|
| WiFi | 전체 기능, 고품질 에셋 |
| 4G/5G | 표준 기능, 중간 품질 |
| 3G/느린 연결 | 최소 기능, 저품질 에셋 |
| 오프라인 | 캐시된 콘텐츠만 |

### 6.2 요청 최적화

```dart
// 요청 배칭
class RequestBatcher {
  final List<Request> _pending = [];
  Timer? _batchTimer;

  void add(Request request) {
    _pending.add(request);
    _scheduleBatch();
  }

  void _scheduleBatch() {
    _batchTimer ??= Timer(Duration(milliseconds: 100), _sendBatch);
  }

  Future<void> _sendBatch() async {
    final batch = List.of(_pending);
    _pending.clear();
    _batchTimer = null;

    // 일괄 전송
    await api.sendBatch(batch);
  }
}
```

### 6.3 오프라인 지원

```dart
// 오프라인 캐시
class OfflineCache {
  static const String cacheBox = 'offline_cache';

  Future<void> cacheData(String key, dynamic data) async {
    final box = await Hive.openBox(cacheBox);
    await box.put(key, {
      'data': data,
      'timestamp': DateTime.now().millisecondsSinceEpoch,
    });
  }

  Future<T?> getCachedData<T>(String key, Duration maxAge) async {
    final box = await Hive.openBox(cacheBox);
    final cached = box.get(key);

    if (cached == null) return null;

    final age = DateTime.now().millisecondsSinceEpoch - cached['timestamp'];
    if (age > maxAge.inMilliseconds) return null;

    return cached['data'] as T;
  }
}
```

---

## 7. 기기별 적응형 품질

### 7.1 기기 감지

```dart
class DeviceCapability {
  final int ramMB;
  final String gpuVendor;
  final int cpuCores;
  final double screenDensity;

  DeviceTier get tier {
    if (ramMB < 3000) return DeviceTier.low;
    if (ramMB < 6000) return DeviceTier.mid;
    return DeviceTier.high;
  }

  static Future<DeviceCapability> detect() async {
    final deviceInfo = DeviceInfoPlugin();
    if (Platform.isAndroid) {
      final android = await deviceInfo.androidInfo;
      return DeviceCapability(
        ramMB: android.totalMemory ~/ 1024 ~/ 1024,
        gpuVendor: android.hardware,
        cpuCores: Platform.numberOfProcessors,
        screenDensity: WidgetsBinding.instance.window.devicePixelRatio,
      );
    }
    // iOS 처리
    return DeviceCapability.defaultValues();
  }
}
```

### 7.2 품질 설정 자동 적용

```dart
class QualitySettings {
  late final DeviceTier tier;

  // 그래픽 품질
  int get particleCount => switch (tier) {
    DeviceTier.low => 20,
    DeviceTier.mid => 50,
    DeviceTier.high => 100,
  };

  double get textureScale => switch (tier) {
    DeviceTier.low => 0.5,
    DeviceTier.mid => 0.75,
    DeviceTier.high => 1.0,
  };

  bool get shadowEnabled => tier != DeviceTier.low;
  bool get postProcessing => tier == DeviceTier.high;

  int get targetFps => switch (tier) {
    DeviceTier.low => 30,
    DeviceTier.mid => 60,
    DeviceTier.high => 60,
  };
}
```

### 7.3 에셋 품질 분기

```dart
// 기기 티어별 에셋 경로
String getAssetPath(String baseName, DeviceTier tier) {
  final quality = switch (tier) {
    DeviceTier.low => 'low',
    DeviceTier.mid => 'mid',
    DeviceTier.high => 'high',
  };
  return 'assets/$quality/$baseName';
}

// 사용
Image.asset(getAssetPath('character.png', device.tier));
```

---

## 8. Emerging 마켓 최적화

### 8.1 인도 시장 (MG-0037~0040)

```dart
class IndiaOptimization {
  // APK 크기: 50-80MB 목표
  static const int maxApkSizeMB = 80;

  // 저사양 기기 지원
  static const int minRamMB = 2000;

  // 네트워크 최적화
  static const bool offlineModeRequired = true;

  // 데이터 절약
  static const bool dataSaverDefault = true;
}
```

### 8.2 아프리카 시장 (MG-0049~0052)

```dart
class AfricaOptimization {
  // APK 크기: 50-75MB 목표 (최소)
  static const int maxApkSizeMB = 75;

  // 2G/3G 네트워크 대응
  static const int connectionTimeoutSec = 30;

  // 오프라인 우선
  static const bool offlineFirstDesign = true;

  // 배터리 최적화 (전력 불안정)
  static const bool batterySaverDefault = true;
}
```

### 8.3 동남아 시장 (MG-0045~0048)

```dart
class SEAOptimization {
  // APK 크기: 45-85MB 목표
  static const int maxApkSizeMB = 85;

  // 열 관리 (고온 환경)
  static const bool thermalThrottling = true;

  // 다양한 기기 지원
  static const double minAndroidVersion = 5.0;
}
```

---

## 9. 성능 모니터링

### 9.1 프레임 모니터링

```dart
// 프레임 드롭 감지
class FrameMonitor {
  final List<int> _frameTimings = [];
  int _droppedFrames = 0;

  void recordFrame(Duration timing) {
    _frameTimings.add(timing.inMilliseconds);

    // 16.67ms(60fps) 초과 시 드롭
    if (timing.inMilliseconds > 17) {
      _droppedFrames++;
    }

    // 최근 60프레임만 유지
    if (_frameTimings.length > 60) {
      _frameTimings.removeAt(0);
    }
  }

  double get averageFps {
    if (_frameTimings.isEmpty) return 60;
    final avgMs = _frameTimings.reduce((a, b) => a + b) / _frameTimings.length;
    return 1000 / avgMs;
  }

  int get droppedFrameCount => _droppedFrames;
}
```

### 9.2 성능 리포트

```dart
// Firebase Performance 연동
class PerformanceReporter {
  void reportGameSession({
    required Duration duration,
    required double averageFps,
    required int droppedFrames,
    required int peakMemoryMB,
  }) async {
    await FirebasePerformance.instance
        .newTrace('game_session')
        .putMetric('duration_sec', duration.inSeconds)
        .putMetric('avg_fps', averageFps.round())
        .putMetric('dropped_frames', droppedFrames)
        .putMetric('peak_memory_mb', peakMemoryMB)
        .stop();
  }
}
```

---

## 10. 체크리스트

### 10.1 출시 전 성능 체크리스트

```
APK/IPA 크기:
[ ] 목표 크기 이하인가?
[ ] 불필요한 에셋이 제거되었는가?
[ ] Asset Delivery가 설정되었는가?

메모리:
[ ] Low-end 기기에서 150MB 이하인가?
[ ] 메모리 누수가 없는가?
[ ] 이미지 캐시가 제한되었는가?

성능:
[ ] 30fps 이상 유지되는가?
[ ] 프레임 드롭이 10% 이하인가?
[ ] 발열이 심하지 않은가?

배터리:
[ ] 1시간 플레이 시 15% 이하 소모인가?
[ ] 백그라운드에서 배터리 소모가 없는가?

네트워크:
[ ] 오프라인 모드가 동작하는가?
[ ] 데이터 사용량이 적절한가?
[ ] 느린 네트워크에서 타임아웃 처리가 되는가?
```

### 10.2 테스트 기기 목록

```
Low-end (필수):
- Samsung Galaxy A03
- Redmi 9A
- Tecno Spark 7

Mid-range (필수):
- Samsung Galaxy A53
- Redmi Note 12
- iPhone SE (2022)

High-end (권장):
- Samsung Galaxy S24
- iPhone 15 Pro
- Pixel 8 Pro
```

---

## 11. 버전 히스토리

| 버전 | 날짜 | 변경 내용 |
|------|------|-----------|
| 1.0.0 | 2025-12-19 | 초기 문서 작성 |

---

*이 문서는 MG-Games 52종 게임의 디바이스 최적화 기준을 정의합니다.*
