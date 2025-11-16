# Issue #135186: FLTEnableWideGamut iOS Bundle Check

## Overview
- **Status:** Open
- **Priority:** P2
- **Created:** September 20, 2023
- **Type:** Bug Fix / Enhancement
- **Estimated Time:** 10-15 minutes
- **Confidence:** 95%

## Problem Description
The `FLTEnableWideGamut` configuration flag is only checked in the main bundle, but should also check the app bundle parameter for add-to-app scenarios.

### Current Implementation
```objc
// Current code only checks mainBundle
NSNumber* nsEnableWideGamut = [mainBundle objectForInfoDictionaryKey:@"FLTEnableWideGamut"];
```

### Expected Implementation
Similar to `FLTEnableImpeller` which checks both bundles:
```objc
NSNumber* nsEnableImpeller = [bundle objectForInfoDictionaryKey:@"FLTEnableImpeller"];
if (nsEnableImpeller == nil) {
  nsEnableImpeller = [mainBundle objectForInfoDictionaryKey:@"FLTEnableImpeller"];
}
```

## Technical Details

### File Location
`/home/user/flutter/engine/src/flutter/shell/platform/darwin/ios/framework/Source/FlutterDartProject.mm`

### Line Number
Approximately line 169

### Current Code
```objc
NSNumber* nsEnableWideGamut = [mainBundle objectForInfoDictionaryKey:@"FLTEnableWideGamut"];
```

### Required Fix
```objc
NSNumber* nsEnableWideGamut = [bundle objectForInfoDictionaryKey:@"FLTEnableWideGamut"];
if (nsEnableWideGamut == nil) {
  nsEnableWideGamut = [mainBundle objectForInfoDictionaryKey:@"FLTEnableWideGamut"];
}
```

## Impact
- **Use Case:** Add-to-app scenarios where Flutter is embedded in existing iOS apps
- **Scope:** iOS platform only
- **Breaking Change:** No - this is backward compatible

## Labels
- `team-engine` - Owned by Engine team
- `platform-ios` - iOS specific
- `good first issue` - Easy for new contributors
- `triaged-engine` - Already triaged
- `P2` - Important priority

## Implementation Steps
1. Locate the file: `FlutterDartProject.mm`
2. Find the `FLTEnableWideGamut` check (around line 169)
3. Replace single mainBundle check with dual bundle/mainBundle check
4. Follow the pattern used for `FLTEnableImpeller` (lines 95-99)
5. Test with add-to-app scenario

## Testing
- **Unit test:** Check both bundle and mainBundle paths
- **Integration test:** Verify add-to-app scenario respects bundle setting
- **Regression test:** Ensure existing apps still work

## Related Code
Reference implementation for `FLTEnableImpeller`:
```objc
// Lines 95-99 in FlutterDartProject.mm
NSNumber* nsEnableImpeller = [bundle objectForInfoDictionaryKey:@"FLTEnableImpeller"];
if (nsEnableImpeller == nil) {
  nsEnableImpeller = [mainBundle objectForInfoDictionaryKey:@"FLTEnableImpeller"];
}
```

## Author
- Reported by: Dan Field (@dnfield) - Flutter team member
- Date: September 20, 2023

## Priority History
- Initially: P1
- Adjusted to: P2 (February 26, 2024)

## Links
- Issue: https://github.com/flutter/flutter/issues/135186
- Related file: FlutterDartProject.mm
