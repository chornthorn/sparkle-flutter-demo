# App Update Rules Implementation Guide

This document explains how to implement various app update strategies using the appcast XML format with the inapp_update package.

## Update Logic Rules

The update decision logic follows these rules, evaluated in order of priority:

### Rule 1: Critical Minimum Version
**Condition:** `app_version < min_supported_version`  
**Action:** Forced Update to latest_version  
**Purpose:** Ensures no user operates on a critically outdated or unsupported version.

### Rule 2A: API Forced Update
**Condition:** `(app_version >= min_supported_version) AND (update_type == "forced") AND (app_version < latest_version)`  
**Action:** Forced Update to latest_version  
**Purpose:** Allows the backend to mandate an update even if the user's current version meets the minimum support criteria.

### Rule 2B: API Optional Update
**Condition:** `(app_version >= min_supported_version) AND (update_type == "optional") AND (app_version < latest_version)`  
**Action:** Optional Update to latest_version  
**Purpose:** Offers users new features or non-critical improvements without forcing an immediate update.

### Rule 3: No Update Needed
**Condition:** `app_version >= latest_version`  
**Action:** No Update Dialog  
**Purpose:** User is already on the latest version or a newer one.

## Version Requirements Clarification

There are two distinct version requirements to consider:

1. **App Version Requirements**:
   - `min_supported_version`: Minimum supported application version (e.g., 2.5.0)
   - `latest_version`: The newest available application version (e.g., 3.0.0)
   - These determine whether an update is needed based on the rules above

2. **OS Version Requirements**:
   - Android: Minimum SDK version 29 (Android 10)
   - iOS: Minimum version 17.0
   - These ensure the app only runs on supported operating systems

## XML Implementation Examples

We've provided four example XML files that demonstrate how to implement each rule:

1. `rule1_critical_min_version.xml` - Forces updates for versions below minimum supported version
2. `rule2a_api_forced_update.xml` - Forces updates when a version is supported but needs a critical update
3. `rule2b_api_optional_update.xml` - Offers optional updates for supported versions
4. `rule3_no_update_needed.xml` - Shows how to handle cases where no update is needed

## Key XML Elements for Update Rules

### 1. Version Information
```xml
<enclosure 
  url="https://example.com/downloads/myapp-3.0.0.apk" 
  sparkle:version="3.0.0" 
  sparkle:os="android" 
/>
```

### 2. Critical Update Flag (Forces Update)
```xml
<sparkle:tags>
  <sparkle:criticalUpdate />
</sparkle:tags>
```

### 3. iOS Minimum OS Version
```xml
<sparkle:minimumSystemVersion>17.0.0</sparkle:minimumSystemVersion>
```

## Implementation Notes

1. **App Version Handling**:
   - The app version minimum (2.5.0) should be checked in your app code
   - The Sparkle framework doesn't have a standard tag for minimum app version
   - Your app should compare its version against backend-defined minimums

2. **OS Version Handling**:
   - For iOS, use the `sparkle:minimumSystemVersion` tag
   - For Android, check SDK version in your app code (minimum 29)

3. **Update Types**:
   - Forced updates use the `sparkle:criticalUpdate` tag
   - Optional updates omit this tag, allowing "Later" option
   - The Sparkle framework doesn't directly support "updateType" as a property

4. **Testing Update Scenarios**:
   - Use the `debugDisplayAlways` option in `InAppUpdateConfig` to test update dialogs
   - Test each update rule separately to ensure correct behavior

## Client Implementation

Use the package in your Flutter app as follows:

```dart
// Initialize with your appcast URL
final config = InAppUpdateConfig(
  appcastURL: 'https://example.com/appcast.xml',
);

final updateService = InAppUpdateService(config: config);
await updateService.initialize();

// Then use one of the widgets to display updates
return InAppUpdateAlert(
  config: config,
  child: YourAppWidget(),
);
```

The package will handle version comparison and apply the appropriate update rule based on the user's current version and the appcast content. 
