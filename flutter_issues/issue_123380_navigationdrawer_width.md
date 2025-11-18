# Issue #123380: NavigationDrawer Width M3 Specification

**Status:** Open
**Priority:** P2
**Labels:** framework, f: material design, team-design, triaged-design
**Created:** March 28, 2023
**Last Updated:** August 21, 2024
**Assignee:** None

## Overview

- **Difficulty:** ⭐⭐⭐ (Medium)
- **Time Estimate:** 1-2 hours
- **Confidence:** 70%
- **Type:** Material 3 Specification Compliance

## Problem Description

The `NavigationDrawer` component uses a hardcoded width of **304 dp** for both Material 2 and Material 3 modes. According to Material Design 3 specifications, the drawer should be **360 dp** wide in M3 mode.

**Current Code (drawer.dart):**
```dart
const double _kWidth = 304.0;
```

## Material 3 Specification

- **M3 Standard Width:** 360 dp
- **M2 Standard Width:** 304 dp (current implementation)

## Design Challenges

The issue author (rydmike) conducted extensive testing and identified problems with a fixed 360 dp width:

**Small Devices (iPhone SE, Galaxy S20):**
- 360 dp consumes nearly the entire screen
- Eliminates visible background for tap-to-dismiss
- Poor user experience

**Large Devices (iPhone 14 Pro Max):**
- 360 dp works adequately
- Provides reasonable background visibility

## Proposed Solution

Add two new properties to control drawer width responsively:

```dart
NavigationDrawer(
  maxWidth: 360.0,  // Maximum drawer width
  edgeSpace: 56.0,   // Minimum uncovered space on open side
  // Actual width = min(maxWidth, screenWidth - edgeSpace)
  // ...
)
```

This approach follows Material 2 specs: "Width = Screen width − 56 dp" with maximum constraints.

## Alternative Approaches

1. **Conditional width based on useMaterial3:**
```dart
final double width = Theme.of(context).useMaterial3 ? 360.0 : 304.0;
```

2. **Add to DrawerThemeData:**
```dart
DrawerThemeData(
  width: 360.0,  // Already exists
  // Add responsive properties
)
```

3. **Device-aware defaults:**
```dart
double _getDefaultWidth(BuildContext context) {
  final width = MediaQuery.of(context).size.width;
  final defaultMax = Theme.of(context).useMaterial3 ? 360.0 : 304.0;
  return min(defaultMax, width - 56.0);
}
```

## Implementation Considerations

**File Locations:**
- `packages/flutter/lib/src/material/drawer.dart` - Main implementation
- `packages/flutter/lib/src/material/drawer_theme.dart` - Theme properties
- `packages/flutter/lib/src/material/navigation_drawer.dart` - NavigationDrawer widget

**Backward Compatibility:**
- Current `width` property in DrawerThemeData must remain functional
- Default behavior should change based on useMaterial3 flag
- Existing apps should not break

## Testing Requirements

- Test M2 mode (304 dp)
- Test M3 mode (360 dp)
- Test responsive behavior on various screen sizes
- Test DrawerThemeData.width override
- Golden tests for visual regression

## Team Status

- **Triaged by design team** ✅
- **P2 Priority** - Important but not urgent
- **Found in releases:** 3.7, 3.9
- **No assignee** - Available for contribution

## Complexity Factors

1. 📋 Requires design team decision on approach
2. 📋 Backward compatibility concerns
3. 📋 Multiple implementation paths available
4. 📋 Responsive design considerations
5. 📋 Material 3 migration path

## Why This Matters

- **Material 3 Compliance:** Aligns with official M3 specifications
- **Developer Experience:** Reduces manual width calculations
- **Responsive Design:** Better UX across device sizes
- **Framework Quality:** Completes M3 migration

## Current Workaround

Developers can manually set width through DrawerThemeData:

```dart
MaterialApp(
  theme: ThemeData(
    useMaterial3: true,
    drawerTheme: DrawerThemeData(width: 360),
  ),
)
```

## Links

- **Issue:** https://github.com/flutter/flutter/issues/123380
- **Related:** #19724 (Drawer width device responsiveness)
- **Related:** #123507 (NavigationDrawer umbrella issue)
- **M3 Spec:** https://m3.material.io/components/navigation-drawer/specs
