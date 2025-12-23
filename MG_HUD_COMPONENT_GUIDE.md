# MG Games HUD Component Usage Guide

> **Complete reference for building game HUDs using mg_common_game components**

## 📚 Table of Contents

1. [Quick Start](#quick-start)
2. [Core Components](#core-components)
3. [HUD Structure Pattern](#hud-structure-pattern)
4. [Component Reference](#component-reference)
5. [Regional Theming](#regional-theming)
6. [Examples by Game Type](#examples-by-game-type)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)

---

## Quick Start

### 1. Add mg_common_game Dependency

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter

  # MG Common Game UI Components
  mg_common_game:
    path: ../../mg-common-game
```

### 2. Import Required Components

```dart
import 'package:mg_common_game/core/ui/theme/mg_colors.dart';
import 'package:mg_common_game/core/ui/layout/mg_spacing.dart';
import 'package:mg_common_game/core/ui/typography/mg_text_styles.dart';
import 'package:mg_common_game/core/ui/widgets/buttons/mg_button.dart';
import 'package:mg_common_game/core/ui/widgets/progress/mg_progress.dart';
```

### 3. Create Your HUD File

**Location**: `lib/ui/hud/mg_[game_name]_hud.dart`

```dart
import 'package:flutter/material.dart';
import 'package:mg_common_game/core/ui/mg_ui.dart';

class MGYourGameHud extends StatelessWidget {
  final int score;
  final VoidCallback? onPause;

  const MGYourGameHud({
    super.key,
    required this.score,
    this.onPause,
  });

  @override
  Widget build(BuildContext context) {
    return SafeArea(
      child: Padding(
        padding: EdgeInsets.all(MGSpacing.md),
        child: Column(
          children: [
            // Top row
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                _buildScore(),
                if (onPause != null) _buildPauseButton(),
              ],
            ),
            const Spacer(),
            // Bottom actions
            _buildActions(),
          ],
        ),
      ),
    );
  }

  Widget _buildScore() {
    return MGResourceBar(
      icon: Icons.star,
      value: score.toString(),
      iconColor: Colors.amber,
    );
  }

  Widget _buildPauseButton() {
    return MGIconButton(
      icon: Icons.pause,
      onPressed: onPause,
      buttonSize: MGIconButtonSize.medium,
    );
  }

  Widget _buildActions() {
    return MGButton.primary(
      label: 'ACTION',
      onPressed: () {},
    );
  }
}
```

### 4. Integrate into Game Screen

```dart
// In your game screen
Stack(
  children: [
    // Game content
    YourGameWidget(),

    // HUD overlay
    Consumer<YourGameState>(
      builder: (context, state, _) => MGYourGameHud(
        score: state.score,
        onPause: () => state.pause(),
      ),
    ),
  ],
)
```

---

## Core Components

### MGColors

**Purpose**: Consistent color palette across all games

```dart
// Primary colors
MGColors.primary          // Default brand color
MGColors.primaryAction    // Actionable elements

// Regional themes
MGColors.indiaPrimary     // #FF6B35 (Orange)
MGColors.africaPrimary    // #FFD700 (Gold)
MGColors.seaPrimary       // #20B2AA (Teal)
MGColors.latamPrimary     // #DC143C (Red)

// UI colors
MGColors.surface          // #1E1E1E (Dark background)
MGColors.border           // #424242 (Border/divider)
MGColors.error            // Error states
MGColors.warning          // Warning states
MGColors.success          // Success states
MGColors.gold             // Currency/rewards
```

### MGTextStyles

**Purpose**: Standardized typography system

```dart
// Display (large text)
MGTextStyles.displayLarge   // 57px, Extra bold
MGTextStyles.displayMedium  // 45px, Bold
MGTextStyles.displaySmall   // 36px, Bold

// Headings
MGTextStyles.h1             // 32px
MGTextStyles.h2             // 28px
MGTextStyles.h3             // 24px

// Body text
MGTextStyles.body           // 16px, Regular
MGTextStyles.bodySmall      // 14px

// HUD-specific
MGTextStyles.hud            // 18px, Medium (main HUD text)
MGTextStyles.hudSmall       // 14px, Medium (secondary HUD info)

// Buttons
MGTextStyles.buttonLarge    // 18px
MGTextStyles.buttonMedium   // 16px
MGTextStyles.buttonSmall    // 14px

// Utility
MGTextStyles.caption        // 12px, Small annotations
```

### MGSpacing

**Purpose**: Consistent spacing/padding system

```dart
// Spacing values
MGSpacing.xxs    // 2px
MGSpacing.xs     // 4px
MGSpacing.sm     // 8px
MGSpacing.md     // 16px
MGSpacing.lg     // 24px
MGSpacing.xl     // 32px
MGSpacing.xxl    // 48px

// HUD-specific
MGSpacing.hudMargin  // 16px (standard HUD padding)

// Spacing widgets (use between components)
MGSpacing.hXs    // Horizontal 4px
MGSpacing.hSm    // Horizontal 8px
MGSpacing.hMd    // Horizontal 16px
MGSpacing.hLg    // Horizontal 24px

MGSpacing.vXs    // Vertical 4px
MGSpacing.vSm    // Vertical 8px
MGSpacing.vMd    // Vertical 16px
MGSpacing.vLg    // Vertical 24px
```

---

## HUD Structure Pattern

### Standard HUD Layout

```dart
class MGStandardHud extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return SafeArea(
      child: Padding(
        padding: EdgeInsets.all(MGSpacing.md),
        child: Column(
          children: [
            // TOP SECTION: Status info (score, resources, etc.)
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                _buildLeftStatus(),
                _buildRightStatus(),
              ],
            ),

            // MIDDLE SECTION: Expandable space for game content
            const Spacer(),

            // BOTTOM SECTION: Action buttons
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                _buildPrimaryActions(),
                _buildSecondaryActions(),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

### Full-Screen Overlay Pattern

```dart
// When HUD needs to overlay entire screen (no padding)
Positioned.fill(
  child: Column(
    children: [
      Container(
        padding: EdgeInsets.only(
          top: MediaQuery.of(context).padding.top + MGSpacing.hudMargin,
          left: MGSpacing.hudMargin,
          right: MGSpacing.hudMargin,
        ),
        child: _buildTopHud(),
      ),
      const Expanded(child: SizedBox()),
      Container(
        padding: EdgeInsets.only(
          bottom: MediaQuery.of(context).padding.bottom + MGSpacing.hudMargin,
          left: MGSpacing.hudMargin,
          right: MGSpacing.hudMargin,
        ),
        child: _buildBottomHud(),
      ),
    ],
  ),
)
```

---

## Component Reference

### 1. MGResourceBar

**Purpose**: Display resources (gold, gems, score, etc.)

```dart
MGResourceBar(
  icon: Icons.monetization_on,
  value: '1,234',
  iconColor: MGColors.gold,
  onTap: () {}, // Optional tap handler
)
```

**Parameters:**
- `icon` (IconData): Resource icon
- `value` (String): Formatted value
- `iconColor` (Color): Icon tint color
- `onTap` (VoidCallback?): Optional tap handler

**Best Practices:**
- Format large numbers: `1234567` → `"1.2M"`
- Use appropriate icons: `monetization_on` for gold, `diamond` for gems
- Match iconColor to resource type

### 2. MGButton

**Purpose**: Standard action buttons

```dart
// Primary button
MGButton.primary(
  label: 'START',
  icon: Icons.play_arrow,
  onPressed: () {},
  size: MGButtonSize.medium,
)

// Secondary button
MGButton.secondary(
  label: 'CANCEL',
  onPressed: () {},
)

// Custom styled button
MGButton(
  label: 'NEXT WAVE',
  icon: Icons.fast_forward,
  style: MGButtonStyle.filled,
  backgroundColor: MGColors.warning,
  onPressed: isDisabled ? null : () {},
  size: MGButtonSize.small,
)
```

**Button Sizes:**
- `MGButtonSize.small` - Compact buttons
- `MGButtonSize.medium` - Standard buttons (default)
- `MGButtonSize.large` - Prominent actions

**Button Styles:**
- `MGButtonStyle.filled` - Solid background
- `MGButtonStyle.outlined` - Border only
- `MGButtonStyle.text` - Text only

### 3. MGIconButton

**Purpose**: Icon-only action buttons

```dart
MGIconButton(
  icon: Icons.pause,
  onPressed: () {},
  buttonSize: MGIconButtonSize.medium,
  backgroundColor: MGColors.surface.withOpacity(0.85),
)
```

**Button Sizes:**
- `MGIconButtonSize.small` - 36x36px
- `MGIconButtonSize.medium` - 44x44px (recommended)
- `MGIconButtonSize.large` - 56x56px

**Common Uses:**
- Pause/Resume: `Icons.pause` / `Icons.play_arrow`
- Settings: `Icons.settings`
- Close/Back: `Icons.close` / `Icons.arrow_back`
- Audio: `Icons.volume_up` / `Icons.volume_off`

### 4. MGLinearProgress

**Purpose**: Progress bars (health, mana, timers, etc.)

```dart
MGLinearProgress(
  value: 0.75,                    // 0.0 to 1.0
  height: 12,
  valueColor: Colors.green,
  backgroundColor: Colors.green.withOpacity(0.3),
  borderRadius: 6,
)
```

**Parameters:**
- `value` (double): 0.0 (empty) to 1.0 (full)
- `height` (double): Bar thickness
- `valueColor` (Color): Fill color
- `backgroundColor` (Color): Track color
- `borderRadius` (double): Corner rounding

**Dynamic Color Example:**
```dart
MGLinearProgress(
  value: healthPercentage,
  height: 8,
  valueColor: healthPercentage < 0.25
      ? MGColors.error
      : Colors.green,
  backgroundColor: Colors.green.withOpacity(0.3),
)
```

---

## Regional Theming

### Theme Selection by Region

```dart
// India region
class MGIndiaGameHud extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return SafeArea(
      child: Container(
        decoration: BoxDecoration(
          border: Border.all(color: MGColors.indiaPrimary),
        ),
        child: _buildHudContent(),
      ),
    );
  }
}

// Africa region
class MGAfricaGameHud extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return SafeArea(
      child: Container(
        decoration: BoxDecoration(
          border: Border.all(color: MGColors.africaPrimary),
        ),
        child: _buildHudContent(),
      ),
    );
  }
}
```

### Regional Color Guide

| Region | Primary Color | Hex Code | Use Cases |
|--------|---------------|----------|-----------|
| India | Orange | `#FF6B35` | Diwali, Cricket, Traditional |
| Africa | Gold | `#FFD700` | Markets, Rhythms, Wildlife |
| SEA | Teal | `#20B2AA` | Islands, Ocean, Tropical |
| LATAM | Red | `#DC143C` | Festivals, Soccer, Carnivals |

---

## Examples by Game Type

### Tower Defense

```dart
class MGTowerDefenseHud extends StatelessWidget {
  final int gold;
  final int wave;
  final int lives;
  final VoidCallback? onBuildTower;
  final VoidCallback? onNextWave;

  @override
  Widget build(BuildContext context) {
    return SafeArea(
      child: Padding(
        padding: EdgeInsets.all(MGSpacing.md),
        child: Column(
          children: [
            // Top: Wave + Resources
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                _buildWaveInfo(),
                Row(
                  children: [
                    MGResourceBar(
                      icon: Icons.monetization_on,
                      value: gold.toString(),
                      iconColor: MGColors.gold,
                    ),
                    MGSpacing.hSm,
                    _buildLivesIndicator(),
                  ],
                ),
              ],
            ),
            const Spacer(),
            // Bottom: Actions
            Row(
              children: [
                MGButton.primary(
                  label: 'BUILD',
                  icon: Icons.add_circle,
                  onPressed: onBuildTower,
                ),
                MGSpacing.hMd,
                MGButton(
                  label: 'NEXT WAVE',
                  icon: Icons.fast_forward,
                  style: MGButtonStyle.filled,
                  backgroundColor: MGColors.warning,
                  onPressed: onNextWave,
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildWaveInfo() {
    return Container(
      padding: EdgeInsets.symmetric(
        horizontal: MGSpacing.md,
        vertical: MGSpacing.sm,
      ),
      decoration: BoxDecoration(
        color: MGColors.surface.withOpacity(0.85),
        borderRadius: BorderRadius.circular(MGSpacing.sm),
        border: Border.all(color: MGColors.warning),
      ),
      child: Row(
        children: [
          Icon(Icons.waves, color: MGColors.warning, size: 20),
          MGSpacing.hXs,
          Text(
            'Wave $wave',
            style: MGTextStyles.hud.copyWith(color: Colors.white),
          ),
        ],
      ),
    );
  }

  Widget _buildLivesIndicator() {
    final percentage = lives / 20.0;
    return Container(
      padding: EdgeInsets.symmetric(
        horizontal: MGSpacing.sm,
        vertical: MGSpacing.xs,
      ),
      decoration: BoxDecoration(
        color: MGColors.surface.withOpacity(0.85),
        borderRadius: BorderRadius.circular(MGSpacing.xs),
      ),
      child: Row(
        children: [
          Icon(Icons.favorite, color: Colors.red, size: 20),
          MGSpacing.hXs,
          SizedBox(
            width: 60,
            child: MGLinearProgress(
              value: percentage,
              height: 8,
              valueColor: Colors.red,
              backgroundColor: Colors.red.withOpacity(0.3),
            ),
          ),
          MGSpacing.hXs,
          Text(
            '$lives',
            style: MGTextStyles.hudSmall.copyWith(color: Colors.white),
          ),
        ],
      ),
    );
  }
}
```

### Card Battle Game

```dart
class MGCardBattleHud extends StatelessWidget {
  final int playerHp;
  final int playerMaxHp;
  final int currentMana;
  final int maxMana;
  final int floor;
  final VoidCallback? onPause;

  @override
  Widget build(BuildContext context) {
    return SafeArea(
      child: Padding(
        padding: EdgeInsets.all(MGSpacing.md),
        child: Column(
          children: [
            // Top: Floor + Pause
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Container(
                  padding: EdgeInsets.all(MGSpacing.sm),
                  decoration: BoxDecoration(
                    color: MGColors.surface.withOpacity(0.85),
                    borderRadius: BorderRadius.circular(MGSpacing.sm),
                  ),
                  child: Text(
                    'Floor $floor',
                    style: MGTextStyles.hud.copyWith(color: Colors.amber),
                  ),
                ),
                if (onPause != null)
                  MGIconButton(
                    icon: Icons.pause,
                    onPressed: onPause,
                    buttonSize: MGIconButtonSize.small,
                  ),
              ],
            ),
            const Spacer(),
            // Bottom: HP + Mana
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                _buildHpBar(),
                MGSpacing.hMd,
                _buildManaBar(),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildHpBar() {
    final hpPercentage = playerHp / playerMaxHp;
    return Expanded(
      child: Container(
        padding: EdgeInsets.all(MGSpacing.sm),
        decoration: BoxDecoration(
          color: MGColors.surface.withOpacity(0.85),
          borderRadius: BorderRadius.circular(MGSpacing.sm),
        ),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              children: [
                Icon(Icons.favorite, color: Colors.red, size: 16),
                MGSpacing.hXs,
                Text(
                  '$playerHp / $playerMaxHp',
                  style: MGTextStyles.hudSmall.copyWith(color: Colors.white),
                ),
              ],
            ),
            MGSpacing.vXs,
            MGLinearProgress(
              value: hpPercentage,
              height: 10,
              valueColor: hpPercentage < 0.3 ? MGColors.error : Colors.red,
              backgroundColor: Colors.red.withOpacity(0.3),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildManaBar() {
    final manaPercentage = currentMana / maxMana;
    return Expanded(
      child: Container(
        padding: EdgeInsets.all(MGSpacing.sm),
        decoration: BoxDecoration(
          color: MGColors.surface.withOpacity(0.85),
          borderRadius: BorderRadius.circular(MGSpacing.sm),
        ),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              children: [
                Icon(Icons.water_drop, color: Colors.blue, size: 16),
                MGSpacing.hXs,
                Text(
                  '$currentMana / $maxMana',
                  style: MGTextStyles.hudSmall.copyWith(color: Colors.white),
                ),
              ],
            ),
            MGSpacing.vXs,
            MGLinearProgress(
              value: manaPercentage,
              height: 10,
              valueColor: Colors.blue,
              backgroundColor: Colors.blue.withOpacity(0.3),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Rhythm Game

```dart
class MGRhythmGameHud extends StatelessWidget {
  final int score;
  final int combo;
  final double accuracy;
  final VoidCallback? onPause;

  @override
  Widget build(BuildContext context) {
    return SafeArea(
      child: Padding(
        padding: EdgeInsets.all(MGSpacing.sm),
        child: Column(
          children: [
            // Top: Score + Accuracy
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                _buildScoreDisplay(),
                _buildAccuracyDisplay(),
                if (onPause != null)
                  MGIconButton(
                    icon: Icons.pause,
                    onPressed: onPause,
                    buttonSize: MGIconButtonSize.small,
                  ),
              ],
            ),
            // Combo (centered, appears when combo > 1)
            if (combo > 1) _buildComboDisplay(),
          ],
        ),
      ),
    );
  }

  Widget _buildScoreDisplay() {
    return Container(
      padding: EdgeInsets.all(MGSpacing.sm),
      decoration: BoxDecoration(
        color: MGColors.surface.withOpacity(0.85),
        borderRadius: BorderRadius.circular(MGSpacing.sm),
      ),
      child: Row(
        children: [
          Icon(Icons.star, color: Colors.amber, size: 20),
          MGSpacing.hXs,
          Text(
            score.toString(),
            style: MGTextStyles.h3.copyWith(
              color: Colors.amber,
              fontWeight: FontWeight.bold,
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildAccuracyDisplay() {
    return Container(
      padding: EdgeInsets.all(MGSpacing.sm),
      decoration: BoxDecoration(
        color: MGColors.surface.withOpacity(0.85),
        borderRadius: BorderRadius.circular(MGSpacing.sm),
      ),
      child: Text(
        '${(accuracy * 100).toStringAsFixed(1)}%',
        style: MGTextStyles.hud.copyWith(
          color: accuracy > 0.95 ? Colors.greenAccent : Colors.white,
        ),
      ),
    );
  }

  Widget _buildComboDisplay() {
    Color comboColor;
    TextStyle comboStyle;

    if (combo >= 20) {
      comboColor = Colors.purpleAccent;
      comboStyle = MGTextStyles.displayLarge;
    } else if (combo >= 10) {
      comboColor = Colors.orangeAccent;
      comboStyle = MGTextStyles.displayMedium;
    } else if (combo >= 5) {
      comboColor = Colors.yellowAccent;
      comboStyle = MGTextStyles.displaySmall;
    } else {
      comboColor = Colors.white;
      comboStyle = MGTextStyles.h3;
    }

    return Padding(
      padding: EdgeInsets.only(top: MGSpacing.sm),
      child: Container(
        padding: EdgeInsets.symmetric(
          horizontal: MGSpacing.md,
          vertical: MGSpacing.xs,
        ),
        decoration: BoxDecoration(
          gradient: LinearGradient(
            colors: [
              comboColor.withOpacity(0.8),
              comboColor.withOpacity(0.3),
            ],
          ),
          borderRadius: BorderRadius.circular(MGSpacing.md),
          border: Border.all(color: comboColor, width: 2),
          boxShadow: [
            BoxShadow(
              color: comboColor.withOpacity(0.5),
              blurRadius: 12,
              spreadRadius: 2,
            ),
          ],
        ),
        child: Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(Icons.local_fire_department, color: Colors.white, size: 20),
            MGSpacing.hXs,
            Text(
              '$combo COMBO!',
              style: comboStyle.copyWith(
                color: Colors.white,
                fontWeight: FontWeight.bold,
                shadows: [
                  const Shadow(
                    color: Colors.black,
                    blurRadius: 4,
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Puzzle Game

```dart
class MGPuzzleGameHud extends StatelessWidget {
  final int moves;
  final int targetMoves;
  final int score;
  final int targetScore;
  final VoidCallback? onHint;
  final VoidCallback? onUndo;
  final VoidCallback? onPause;

  @override
  Widget build(BuildContext context) {
    return SafeArea(
      child: Padding(
        padding: EdgeInsets.all(MGSpacing.md),
        child: Column(
          children: [
            // Top: Progress
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                _buildMovesCounter(),
                _buildScoreCounter(),
              ],
            ),
            const Spacer(),
            // Bottom: Actions
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Row(
                  children: [
                    if (onHint != null)
                      MGIconButton(
                        icon: Icons.lightbulb_outline,
                        onPressed: onHint,
                        buttonSize: MGIconButtonSize.small,
                        backgroundColor: Colors.yellow.withOpacity(0.3),
                      ),
                    if (onUndo != null) ...[
                      MGSpacing.hSm,
                      MGIconButton(
                        icon: Icons.undo,
                        onPressed: onUndo,
                        buttonSize: MGIconButtonSize.small,
                      ),
                    ],
                  ],
                ),
                if (onPause != null)
                  MGIconButton(
                    icon: Icons.pause,
                    onPressed: onPause,
                    buttonSize: MGIconButtonSize.medium,
                  ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildMovesCounter() {
    final isLow = moves >= targetMoves * 0.8;
    return Container(
      padding: EdgeInsets.all(MGSpacing.sm),
      decoration: BoxDecoration(
        color: MGColors.surface.withOpacity(0.85),
        borderRadius: BorderRadius.circular(MGSpacing.sm),
        border: isLow ? Border.all(color: MGColors.warning) : null,
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            'Moves',
            style: MGTextStyles.caption.copyWith(color: Colors.white70),
          ),
          Text(
            '$moves / $targetMoves',
            style: MGTextStyles.hud.copyWith(
              color: isLow ? MGColors.warning : Colors.white,
              fontWeight: FontWeight.bold,
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildScoreCounter() {
    final achieved = score >= targetScore;
    return Container(
      padding: EdgeInsets.all(MGSpacing.sm),
      decoration: BoxDecoration(
        color: MGColors.surface.withOpacity(0.85),
        borderRadius: BorderRadius.circular(MGSpacing.sm),
        border: achieved ? Border.all(color: Colors.greenAccent) : null,
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.end,
        children: [
          Text(
            'Score',
            style: MGTextStyles.caption.copyWith(color: Colors.white70),
          ),
          Text(
            '$score / $targetScore',
            style: MGTextStyles.hud.copyWith(
              color: achieved ? Colors.greenAccent : Colors.white,
              fontWeight: FontWeight.bold,
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## Best Practices

### 1. Always Use SafeArea

```dart
// ✅ CORRECT
return SafeArea(
  child: Padding(
    padding: EdgeInsets.all(MGSpacing.md),
    child: _buildHudContent(),
  ),
);

// ❌ WRONG - HUD may be hidden by notch/status bar
return Padding(
  padding: EdgeInsets.all(MGSpacing.md),
  child: _buildHudContent(),
);
```

### 2. Format Large Numbers

```dart
String _formatNumber(int number) {
  if (number >= 1000000) {
    return '${(number / 1000000).toStringAsFixed(1)}M';
  } else if (number >= 1000) {
    return '${(number / 1000).toStringAsFixed(1)}K';
  }
  return number.toString();
}

// Examples:
// 500 → "500"
// 1500 → "1.5K"
// 2500000 → "2.5M"
```

### 3. Use Opacity for Semi-Transparent Backgrounds

```dart
// ✅ CORRECT - Readable over game content
Container(
  decoration: BoxDecoration(
    color: MGColors.surface.withOpacity(0.85),
    borderRadius: BorderRadius.circular(MGSpacing.sm),
  ),
  child: _buildContent(),
)

// ❌ WRONG - May block game view
Container(
  decoration: BoxDecoration(
    color: MGColors.surface, // Fully opaque
  ),
  child: _buildContent(),
)
```

### 4. Provide Visual Feedback for Critical States

```dart
// Example: Low health warning
Widget _buildHealthBar() {
  final isLow = health < maxHealth * 0.25;

  return Container(
    decoration: BoxDecoration(
      border: isLow ? Border.all(color: MGColors.error, width: 2) : null,
      // Flashing animation for critical state
    ),
    child: MGLinearProgress(
      value: health / maxHealth,
      valueColor: isLow ? MGColors.error : Colors.green,
    ),
  );
}
```

### 5. Use Const Constructors When Possible

```dart
// ✅ CORRECT - Better performance
const Spacer()
const SizedBox(width: 16)
const Icon(Icons.star, color: Colors.amber)

// ❌ WRONG - Unnecessary rebuilds
Spacer()
SizedBox(width: 16)
Icon(Icons.star, color: Colors.amber)
```

### 6. Consistent File Naming

```dart
// ✅ CORRECT
lib/ui/hud/mg_tower_defense_hud.dart  // File name
class MGTowerDefenseHud                 // Class name

// ❌ WRONG
lib/ui/tower_defense_hud.dart          // Missing mg_ prefix
class TowerDefenseHud                   // Missing MG prefix
```

### 7. Disable Buttons Properly

```dart
// ✅ CORRECT - Pass null to disable
MGButton.primary(
  label: 'START',
  onPressed: canStart ? _onStart : null,  // null disables button
)

// ❌ WRONG - Empty callback doesn't disable visually
MGButton.primary(
  label: 'START',
  onPressed: canStart ? _onStart : () {},
)
```

---

## Troubleshooting

### Import Errors

**Error**: `Target of URI doesn't exist: 'package:mg_common_game/...'`

**Solution**:
1. Check `pubspec.yaml` has correct path:
   ```yaml
   mg_common_game:
     path: ../../mg-common-game
   ```
2. Run `flutter pub get`
3. Restart your IDE

---

**Error**: `Undefined name 'MGColors'`

**Solution**: Add import:
```dart
import 'package:mg_common_game/core/ui/theme/mg_colors.dart';
```

Or use the all-in-one import:
```dart
import 'package:mg_common_game/core/ui/mg_ui.dart';
```

---

### Component Errors

**Error**: `The argument type 'MGIconButtonSize' can't be assigned to parameter type 'double?'`

**Solution**: Use `buttonSize` parameter, not `size`:
```dart
// ✅ CORRECT
MGIconButton(
  icon: Icons.pause,
  buttonSize: MGIconButtonSize.medium,
)

// ❌ WRONG
MGIconButton(
  icon: Icons.pause,
  size: MGIconButtonSize.medium,
)
```

---

**Error**: `The named parameter 'progressColor' isn't defined`

**Solution**: Use `valueColor` instead:
```dart
// ✅ CORRECT
MGLinearProgress(
  value: 0.75,
  valueColor: Colors.green,
)

// ❌ WRONG
MGLinearProgress(
  value: 0.75,
  progressColor: Colors.green,
)
```

---

### Layout Issues

**Problem**: HUD elements overlapping game content

**Solution**: Use Stack with proper layering:
```dart
Stack(
  children: [
    // Game content (bottom layer)
    YourGameWidget(),

    // HUD overlay (top layer)
    YourHudWidget(),
  ],
)
```

---

**Problem**: HUD hidden by notch or status bar

**Solution**: Always wrap in SafeArea:
```dart
return SafeArea(
  child: YourHudContent(),
);
```

---

**Problem**: HUD not updating when state changes

**Solution**: Wrap in state management widget:
```dart
// Using Provider
Consumer<GameState>(
  builder: (context, state, _) => MGYourGameHud(
    score: state.score,
  ),
)

// Using ValueNotifier
ValueListenableBuilder<int>(
  valueListenable: scoreNotifier,
  builder: (context, score, _) => MGYourGameHud(
    score: score,
  ),
)
```

---

## Additional Resources

- **HUD Integration Report**: See `HUD_INTEGRATION_REPORT.md` for complete integration details
- **mg_common_game Source**: `D:/mg-games/repos/mg-common-game/`
- **Example Games**:
  - MG-0001 (Tower Defense): Complex HUD with multiple resources
  - MG-0022 (Minigame): Combo display example
  - MG-0037 (Ludo): Turn-based game HUD
  - MG-0051 (Rhythm): Dynamic combo visualization

---

**Last Updated**: December 23, 2024
**Version**: 1.0
**Maintained By**: MG Games Team
