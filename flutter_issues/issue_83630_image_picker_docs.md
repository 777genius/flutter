# Issue #83630: [image_picker] maxDuration Documentation

## Overview
- **Status:** Open
- **Priority:** P2
- **Created:** May 30, 2021
- **Type:** Documentation Fix
- **Estimated Time:** 10 minutes
- **Confidence:** 100%

## Problem Description
The `maxDuration` parameter in `image_picker` package is misleading. Documentation doesn't clarify that it only applies to **recording videos**, not filtering gallery selections.

### Current Behavior
- **Camera recordings:** Respects maxDuration (stops at specified time)
- **Gallery selections:** Allows videos exceeding maxDuration to be selected ❌

### Example Code
```dart
final XFile? video = await picker.pickVideo(
  source: ImageSource.gallery,
  maxDuration: Duration(seconds: 30)
);
// Videos longer than 30 seconds CAN still be selected from gallery!
```

## Expected Outcome
Documentation should clearly state that `maxDuration` **only applies to recording**, not to filtering gallery content.

## Technical Details
- **Package:** image_picker ^0.7.5+3 (and later versions)
- **Platforms:** iOS 14+ and Android 10+
- **Confirmed by:** Flutter team member (iOS 14.4.1)

## Community Impact
- 12 comments from affected users
- Multiple developers confirmed this confusion
- Common misconception in Flutter community

## Labels
- `P2` - Important priority
- `good first issue` - Easy entry point
- `has reproducible steps`
- `Owned by Ecosystem team`

## Proposed Fix
Update documentation in:
1. `image_picker` package README
2. API docs for `pickVideo()` method
3. Parameter documentation for `maxDuration`

**Documentation should state:**
> The `maxDuration` parameter sets the maximum recording duration for video capture from the camera. It does NOT filter videos when selecting from the gallery. Gallery videos of any duration can be selected regardless of this parameter.

## Files to Modify
- `packages/image_picker/image_picker/README.md`
- `packages/image_picker/image_picker/lib/src/image_picker.dart` (dartdoc comments)

## Implementation Notes
- Pure documentation change - no code modifications needed
- Should add example showing both camera recording and gallery selection behavior
- Consider adding a note in CHANGELOG.md

## Related Issues
None

## Links
- Issue: https://github.com/flutter/flutter/issues/83630
