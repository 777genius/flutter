# Issue #135525: io.worker Thread Pool Sizing

## Overview
- **Status:** Open
- **Priority:** P2
- **Created:** September 26, 2023
- **Type:** Performance Optimization
- **Estimated Time:** 30-45 minutes
- **Confidence:** 85%

## Problem Description
The `io.worker` thread pool creates one thread per CPU hardware thread, leading to excessive thread creation on high-core-count machines.

### Current Behavior
- 16-core machine (32 hardware threads) → **32 io.worker threads** ❌
- 24-core machine (48 hardware threads) → **48 io.worker threads** ❌

### Expected Behavior
Thread pool should cap at a reasonable number (e.g., ~4-8 threads maximum)

## Real-World Impact

### GDB Output Example
```
Thread 17 (Thread ...): io.worker
Thread 18 (Thread ...): io.worker
Thread 19 (Thread ...): io.worker
...
Thread 48 (Thread ...): io.worker
```
**Result:** 32 threads on a 16-core system!

### Why This is Problematic
1. **Excessive context switching** - Too many threads competing for CPU
2. **Memory overhead** - Each thread has stack space (~1-8 MB)
3. **Resource contention** - Threads fighting for locks/resources
4. **No performance gain** - I/O operations don't benefit from this many threads

## Technical Details

### Current Implementation
```cpp
// Uses std::thread::hardware_concurrency() directly
const size_t num_threads = std::thread::hardware_concurrency();
for (size_t i = 0; i < num_threads; ++i) {
  threads.emplace_back(/* ... */);
}
```

### Proposed Fix
From issue author (chinmaygarde):
> "This could be 2 to half of available device concurrency capped to 4."

### Recommended Implementation
```cpp
size_t GetOptimalWorkerThreadCount() {
  const size_t hardware_threads = std::thread::hardware_concurrency();

  // Handle edge case where hardware_concurrency returns 0
  if (hardware_threads == 0) {
    return 2;  // Fallback to 2 threads
  }

  // Calculate: half of available threads, capped at 4, minimum 2
  size_t optimal = hardware_threads / 2;
  optimal = std::max(optimal, size_t(2));    // At least 2
  optimal = std::min(optimal, size_t(4));    // At most 4

  return optimal;
}
```

### Expected Results
| Machine | Hardware Threads | Old | New |
|---------|-----------------|-----|-----|
| 4-core  | 8               | 8   | 4   |
| 8-core  | 16              | 16  | 4   |
| 16-core | 32              | 32  | 4   |
| 24-core | 48              | 48  | 4   |

## Edge Cases to Handle

### 1. Hypervisor Environments
`hardware_concurrency()` may return:
- **0** - No information available
- **Odd values** - Virtual CPU allocation

### 2. Single-Core Systems
Ensure at least 2 threads for I/O operations

### 3. Performance Testing
Need benchmarks to verify 4 is optimal cap

## File Locations

### Primary File
Engine's thread pool implementation (exact location TBD - likely in `fml` or embedder code)

### Files to Review
- `flutter/fml/thread.h`
- `flutter/fml/thread.cc`
- `flutter/shell/common/io_manager.cc`

## Implementation Steps

1. **Create helper function** `GetOptimalWorkerThreadCount()`
2. **Replace** direct `hardware_concurrency()` calls
3. **Add unit tests** for edge cases
4. **Benchmark** on various platforms
5. **Document** the threading strategy

## Testing Requirements

### Unit Tests
```cpp
TEST(ThreadPoolTest, OptimalThreadCount) {
  // Mock hardware_concurrency to return various values
  EXPECT_EQ(GetOptimalWorkerThreadCount(0), 2);   // Fallback
  EXPECT_EQ(GetOptimalWorkerThreadCount(4), 2);   // 4/2 = 2
  EXPECT_EQ(GetOptimalWorkerThreadCount(16), 4);  // 16/2 = 8, capped at 4
  EXPECT_EQ(GetOptimalWorkerThreadCount(48), 4);  // 48/2 = 24, capped at 4
}
```

### Performance Tests
- Benchmark I/O operations with different thread counts
- Verify no regression on low-core machines
- Test on high-core machines (16+, 24+, 32+ cores)

### Platform Tests
- Linux (primary reproduction platform)
- Windows
- macOS
- iOS
- Android

## Labels
- `engine` - Flutter engine
- `e: embedder` - Embedder API
- `platform-linux` - Linux platform
- `good first issue` - Approachable for new contributors
- `P2` - Important priority
- `team-engine` - Owned by Engine team
- `triaged-engine` - Already triaged

## Community Discussion

### Comment from Joel Winarske (Nov 29, 2023)
Proposed helper function handling:
- Scaled values from hardware_concurrency
- Minimum requirements
- Edge cases (0 return, odd values)

### Key Insight
> "If hardware_concurrency returns an even number, you may get a scaled value"

Important for hypervisor environments.

## Performance Implications

### Before
- 32 threads on 16-core machine
- High context switching
- ~32-256 MB memory for thread stacks

### After
- 4 threads regardless of core count
- Reduced context switching
- ~4-32 MB memory for thread stacks
- More predictable performance

## Reproduction
```bash
# On Linux machine with 16+ cores
flutter create test_app
cd test_app
flutter run -d linux
# Attach GDB and check thread count
gdb --pid=$(pgrep flutter)
(gdb) info threads
# See 30+ io.worker threads
```

## Related Work
- Thread pool best practices in Chromium
- Go language GOMAXPROCS (uses P = num CPUs)
- Java's ForkJoinPool (defaults to CPU count - 1)

## Timeline
- **Reported:** September 26, 2023
- **Last Activity:** April 29, 2025
- **Community Engagement:** 5 comments

## Links
- Issue: https://github.com/flutter/flutter/issues/135525
- Engine repo: https://github.com/flutter/engine
