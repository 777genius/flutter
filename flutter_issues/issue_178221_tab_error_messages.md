# Issue #178221: Improve Tab Widget Error Messages

## Overview
- **Status:** Open
- **Priority:** P3
- **Created:** November 2025
- **Type:** Developer Experience / Error Messages
- **Estimated Time:** 20-30 minutes
- **Confidence:** 75%

## Problem Description
The `Tab` widget has assertions without descriptive error messages, making it hard to understand what went wrong when invalid arguments are provided.

### Current Error (Unhelpful)
```dart
const Tab(text: 'Hello', child: Text('World'))
// Error: "The assertion in this constant expression failed."
// ❌ No context about what's wrong!
```

### Expected Error (Helpful)
```
Error: Provide either text or child, not both.
```

## Technical Details

### File Location
`packages/flutter/lib/src/material/tabs.dart`

### Problem Lines (123-124)
```dart
const Tab({
  super.key,
  this.text,
  this.icon,
  this.iconMargin,
  this.height,
  this.child
}) : assert(text != null || child != null || icon != null),  // Line 123 - No message!
     assert(text == null || child == null);                  // Line 124 - No message!
```

### Desired Implementation
```dart
const Tab({
  super.key,
  this.text,
  this.icon,
  this.iconMargin,
  this.height,
  this.child
}) : assert(
       text != null || child != null || icon != null,
       'At least one of text, icon, and child must be non-null.'  // ✅ Descriptive!
     ),
     assert(
       text == null || child == null,
       'Cannot provide both text and child. Use one or the other.'  // ✅ Helpful!
     );
```

## The Challenge: Const Constructors

### Problem Encountered
A contributor tried to add error messages but hit this error:
```
Const constructor can't evaluate expression with string interpolation
```

### Why This Happens
Dart's const evaluation has restrictions on what can be used in const contexts. However, **plain string literals ARE allowed** in const assertions!

### The Solution
```dart
// ❌ WRONG - String interpolation not allowed
assert(condition, 'Value is $value')

// ✅ CORRECT - Plain strings ARE allowed
assert(condition, 'At least one parameter must be non-null')

// ✅ ALSO CORRECT - String concatenation works
assert(condition, 'Cannot provide both ' + 'text and child')
```

## Implementation Steps

### 1. Update Tab Constructor
File: `packages/flutter/lib/src/material/tabs.dart`

Replace assertions on lines 123-124 with:
```dart
assert(
  text != null || child != null || icon != null,
  'Tab must have at least one of text, icon, or child'
),
assert(
  text == null || child == null,
  'Tab cannot have both text and child properties. '
  'Please use either text or child, but not both.'
),
```

### 2. Add Tests
```dart
test('Tab throws descriptive error for both text and child', () {
  expect(
    () => Tab(text: 'Hello', child: Text('World')),
    throwsAssertionError,
  );

  // Verify error message is descriptive
  try {
    Tab(text: 'Hello', child: Text('World'));
  } on AssertionError catch (e) {
    expect(
      e.message,
      contains('cannot have both text and child'),
    );
  }
});

test('Tab throws descriptive error for missing content', () {
  expect(
    () => Tab(),
    throwsAssertionError,
  );

  try {
    Tab();
  } on AssertionError catch (e) {
    expect(
      e.message,
      contains('must have at least one'),
    );
  }
});
```

### 3. Update Documentation
Add examples showing proper usage:
```dart
/// Examples:
///
/// Valid uses:
/// ```dart
/// Tab(text: 'Home')
/// Tab(icon: Icon(Icons.home))
/// Tab(child: Text('Custom'))
/// Tab(text: 'Home', icon: Icon(Icons.home)) // text + icon OK
/// ```
///
/// Invalid uses:
/// ```dart
/// Tab() // Error: Must have text, icon, or child
/// Tab(text: 'Home', child: Text('Home')) // Error: Cannot have both
/// ```
```

## Examples of Good Error Messages

From Flutter framework:
```dart
// From RenderBox
assert(size.isFinite, 'RenderBox size is not finite');

// From Container
assert(
  margin == null || margin.isNonNegative,
  'Container margin must be non-negative'
);

// From Padding
assert(
  padding != null && padding.isNonNegative,
  'Padding must be non-null and non-negative'
);
```

## Benefits

### Before
```
Error: The assertion in this constant expression failed.
```
Developer has to:
1. Read Tab widget source code
2. Figure out which assertion failed
3. Understand the constraint

### After
```
Error: Tab cannot have both text and child properties.
Please use either text or child, but not both.
```
Developer:
1. Immediately understands the problem
2. Knows how to fix it
3. Continues development

## Testing Requirements

### Manual Testing
Create test cases for all invalid combinations:
```dart
Tab()                                    // Missing all
Tab(text: 'A', child: Text('B'))        // Both text and child
Tab(text: null, icon: null, child: null) // All null
```

### Automated Testing
- Unit tests for assertion messages
- Integration tests for error display
- Documentation tests for examples

## Labels
- `P3` - Nice to have
- `good first issue` - Approachable
- `a: error message` - Error message improvement
- `f: material design` - Material widgets
- `framework` - Framework issue
- `team-design` - Design team
- `triaged-design` - Already triaged

## Community Engagement
- **5 comments** discussing implementation
- Common pain point for new Flutter developers
- Related to overall DX improvement initiative

## Related Issues
- Similar issues for other widgets with unclear errors
- General framework error message improvements

## Const Evaluation Reference

### What Works in Const Assertions
✅ String literals
✅ String concatenation with `+`
✅ Boolean expressions
✅ Arithmetic with const values
✅ Conditional expressions `? :`

### What Doesn't Work
❌ String interpolation `$variable`
❌ Function calls (non-const)
❌ Mutable variables
❌ Non-const constructors

## Timeline
- **Reported:** November 2025
- **Contributor attempted fix:** Blocked by const evaluation misunderstanding
- **Status:** Awaiting updated implementation

## Success Criteria
1. Both assertions have clear error messages
2. Tests verify message content
3. Documentation includes examples
4. No const evaluation errors
5. Error messages follow Flutter conventions

## Links
- Issue: https://github.com/flutter/flutter/issues/178221
- File: tabs.dart:123-124
