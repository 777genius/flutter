# Issue #72267: IconButtonThemeData Splash Radius

## Overview
- **Status:** Open
- **Priority:** P3
- **Created:** December 14, 2020
- **Type:** API Enhancement
- **Estimated Time:** 20-30 minutes
- **Confidence:** 85%

## Problem Description
`IconButton` supports customizing splash radius per-widget, but `IconButtonThemeData` lacks a property to set a default splash radius at theme level.

### Current Limitation
```dart
// Must set on every IconButton individually ❌
IconButton(
  icon: Icon(Icons.home),
  splashRadius: 20,  // Have to repeat this everywhere!
  onPressed: () {},
)

IconButton(
  icon: Icon(Icons.settings),
  splashRadius: 20,  // Again...
  onPressed: () {},
)
```

### Desired API
```dart
// Set once in theme ✅
MaterialApp(
  theme: ThemeData(
    iconButtonTheme: IconButtonThemeData(
      style: ButtonStyle(
        splashRadius: 20,  // Proposed property
      ),
    ),
  ),
)

// All IconButtons use theme splash radius automatically
IconButton(icon: Icon(Icons.home), onPressed: () {})
IconButton(icon: Icon(Icons.settings), onPressed: () {})
```

## User Request
From reporter (putra4648):
> "Whenever I want to set `defaultSplashRadius` I need to change that IconButton property one by one... add `defaultSplashRadius` to become customize? Since the default size is might large to me"

## Technical Details

### Current IconButtonThemeData
File: `packages/flutter/lib/src/material/icon_button_theme.dart`

```dart
class IconButtonThemeData {
  const IconButtonThemeData({
    this.style,
  });

  final ButtonStyle? style;
  // ❌ No splash radius property!
}
```

### IconButton Implementation
File: `packages/flutter/lib/src/material/icon_button.dart`

```dart
class IconButton extends StatelessWidget {
  const IconButton({
    // ...
    this.splashRadius,  // ✅ Per-widget property exists
  });

  final double? splashRadius;
}
```

## Proposed Implementation

### Option 1: Add to IconButtonThemeData
```dart
class IconButtonThemeData {
  const IconButtonThemeData({
    this.style,
    this.splashRadius,  // ✅ NEW!
  });

  final ButtonStyle? style;
  final double? splashRadius;  // ✅ NEW!

  // Update copyWith, lerp, ==, hashCode accordingly
}
```

### Option 2: Add to ButtonStyle (Better!)
```dart
// ButtonStyle already exists in IconButtonThemeData.style
// Just extend ButtonStyle to include splashRadius

@immutable
class ButtonStyle {
  // ... existing properties
  final MaterialStateProperty<double?>? splashRadius;  // ✅ NEW!
}
```

**Option 2 is preferred** because it's more consistent with Material 3 theming patterns.

## Material 3 Consideration

### Important Note
From sarushili0430 (Feb 2025):
> "splashRadius is no longer used in Material3"

### Impact
- Material 2: Splash radius is visible circular ripple
- Material 3: Uses different splash effect (state overlay)

### Recommendation
Still implement this for:
1. **Backward compatibility** with Material 2 apps
2. **Migration path** - apps transitioning to M3
3. **Flexibility** - some apps may want to keep M2 splash style

Mark property with:
```dart
/// This property is only used when [ThemeData.useMaterial3] is false.
/// Material 3 uses state overlay layers instead of splash radius.
final double? splashRadius;
```

## Pattern from Other Widgets

### Similar Properties in ThemeData
```dart
// FloatingActionButtonThemeData has shape, elevation
FloatingActionButtonThemeData(
  shape: CircleBorder(),
  elevation: 6,
)

// ElevatedButtonThemeData has style
ElevatedButtonThemeData(
  style: ButtonStyle(
    elevation: MaterialStateProperty.all(2),
  ),
)

// ChipThemeData has padding
ChipThemeData(
  padding: EdgeInsets.all(4),
)
```

Pattern: Widget-specific ThemeData should mirror widget properties.

## Implementation Steps

### 1. Update ButtonStyle
File: `packages/flutter/lib/src/material/button_style.dart`

Add property:
```dart
final MaterialStateProperty<double?>? splashRadius;
```

Update:
- Constructor
- `copyWith` method
- `lerp` method
- `==` operator
- `hashCode`
- `debugFillProperties`

### 2. Update IconButton
File: `packages/flutter/lib/src/material/icon_button.dart`

Use theme value:
```dart
@override
Widget build(BuildContext context) {
  final IconButtonThemeData iconButtonTheme = IconButtonTheme.of(context);
  final double? effectiveSplashRadius = splashRadius
    ?? iconButtonTheme.style?.splashRadius?.resolve(states)
    ?? _defaultSplashRadius;

  // Use effectiveSplashRadius
}
```

### 3. Add Tests
```dart
testWidgets('IconButton respects theme splash radius', (tester) async {
  await tester.pumpWidget(
    MaterialApp(
      theme: ThemeData(
        iconButtonTheme: IconButtonThemeData(
          style: ButtonStyle(
            splashRadius: MaterialStateProperty.all(15.0),
          ),
        ),
      ),
      home: IconButton(
        icon: Icon(Icons.add),
        onPressed: () {},
      ),
    ),
  );

  // Verify splash radius is 15.0
  final InkResponse inkResponse = tester.widget(find.byType(InkResponse));
  expect(inkResponse.radius, 15.0);
});
```

### 4. Update Documentation
- Add API docs for new property
- Add example in IconButtonThemeData docs
- Update migration guide (Material 2 to Material 3)

## Community Engagement
- **17 comments** over 4 years
- **4 👍 reactions**
- Common request from developers
- Multiple users confirming need

## Labels
- `P3` - Nice to have
- `c: new feature` - New feature request
- `c: proposal` - Proposal stage
- `f: material design` - Material Design
- `framework` - Framework issue
- `good first issue` - Approachable
- `team-design` - Design team
- `triaged-design` - Triaged

## Breaking Changes
**None** - This is purely additive API.

## Testing Requirements

### Unit Tests
- Theme property serialization
- `copyWith` behavior
- `lerp` interpolation
- Default value fallback

### Widget Tests
- Theme value usage
- Widget override precedence
- Material 2 vs Material 3 behavior

### Integration Tests
- Full app with themed IconButtons
- Theme switching

## Migration Path
```dart
// Before (Material 2)
IconButton(
  icon: Icon(Icons.home),
  splashRadius: 20,
  onPressed: () {},
)

// After with theme (Material 2)
// In ThemeData
iconButtonTheme: IconButtonThemeData(
  style: ButtonStyle(
    splashRadius: MaterialStateProperty.all(20),
  ),
)

// Widget code - splash radius automatic!
IconButton(
  icon: Icon(Icons.home),
  onPressed: () {},
)

// Material 3 - property ignored (as documented)
useMaterial3: true,
```

## Timeline
- **Reported:** December 14, 2020
- **Last activity:** February 2025
- **Duration open:** 4+ years
- **Status:** Still awaiting implementation

## Related Issues
- General Material 3 migration issues
- Theme consistency improvements

## Links
- Issue: https://github.com/flutter/flutter/issues/72267
- File: icon_button_theme.dart
- IconButton: icon_button.dart
