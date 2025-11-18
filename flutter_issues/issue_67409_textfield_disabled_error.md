# Issue #67409: TextField Disabled ErrorText Spacing

**Status:** Open
**Priority:** P2
**Labels:** framework, f: material design, text input, typography, triaged-text-input
**Created:** October 26, 2020
**Last Updated:** March 13, 2025
**Assignee:** None
**Found in:** 1.22, 3.3, 3.6

## Overview

- **Difficulty:** ⭐⭐ (Easy-Medium)
- **Time Estimate:** 30-60 minutes
- **Confidence:** 85%
- **Type:** UI Bug Fix / Developer Experience

## Problem Description

When a `TextField` has `enabled: false` and includes `errorText` in its `InputDecoration`, the widget allocates space for the error message but does not display it visually, creating unwanted whitespace.

## Current Behavior

```dart
TextField(
  enabled: false,
  decoration: InputDecoration(
    labelText: 'Username',
    errorText: 'This field is required',
  ),
)
```

**Result:** Empty space appears below the TextField where error text should be, but no error text is visible.

## Expected Behavior

The TextField should either:
1. **Not allocate space** for error text when disabled (preferred), OR
2. **Display the error text** even when the field is disabled

## Design Consideration

The first option (not allocating space) makes more sense because:
- Disabled fields typically don't show validation errors
- User cannot interact with disabled fields to fix errors
- Showing errors on disabled fields is confusing UX

## Proposed Solution

**Option 1 (Recommended):** Don't show or allocate space for error text when disabled

```dart
// In input_decorator.dart
Widget? _buildError() {
  if (!decoration.enabled && decoration.errorText != null) {
    return null; // Don't show error for disabled fields
  }

  // Existing error building logic
  if (decoration.errorText == null && decoration.error == null) {
    return null;
  }
  // ...
}
```

**Option 2:** Show error text even when disabled (if team prefers)

```dart
// Allow error text to be visible for disabled fields
// Just remove the check that hides it
```

## Implementation Steps

1. **File:** `packages/flutter/lib/src/material/input_decorator.dart`

2. **Locate error rendering logic** (search for `errorText` handling)

3. **Add enabled check:**
   - Before allocating space for error
   - Before rendering error text

4. **Consider both errorText and error properties** (per issue #142398)

5. **Update tests** in `packages/flutter/test/material/input_decorator_test.dart`:
   - Test disabled TextField with errorText doesn't show space
   - Test enabled TextField with errorText shows normally
   - Test disabled TextField with error widget

## Testing Requirements

```dart
testWidgets('Disabled TextField with errorText does not allocate space', (tester) async {
  await tester.pumpWidget(
    MaterialApp(
      home: Material(
        child: TextField(
          enabled: false,
          decoration: InputDecoration(
            labelText: 'Test',
            errorText: 'Error message',
          ),
        ),
      ),
    ),
  );

  // Verify error text is not present
  expect(find.text('Error message'), findsNothing);

  // Verify no extra space allocated (measure height)
  final textFieldHeight = tester.getSize(find.byType(TextField)).height;

  // Compare with disabled field without error
  await tester.pumpWidget(
    MaterialApp(
      home: Material(
        child: TextField(
          enabled: false,
          decoration: InputDecoration(labelText: 'Test'),
        ),
      ),
    ),
  );

  final heightWithoutError = tester.getSize(find.byType(TextField)).height;
  expect(textFieldHeight, equals(heightWithoutError));
});

testWidgets('Enabled TextField with errorText shows normally', (tester) async {
  await tester.pumpWidget(
    MaterialApp(
      home: Material(
        child: TextField(
          enabled: true,
          decoration: InputDecoration(
            labelText: 'Test',
            errorText: 'Error message',
          ),
        ),
      ),
    ),
  );

  expect(find.text('Error message'), findsOneWidget);
});
```

## Files to Modify

- `packages/flutter/lib/src/material/input_decorator.dart` - Add enabled check
- `packages/flutter/test/material/input_decorator_test.dart` - Add tests
- Possibly `packages/flutter/lib/src/material/text_field.dart` - Verify integration

## Why This Is a Good Pick

1. ✅ **P2 Priority** - Team triaged as important
2. ✅ **5-year old issue** - Long-standing pain point
3. ✅ **Clear reproduction** - Has reproducible steps
4. ✅ **Multiple versions affected** - 1.22, 3.3, 3.6
5. ✅ **No assignee** - Available for contribution
6. ✅ **Developer experience** - Common use case
7. ✅ **Clear solution** - Straightforward fix

## Alternative Approaches

1. Add a new property `showErrorWhenDisabled` (more complex, breaking)
2. Make it theme-configurable (overkill for this use case)
3. Document current behavior (doesn't fix the issue)

## Community Impact

- **Found in Releases:** 1.22, 3.3, 3.6
- **Last Activity:** March 2025 (still active!)
- **Affects:** Forms with conditional field disabling
- **Pain Point:** Unwanted whitespace in disabled form fields

## Edge Cases to Consider

1. **error widget vs errorText:** Both should behave consistently
2. **helperText:** Should also not show for disabled fields (verify)
3. **Animation:** Ensure smooth transition when toggling enabled state
4. **Accessibility:** Screen readers should handle appropriately

## Related Issues

- #142398 - errorText and error parameters don't alter borders equally
- #131666 - InputDecorationTheme consistency proposal

## Links

- **Issue:** https://github.com/flutter/flutter/issues/67409
- **Related PR #134001:** InputDecoration.error should activate error state
