# Issue #59462: Add Dialog Theme for SimpleDialog

**Status:** Open
**Priority:** P2
**Labels:** c: new feature, c: proposal, f: material design, framework, team-design, triaged-design
**Created:** June 15, 2020
**Last Updated:** January 22, 2025
**Assignee:** None

## Overview

- **Difficulty:** ⭐⭐ (Easy)
- **Time Estimate:** 30-45 minutes
- **Confidence:** 95%
- **Type:** Enhancement - Missing Theme Property

## Problem Description

SimpleDialog components currently only use the background color from DialogTheme defined in MaterialApp. They ignore the `titleTextStyle` and `contentTextStyle` properties, instead defaulting to headline16 typography.

This creates an inconsistency where developers cannot theme SimpleDialog text styles through the global theme, forcing them to specify styles individually on each SimpleDialog instance.

## Current Behavior

```dart
MaterialApp(
  theme: ThemeData(
    dialogTheme: DialogTheme(
      titleTextStyle: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
      contentTextStyle: TextStyle(fontSize: 14, color: Colors.grey),
    ),
  ),
  // ...
)

// SimpleDialog IGNORES titleTextStyle and contentTextStyle from theme
SimpleDialog(
  title: Text('Title'),  // Uses default headline16, not theme style
  children: [
    Text('Content'),  // Uses default style, not theme style
  ],
)
```

## Expected Behavior

SimpleDialog should respect both `titleTextStyle` and `contentTextStyle` from DialogTheme, similar to how AlertDialog works.

## Team Comments

**March 1, 2021** - @darshankawar noted:
> `titleTextStyle` has since become available for SimpleDialog, but `contentTextStyle` remains missing.

This means **only contentTextStyle needs to be added** now.

## Implementation Steps

1. **File:** `packages/flutter/lib/src/material/dialog.dart`

2. **Locate SimpleDialog build method** (around line 800-900)

3. **Add contentTextStyle resolution:**
```dart
final DialogTheme dialogTheme = DialogTheme.of(context);
final DialogTheme defaults = DialogTheme.defaults(context);

final TextStyle? contentTextStyle =
  dialogTheme.contentTextStyle ?? defaults.contentTextStyle;
```

4. **Apply to children:**
```dart
DefaultTextStyle(
  style: contentTextStyle!,
  child: Column(children: children),
)
```

5. **Update tests** in `packages/flutter/test/material/dialog_test.dart`

## Testing Requirements

- Add test verifying contentTextStyle from theme is applied
- Test that explicit TextStyle on child Text widgets overrides theme
- Test null safety handling

## Files to Modify

- `packages/flutter/lib/src/material/dialog.dart` - Add contentTextStyle support
- `packages/flutter/test/material/dialog_test.dart` - Add tests

## Community Impact

- **Issue Reactions:** 1 👍
- **Comments:** 3
- **Opened:** 4.5 years ago
- **Affects:** All developers using SimpleDialog who want consistent theming

## Why This Is a Good Pick

1. ✅ Clearly scoped - only add contentTextStyle
2. ✅ Team confirmed titleTextStyle already works
3. ✅ Recently prioritized (P2 label added Jan 2025)
4. ✅ No assignee - available for contribution
5. ✅ Straightforward implementation - follow AlertDialog pattern
6. ✅ High confidence of acceptance (triaged-design label)

## Links

- **Issue:** https://github.com/flutter/flutter/issues/59462
- **Related:** AlertDialog implementation for reference pattern
