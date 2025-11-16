# Issue #174828: textScaler.clamp Regression

## Overview
- **Status:** Open
- **Priority:** P2
- **Type:** REGRESSION BUG
- **Estimated Time:** 15-20 minutes
- **Confidence:** 95%
- **Affected Versions:** Flutter 3.35.2, 3.36.0-1.0.pre
- **Working Versions:** Flutter 3.32.8 and earlier

## Problem Description
The `TextScaler.clamp()` method throws an assertion error even when parameters are valid.

### Error Message
```
Failed assertion: line 118 pos 80: 'maxScale > minScale': is not true.
```

### Reproduction Code
```dart
final textScaler = MediaQuery.textScalerOf(context);
final clampedScaler = textScaler.clamp(
  minScaleFactor: 1.0,
  maxScaleFactor: 1.1
);
// ❌ Throws assertion error even though 1.0 < 1.1
```

## Technical Details

### File Location
`packages/flutter/lib/src/painting/text_scaler.dart`

### Problematic Code (Line 118)
```dart
_ClampedTextScaler(TextScaler scaler, double minScale, double maxScale)
  : assert(maxScale > minScale),  // ❌ This assertion fails incorrectly
    // ...
```

### Root Cause Analysis
The issue occurs when creating nested `_ClampedTextScaler` instances. The constructor on line 137:
```dart
_ClampedTextScaler(
  scaler,
  max(minScaleFactor, minScale),  // Could be >= maxScale
  min(maxScaleFactor, maxScale)   // Due to nested clamp logic
);
```

When `max(minScaleFactor, minScale)` >= `min(maxScaleFactor, maxScale)`, the assertion `maxScale > minScale` fails.

## Stack Trace
```
#0 _ClampedTextScaler._ClampedTextScaler.<anonymous closure>
   package:flutter/src/painting/text_scaler.dart:118
#1 _ClampedTextScaler._ClampedTextScaler
   package:flutter/src/painting/text_scaler.dart:118
#2 _ClampedTextScaler.clamp
   package:flutter/src/painting/text_scaler.dart:137
#3 MediaQuery.withClampedTextScaling
   package:flutter/src/widgets/media_query.dart:613
```

## Real-World Impact
Affects apps using:
- `MediaQuery.withClampedTextScaling`
- `GetMaterialApp` with global text scale limits
- Date pickers with clamped text scaling
- Any widget that internally clamps text scaling

### Example Crash Scenario
```dart
GetMaterialApp(
  builder: (context, child) {
    return MediaQuery.withClampedTextScaling(
      minScaleFactor: 1,
      maxScaleFactor: 1.1,
      child: child!,
    );
  },
  // App crashes when showing date picker or other dialogs
);
```

## Proposed Fix

### Option 1: Fix the assertion logic
```dart
_ClampedTextScaler(TextScaler scaler, double minScale, double maxScale)
  : assert(maxScale >= minScale),  // Use >= instead of >
    // ...
```

### Option 2: Fix the clamp calculation (line 137)
```dart
_ClampedTextScaler(
  scaler,
  min(max(minScaleFactor, minScale), maxScale),  // Ensure min <= max
  max(min(maxScaleFactor, maxScale), minScale)   // Ensure max >= min
);
```

## Labels
- `framework` - Flutter framework
- `c: regression` - Regression from previous versions
- `P2` - Important priority
- `has reproducible steps`
- `a: error message`
- `team-framework`

## Community Impact
- 9 comments confirming the issue
- Multiple reports from production apps
- Blocking upgrades to Flutter 3.35+

## Testing
**Test cases needed:**
1. Single clamp with valid parameters
2. Nested clamps with overlapping ranges
3. Edge case: minScale == maxScale
4. Integration test with `MediaQuery.withClampedTextScaling`

## Related Issues
None

## Timeline
- **Reported:** September 2, 2025
- **Triaged:** Same day
- **Status:** Awaiting fix

## Links
- Issue: https://github.com/flutter/flutter/issues/174828
- File: text_scaler.dart:118
