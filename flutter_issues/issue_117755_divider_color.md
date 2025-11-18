# Issue #117755: Divider Color Inconsistencies (M3)

**Status:** Open
**Priority:** P2
**Labels:** framework, f: material design, a: quality, team-design
**Created:** January 4, 2023
**Last Updated:** March 6, 2024
**Assignee:** None

## Overview

- **Difficulty:** ⭐⭐ (Easy-Medium)
- **Time Estimate:** 30-45 minutes
- **Confidence:** 90%
- **Type:** Material 3 Specification Compliance / Theme Bug Fix

## Problem Description

Flutter's `dividerColor` property returns inconsistent values across different theme creation methods and Material Design versions. The `Theme.of(context).dividerColor` often differs from the actual color used by the `Divider` widget, causing developer confusion and specification violations.

## Current Issues (Actual Behavior)

| Theme Factory | Material Version | dividerColor Property | Actual Divider Widget | Issue |
|---|---|---|---|---|
| ThemeData(colorSchemeSeed) | M2 | outline | outline | ❌ Wrong per M2 spec |
| ThemeData() | M3 | outline | outlineVariant | ❌ Property mismatched |
| ThemeData.from() | M3 | onSurface.withOpacity(0.12) | outlineVariant | ❌ Property mismatched |
| ThemeData(colorSchemeSeed) | M3 | outline | outlineVariant | ❌ Wrong per M3 spec |

## Expected Behavior

- **Material 2 mode**: Both `dividerColor` and `Divider` widget should use `ColorScheme.onSurface.withOpacity(0.12)`
- **Material 3 mode**: Both should use `ColorScheme.outlineVariant`

## Proposed Solution

Update theme factories with conditional logic:

**In `ThemeData` constructor:**
```dart
dividerColor: dividerColor ?? (useMaterial3
  ? colorScheme.outlineVariant
  : colorScheme.onSurface.withOpacity(0.12)),
```

**In `ThemeData.from` constructor:**
```dart
dividerColor: dividerColor ?? (useMaterial3
  ? colorScheme.outlineVariant
  : colorScheme.onSurface.withOpacity(0.12)),
```

## Implementation Steps

1. **File:** `packages/flutter/lib/src/material/theme_data.dart`

2. **Update ThemeData constructor** (around line 600-800):
   - Locate dividerColor initialization
   - Add conditional based on useMaterial3

3. **Update ThemeData.from constructor** (around line 1300-1500):
   - Locate dividerColor initialization
   - Add same conditional logic

4. **Update Divider defaults** if needed (verify current implementation)

5. **Update tests** in `packages/flutter/test/material/theme_test.dart`:
   - Test M2 dividerColor matches spec
   - Test M3 dividerColor matches spec
   - Test Divider widget uses same color

## Testing Requirements

```dart
testWidgets('ThemeData dividerColor matches Divider widget in M2', (tester) async {
  final theme = ThemeData(
    colorSchemeSeed: Colors.blue,
    useMaterial3: false,
  );

  expect(
    theme.dividerColor,
    theme.colorScheme.onSurface.withOpacity(0.12),
  );

  // Test actual Divider widget uses same color
});

testWidgets('ThemeData dividerColor matches Divider widget in M3', (tester) async {
  final theme = ThemeData(
    colorSchemeSeed: Colors.blue,
    useMaterial3: true,
  );

  expect(theme.dividerColor, theme.colorScheme.outlineVariant);

  // Test actual Divider widget uses same color
});

testWidgets('ThemeData.from dividerColor matches spec', (tester) async {
  // Similar tests for ThemeData.from
});
```

## Files to Modify

- `packages/flutter/lib/src/material/theme_data.dart` - Update dividerColor defaults
- `packages/flutter/lib/src/material/divider.dart` - Verify widget implementation
- `packages/flutter/test/material/theme_test.dart` - Add comprehensive tests
- `packages/flutter/test/material/divider_test.dart` - Add widget tests

## Why This Is a Good Pick

1. ✅ **P2 Priority** - Team considers it important
2. ✅ **Clear specification** - M2/M3 specs define exact colors
3. ✅ **Complete reproduction code** - Issue includes test cases
4. ✅ **Exact fix proposals** - Solution clearly defined
5. ✅ **No assignee** - Available for contribution
6. ✅ **Quality issue** - Affects developer experience
7. ✅ **Multiple theme factories affected** - High impact

## Material Design Specifications

**Material 2:**
- Divider color: `onSurface` at 12% opacity
- Source: Material Design 2 guidelines

**Material 3:**
- Divider color: `outlineVariant`
- Source: Material Design 3 color system

## Community Impact

- **Issue Reactions:** Multiple developers affected
- **Last Activity:** March 2024
- **Affects:** All apps using dividers with custom themes
- **Developer Confusion:** Theme property doesn't match widget behavior

## Related Issues

- #10464 - Inconsistent divider color during ExpansionTile animation
- Theme consistency issues across Material versions

## Links

- **Issue:** https://github.com/flutter/flutter/issues/117755
- **M3 Spec:** https://m3.material.io/components/divider/specs
- **M2 Spec:** Material Design 2 guidelines (archived)
