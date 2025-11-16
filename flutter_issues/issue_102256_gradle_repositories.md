# Issue #102256: Gradle Custom Repository Overrides

## Overview
- **Status:** Open
- **Priority:** P3
- **Created:** April 20, 2022
- **Type:** Enterprise Feature
- **Estimated Time:** 1-2 hours
- **Confidence:** 75%

## Problem Description
Organizations with restricted network access cannot use internal Maven repositories because Flutter's `flutter.gradle` hardcodes public repositories (Google, Maven Central).

### Use Case: Enterprise Environment
```
┌─────────────────┐
│  Developer PC   │
│                 │
│  ❌ No access  │
│  to internet    │
│                 │
│  ✅ Access to  │
│  internal repo  │
└─────────────────┘
         ↓
    Firewall
         ↓
┌─────────────────┐
│ Internal Maven  │
│   Repository    │
│  (Nexus/Artifac │
│   tory/JFrog)   │
└─────────────────┘
```

### Current Problem
File: `packages/flutter_tools/gradle/flutter.gradle` (lines 57-66)

```groovy
buildscript {
  repositories {
    google()        // ❌ Hardcoded!
    mavenCentral()  // ❌ Hardcoded!
  }
}
```

**Result:** Build fails because developer can't reach google.com or maven.org

## Real-World Scenarios

### Scenario 1: Corporate Firewall
- Company blocks external Maven repositories for security
- All dependencies mirrored to internal Nexus/Artifactory
- Flutter builds fail: "Could not resolve all artifacts"

### Scenario 2: Air-Gapped Environment
- High-security environment with no internet
- All dependencies pre-loaded to internal repository
- Flutter cannot build Android apps

### Scenario 3: Regional Access
- Some countries have restricted access to Maven Central
- Companies must use regional mirrors
- Flutter forces use of blocked repositories

## Proposed Solution

### Approach: Environment Variable Override

Similar to existing `FLUTTER_STORAGE_BASE_URL`, add `FLUTTER_GRADLE_REPOSITORY_URL`:

```bash
export FLUTTER_GRADLE_REPOSITORY_URL="https://nexus.company.internal/repository/maven-public/"
flutter build apk
```

### Implementation in flutter.gradle

```groovy
buildscript {
  def flutterGradleRepo = System.getenv('FLUTTER_GRADLE_REPOSITORY_URL')

  repositories {
    if (flutterGradleRepo) {
      // Custom repository takes priority
      maven {
        url flutterGradleRepo
        allowInsecureProtocol = flutterGradleRepo.startsWith('http://')
      }
    }
    // Fallback to public repositories
    google()
    mavenCentral()
  }
}
```

### Advanced: Multiple Repositories

```bash
# Comma-separated list
export FLUTTER_GRADLE_REPOSITORIES="https://nexus.company.internal,https://artifactory.company.internal"
```

```groovy
def gradleRepos = System.getenv('FLUTTER_GRADLE_REPOSITORIES')?.split(',')

repositories {
  if (gradleRepos) {
    gradleRepos.each { repoUrl ->
      maven {
        url repoUrl.trim()
      }
    }
  } else {
    google()
    mavenCentral()
  }
}
```

## Technical Details

### Files to Modify
1. **Primary:** `packages/flutter_tools/gradle/flutter.gradle`
2. **Secondary:** `packages/flutter_tools/gradle/src/main/groovy/flutter.groovy` (if exists)

### Locations in Code
```groovy
// flutter.gradle ~line 57
buildscript {
  repositories {
    // ADD CUSTOM REPOSITORY LOGIC HERE
  }
  dependencies {
    // Also check dependencies repositories
  }
}

// flutter.gradle ~line 100+
allprojects {
  repositories {
    // ALSO ADD CUSTOM REPOSITORY LOGIC HERE
  }
}
```

### Configuration Precedence
```
1. Environment variable (highest priority)
   ↓
2. gradle.properties setting
   ↓
3. Default public repositories (fallback)
```

## Implementation Steps

### Phase 1: Basic Support (30 min)
```groovy
def customRepo = System.getenv('FLUTTER_GRADLE_REPOSITORY_URL')

repositories {
  if (customRepo) {
    maven { url customRepo }
  }
  google()
  mavenCentral()
}
```

### Phase 2: Multiple Repos (15 min)
```groovy
def customRepos = System.getenv('FLUTTER_GRADLE_REPOSITORIES')?.split(',')

repositories {
  customRepos?.each { repo ->
    maven { url repo.trim() }
  }
  google()
  mavenCentral()
}
```

### Phase 3: gradle.properties (30 min)
```groovy
// Read from gradle.properties
def customRepo = project.findProperty('flutter.gradle.repository')

if (customRepo) {
  maven { url customRepo }
}
```

### Phase 4: Exclusive Mode (15 min)
```bash
# Only use custom repos, skip public
export FLUTTER_GRADLE_REPOSITORIES_EXCLUSIVE=true
```

```groovy
def exclusiveMode = System.getenv('FLUTTER_GRADLE_REPOSITORIES_EXCLUSIVE')

repositories {
  if (customRepos) {
    customRepos.each { maven { url it } }
    if (!exclusiveMode) {
      google()
      mavenCentral()
    }
  } else {
    google()
    mavenCentral()
  }
}
```

## Testing Requirements

### Test Case 1: Custom Repository Works
```bash
export FLUTTER_GRADLE_REPOSITORY_URL="https://repo.maven.apache.org/maven2/"
flutter build apk
# Should download from custom URL
```

### Test Case 2: Fallback Works
```bash
unset FLUTTER_GRADLE_REPOSITORY_URL
flutter build apk
# Should use google() and mavenCentral()
```

### Test Case 3: Multiple Repositories
```bash
export FLUTTER_GRADLE_REPOSITORIES="https://repo1.com,https://repo2.com"
flutter build apk
# Should try both repositories
```

### Test Case 4: Invalid Repository Fails Gracefully
```bash
export FLUTTER_GRADLE_REPOSITORY_URL="https://invalid-repo.com"
flutter build apk
# Should fall back to public repos with warning
```

## Documentation Requirements

### 1. Update flutter.dev
Add to "Building for Android" docs:

```markdown
### Using Custom Maven Repositories

For enterprise environments with restricted internet access:

```bash
export FLUTTER_GRADLE_REPOSITORY_URL="https://nexus.company.com/repository/maven-public/"
flutter build apk
```

### 2. Update flutter.gradle Comments
```groovy
// Custom repository support via environment variables:
// - FLUTTER_GRADLE_REPOSITORY_URL: Single custom repository
// - FLUTTER_GRADLE_REPOSITORIES: Comma-separated list
// - FLUTTER_GRADLE_REPOSITORIES_EXCLUSIVE: Skip public repos if true
```

### 3. Add Example to FAQ
"How do I use Flutter with an internal Maven repository?"

## Related Patterns

### Existing Similar Features
```bash
# Flutter already supports storage base URL override
FLUTTER_STORAGE_BASE_URL=https://custom.storage.com

# Pub supports custom package server
PUB_HOSTED_URL=https://custom.pub.com
```

### Gradle Best Practices
```groovy
// Many projects support repository configuration
repositories {
  maven { url project.findProperty('customMavenUrl') ?: 'https://default.com' }
}
```

## Community Discussion

### From Bill-Shaffer (Reporter)
> "Our organization recently made a change restricting access to public maven repositories... This has prevented us from being able to build Flutter apps."

### Transitive Dependencies Challenge
Community noted that pub.dev packages may also hardcode repositories. This issue focuses on Flutter's own gradle files.

## Labels
- `P3` - Nice to have
- `c: new feature` - New feature
- `c: proposal` - Proposal
- `good first issue` - Approachable
- `platform-android` - Android specific
- `t: gradle` - Gradle build system
- `team-android` - Android team
- `tool` - Flutter tools
- `triaged-android` - Triaged

## Impact
- **7 👍 reactions**
- **5 comments** from affected organizations
- Critical for enterprise adoption
- Blocks Flutter use in secure environments

## Breaking Changes
**None** - Backward compatible, opt-in feature.

## Performance Impact
**None** - Only adds conditional repository check.

## Security Considerations
- Allow HTTP for internal networks: `allowInsecureProtocol`
- Validate repository URLs
- Document security implications

## Timeline
- **Reported:** April 20, 2022
- **Status:** Open for 2.5+ years
- **Community demand:** Consistent

## Success Criteria
1. Custom repository URL working
2. Fallback to public repos when no custom URL
3. Multiple repositories supported
4. Documentation updated
5. Tests passing
6. No breaking changes for existing users

## Links
- Issue: https://github.com/flutter/flutter/issues/102256
- File: flutter.gradle:57-66
- Related: FLUTTER_STORAGE_BASE_URL implementation
