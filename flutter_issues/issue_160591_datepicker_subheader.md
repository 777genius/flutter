# Issue #160591: DatePickerThemeData Sub-Header Styling

## Overview
- **Status:** Open
- **Priority:** P3
- **Created:** December 19, 2024
- **Type:** API Enhancement
- **Estimated Time:** 30-60 minutes
- **Confidence:** 90%

## Problem Description
The DatePicker widget lacks API support for customizing the sub-header text (month/year display). The sub-header color is hardcoded, making it impossible to style independently.

### Current Limitation
```dart
// Line 787 in calendar_date_picker.dart - HARDCODED!
final Color controlColor = Theme.of(context).colorScheme.onSurface.withOpacity(0.60);
```

### User Impact
Developers must modify the global `colorScheme.onSurface` to change sub-header color, which affects ALL widgets using `onSurface`.

## Visual Example
```
┌──────────────────────┐
│   December 2024      │ ← Sub-header (hardcoded color)
├──────────────────────┤
│  Mo Tu We Th Fr ...  │
│   1  2  3  4  5 ...  │
│  ...                 │
└──────────────────────┘
```

## Expected vs Actual Behavior

### Expected
```dart
DatePickerThemeData(
  subheaderColor: Colors.blue, // ✅ Should be possible
  // ... other theme properties
)
```

### Actual
```dart
DatePickerThemeData(
  // ❌ No subheaderColor property available!
  // Must modify global colorScheme instead
)
```

## Technical Details

### File Location
`packages/flutter/lib/src/material/calendar_date_picker.dart`

### Problem Line
Line 787:
```dart
final Color controlColor = Theme.of(context).colorScheme.onSurface.withOpacity(0.60);
```

### Proposed Solution

**Step 1:** Add property to `DatePickerThemeData`:
```dart
class DatePickerThemeData {
  // ... existing properties

  /// The color of the sub-header text (month/year display)
  final Color? subheaderColor;

  const DatePickerThemeData({
    // ... existing parameters
    this.subheaderColor,
  });
}
```

**Step 2:** Use the property in `calendar_date_picker.dart`:
```dart
final DatePickerThemeData datePickerTheme = DatePickerTheme.of(context);
final Color controlColor = datePickerTheme.subheaderColor
  ?? Theme.of(context).colorScheme.onSurface.withOpacity(0.60);
```

**Step 3:** Add to `copyWith` and equality methods

## Community Validation
- **Maheshj01** (Design team): Confirmed limitation
- **15 comments** from affected users
- Transferred from framework to design team (architectural significance)

## Labels
- `c: proposal` - Proposal for new API
- `f: date/time picker` - Date/time picker widgets
- `f: material design` - Material Design components
- `framework` - Flutter framework
- `good first issue` - Approachable
- `P3` - Nice to have
- `team-design` - Owned by Design team
- `triaged-design` - Triaged by Design team

## Implementation Steps

### 1. Update DatePickerThemeData
File: `packages/flutter/lib/src/material/date_picker_theme.dart`
- Add `subheaderColor` property
- Update constructor
- Update `copyWith` method
- Update `lerp` method
- Update equality operator
- Update `debugFillProperties`

### 2. Update calendar_date_picker.dart
File: `packages/flutter/lib/src/material/calendar_date_picker.dart`
- Line 787: Replace hardcoded color with theme property
- Add null-aware fallback to current default

### 3. Add Tests
- Theme property tests
- Visual golden tests
- Integration tests

### 4. Update Documentation
- Add API documentation for new property
- Update migration guide if needed
- Add example showing subheader customization

## Testing Requirements

### Unit Tests
```dart
test('DatePickerThemeData subheaderColor', () {
  const Color testColor = Color(0xFF123456);
  final theme = DatePickerThemeData(subheaderColor: testColor);
  expect(theme.subheaderColor, testColor);
});
```

### Widget Tests
```dart
testWidgets('DatePicker uses theme subheaderColor', (tester) async {
  await tester.pumpWidget(
    MaterialApp(
      theme: ThemeData(
        datePickerTheme: DatePickerThemeData(
          subheaderColor: Colors.red,
        ),
      ),
      home: DatePickerDialog(/* ... */),
    ),
  );
  // Verify sub-header uses red color
});
```

## Breaking Changes
**None** - This is backward compatible. Existing code continues to work with default behavior.

## Migration Guide
Not required - purely additive API.

## Example Usage
```dart
MaterialApp(
  theme: ThemeData(
    datePickerTheme: DatePickerThemeData(
      backgroundColor: Colors.white,
      subheaderColor: Colors.blue.shade700, // ✅ NEW!
      headerForegroundColor: Colors.white,
    ),
  ),
);
```

## Timeline
- **Reported:** December 19, 2024
- **Triaged:** January 2025
- **Transferred to Design:** January 8, 2025

## Related Patterns
Similar properties exist in other theme data classes:
- `AppBarTheme.foregroundColor`
- `CardTheme.color`
- `DialogTheme.backgroundColor`

## Links
- Issue: https://github.com/flutter/flutter/issues/160591
- File: calendar_date_picker.dart:787
- ThemeData file: date_picker_theme.dart
