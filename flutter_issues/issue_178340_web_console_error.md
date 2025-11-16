# Issue #178340: Flutter Web console.error vs console.log

## Overview
- **Status:** Open
- **Priority:** P1 (HIGHEST!)
- **Created:** November 11, 2025
- **Type:** Bug Fix / Developer Experience
- **Estimated Time:** 60-90 minutes
- **Confidence:** 80%

## Problem Description
Flutter web applications use `console.log()` instead of `console.error()` for error messages, making it impossible to distinguish errors from debug output in dev tools.

### Impact on Debugging
```javascript
// Current behavior
console.log("An error occurred!");  // ❌ Errors look like normal logs

// Expected behavior
console.error("An error occurred!"); // ✅ Errors show up red in console
```

## Real-World Impact

### For Developers
- **Cannot filter errors** in browser DevTools
- **Mixed stderr/stdout** - all goes to console.log
- **Hard to spot critical issues** in log flood
- **Monitoring tools confused** - can't categorize error severity

### For Tooling (DWDS)
The DWDS (Dart Web Debug Service) tool converts Chrome Debug Protocol to VM Service:
- `console.log` → stdout stream
- `console.error` → stderr stream

**Current problem:** Errors incorrectly routed to stdout!

## Technical Details

### Error Flow Chain
```
1. gestures/binding.dart (error occurs)
   ↓
2. assertions.dart (FlutterError.presentError)
   ↓
3. print.dart (final output)
   ↓
4. console.log() ❌ WRONG! Should be console.error()
```

### Root Cause
Web platform doesn't override `FlutterError.presentError` to use proper error streams like other platforms do.

### Files Involved
1. `packages/flutter/lib/src/foundation/assertions.dart`
2. `packages/flutter/lib/src/foundation/print.dart`
3. `packages/flutter/lib/src/widgets/binding.dart`

## How Other Platforms Handle This

### iOS/Android/Desktop
```dart
// Non-web platforms override FlutterError.presentError
FlutterError.presentError = (details) {
  // Routes to Flutter.Error stream
  developer.log(
    details.toString(),
    level: 900, // ERROR level
    name: 'Flutter.Error',
  );
};
```

### What Web Should Do
```dart
// Web should similarly route errors properly
void _webErrorHandler(FlutterErrorDetails details) {
  // Use console.error instead of console.log
  web.console.error(details.toStringShort());
  web.console.error(details.stack.toString());
}
```

## Proposed Solutions

### Option 1: Override presentError for Web
```dart
// In widget_inspector.dart or similar
if (kIsWeb) {
  FlutterError.presentError = (details) {
    // Use console.error API
    _printErrorToWebConsole(details);
  };
}
```

### Option 2: Update print.dart for Web
```dart
// In print.dart
void debugPrintError(String message) {
  if (kIsWeb) {
    web.console.error(message);
  } else {
    debugPrint(message);
  }
}
```

### Option 3: Check Error Context in print()
```dart
// Detect if message is an error
void print(Object? object) {
  if (kIsWeb && _isError(object)) {
    web.console.error(object.toString());
  } else {
    // Normal print logic
  }
}
```

## Implementation Plan

### Phase 1: Investigation (20 min)
1. Map complete error flow on web
2. Identify all error output points
3. Check for breaking changes

### Phase 2: Implementation (40-60 min)
1. Add web-specific error handler
2. Override `FlutterError.presentError` for web
3. Update print.dart if needed
4. Add tests

### Phase 3: Testing (20 min)
1. Manual testing in Chrome DevTools
2. Verify DWDS integration
3. Check VM service error stream
4. Test with various error types

## Testing Requirements

### Manual Test
```dart
// Test app
void main() {
  runApp(MaterialApp(
    home: ElevatedButton(
      onPressed: () => throw Exception('Test error'),
      child: Text('Trigger Error'),
    ),
  ));
}
```

**Verify:**
- ✅ Error shows in red in console
- ✅ Appears in "Errors" filter
- ✅ Routes to stderr in DWDS
- ✅ No breaking changes for existing code

### Automated Tests
- Unit tests for error routing logic
- Integration tests with DWDS
- Golden tests for error format

## Labels
- `P1` - Highest priority (blocks tooling)
- `platform-web` - Web specific
- `framework` - Framework issue
- `team-web` - Owned by Web team
- `triaged-web` - Already triaged
- `has reproducible steps`
- `found in release: 3.35`
- `found in release: 3.38`

## Related Issues
- dart-lang/ai#283 - Error monitoring blocked by this

## Community Impact
- **6 comments** discussing the issue
- Critical for web debugging experience
- Blocks AI/monitoring integrations

## Reporter
- **Jacob MacDonald** (@jakemac53) - Dart team member
- **Jason Simmons** - Also involved in discussion

## Breaking Changes
**None expected** - This should be transparent to app code.

## Performance Impact
**Minimal** - Just changing which console API is called.

## Timeline
- **Reported:** November 11, 2025
- **Triaged:** Same day (P1 priority)
- **Status:** Awaiting implementation

## Success Criteria
1. Errors appear red in Chrome DevTools
2. DevTools "Errors" filter works
3. DWDS routes errors to stderr stream
4. VM service `stderr` stream receives errors
5. No regression in error reporting
6. All platforms behave consistently

## Links
- Issue: https://github.com/flutter/flutter/issues/178340
- DWDS: https://github.com/dart-lang/webdev
- Related AI issue: https://github.com/dart-lang/ai/issues/283
