---
title: "[en] Complete Guide to iOS App Versioning: From Basics to Advanced Strategies"
date: 2024-11-13T00:00:00+00:00
author: Hieu Xuan Leu
layout: post
permalink: /ios-versioning-complete-guide/
categories: git
tags: [git, git commands]
# image: ../assets/images/map_in_go.png
published: true
---

# Complete Guide to iOS App Versioning: From Basics to Advanced Strategies

## What is App Versioning?

App versioning in iOS consists of two key components that work together to identify your app releases:

1. Version Number (Marketing Version)
2. Build Number (Internal Version)

Let's dive deep into each component and understand how to use them effectively.

## Version Number (Marketing Version)

### Understanding X.Y.Z Format

The Version Number follows Semantic Versioning (SemVer) using the format: `X.Y.Z`

```
Example: 2.1.3
Where:
X = Major version (2)
Y = Minor version (1)
Z = Patch version (3)
```

### When to Change Each Number

#### X (Major Version)
- Increment when making incompatible changes
- Examples:
  ```
  1.0.0 → 2.0.0: Complete UI redesign
  2.0.0 → 3.0.0: New architecture implementation
  3.0.0 → 4.0.0: Dropping support for older iOS versions
  ```

#### Y (Minor Version)
- Increment when adding features backward-compatibly
- Examples:
  ```
  1.0.0 → 1.1.0: Added dark mode
  1.1.0 → 1.2.0: Added push notifications
  1.2.0 → 1.3.0: Added new payment method
  ```

#### Z (Patch Version)
- Increment for bug fixes and minor updates
- Examples:
  ```
  1.1.0 → 1.1.1: Fixed crash in login screen
  1.1.1 → 1.1.2: Corrected text formatting
  1.1.2 → 1.1.3: Fixed memory leak
  ```

## Build Number

### Purpose
- Unique identifier for each App Store submission
- Must always increase
- Not visible to users by default

### Common Formats

1. **Simple Sequential Numbers**
```
1, 2, 3, 4, 5...
Pros: Simple to track
Cons: No context about version
```

2. **Date-Based Format**
```
YYYYMMDD: 20240113
YYYYMMDDNN: 2024011301 (multiple builds per day)
Pros: Provides timestamp context
Cons: Large numbers
```

3. **Version-Based Format**
```
Major.Minor.Patch.Build
1.0.0.1
1.0.0.2
Pros: Clear connection to version
Cons: More complex to manage
```

## Implementation in Xcode

### Setting Version and Build Numbers

#### Method 1: Using Xcode UI
1. Open your project in Xcode
2. Select your target
3. Go to the "General" tab
4. Under "Identity" section:
```
Version: 1.0.0
Build: 1
```

#### Method 2: Using Info.plist
```xml
<key>CFBundleShortVersionString</key>
<string>1.0.0</string>
<key>CFBundleVersion</key>
<string>1</string>
```

#### Method 3: Using Build Script
```bash
#!/bin/bash

# Increment build number
buildNumber=$(/usr/libexec/PlistBuddy -c "Print CFBundleVersion" "${PROJECT_DIR}/${INFOPLIST_FILE}")
buildNumber=$(($buildNumber + 1))
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $buildNumber" "${PROJECT_DIR}/${INFOPLIST_FILE}"
```

## Real-World Version Management

### Development Cycle Example

```
Initial Development:
Version: 1.0.0 (Build 1)
- First submission to App Store

Bug Fix:
Version: 1.0.1 (Build 2)
- Fixed critical crash

Feature Update:
Version: 1.1.0 (Build 3)
- Added new feature

Major Update:
Version: 2.0.0 (Build 4)
- Complete redesign
```

### TestFlight Versioning

```
Production App Store:
Version: 1.0.0 (Build 1)

TestFlight Beta:
Version: 1.1.0-beta.1 (Build 100)
Version: 1.1.0-beta.2 (Build 101)

Internal Testing:
Version: 1.1.0-alpha.1 (Build 500)
```

## Version Display in Your App

### Swift Code for Version Display

```swift
class VersionManager {
    static let shared = VersionManager()
    
    var marketingVersion: String {
        Bundle.main.object(forInfoDictionaryKey: "CFBundleShortVersionString") as? String ?? "Unknown"
    }
    
    var buildNumber: String {
        Bundle.main.object(forInfoDictionaryKey: "CFBundleVersion") as? String ?? "Unknown"
    }
    
    var fullVersion: String {
        "\(marketingVersion) (\(buildNumber))"
    }
}

// Usage in UI
let versionLabel = UILabel()
versionLabel.text = "Version \(VersionManager.shared.fullVersion)"
```

## Best Practices

### DO's
1. Always increment build numbers
2. Use semantic versioning consistently
3. Document version changes
4. Plan version numbers ahead
5. Include version number in bug reports

### DON'Ts
1. Reuse version numbers
2. Skip version numbers
3. Decrease version numbers
4. Use complex versioning schemes
5. Forget to update both version and build numbers

## Version Control Integration

### Git Tags for Versions
```bash
# Tag release versions
git tag -a v1.0.0 -m "Version 1.0.0 release"
git push origin v1.0.0

# Tag beta versions
git tag -a v1.1.0-beta.1 -m "Beta 1 for version 1.1.0"
```

## Common Scenarios and Solutions

### Scenario 1: App Store Rejection
```
Initial submission:
Version: 1.0.0 (Build 1)
Status: Rejected

Fixed submission:
Version: 1.0.0 (Build 2)
Status: Approved
```

### Scenario 2: Multiple TestFlight Builds
```
TestFlight builds:
Version: 1.1.0-beta.1 (Build 100)
Version: 1.1.0-beta.2 (Build 101)
Version: 1.1.0-beta.3 (Build 102)

Final App Store release:
Version: 1.1.0 (Build 103)
```

## Versioning for Multiple Targets

### Example Structure
```
Main App:        1.0.0 (1)
Watch App:       1.0.0 (1)
Widget Ext:      1.0.0 (1)
```

## Automated Version Management

### Using fastlane
```ruby
lane :increment_version do |options|
  increment_version_number(
    version_number: options[:version]
  )
  increment_build_number
end
```

## Testing Version Information

### Unit Test Example
```swift
func testVersionFormat() {
    let version = VersionManager.shared.marketingVersion
    XCTAssertTrue(version.matches("^\\d+\\.\\d+\\.\\d+$"))
}
```

## Conclusion

Proper version management is crucial for successful iOS app development and maintenance. By following semantic versioning principles and maintaining consistent build numbers, you can:

1. Track app evolution effectively
2. Manage App Store submissions smoothly
3. Handle beta testing efficiently
4. Maintain clear communication with users

Remember: Version numbers tell your app's story to users, while build numbers keep your submissions organized.

## Additional Resources

- [App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)
- [TestFlight Beta Testing](https://developer.apple.com/testflight/)
- [Semantic Versioning](https://semver.org/)
