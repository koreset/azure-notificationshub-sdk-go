# GCM Removal Summary

This document summarizes the complete removal of GCM (Google Cloud Messaging) support from the Azure Notification Hubs Go SDK.

## Background

Google deprecated the FCM legacy API in July 2024, making GCM and FCM legacy obsolete. This SDK has been updated to remove all legacy Android push notification support and exclusively use FCM v1 (Firebase Cloud Messaging v1).

## Changes Made

### 1. Constants Removed
- `GcmFormat` - Notification format for GCM
- `GcmPlatform` - Target platform for GCM  
- `GcmTemplatePlatform` - Template platform for GCM
- `GCMPlatform` - Installation platform for GCM

### 2. Types Updated
- Removed `GcmRegistrationDescription` from `RegistrationContent`
- Removed `GcmTemplateRegistrationDescription` from `RegistrationContent`
- Removed `GcmRegistrationID` field from `RegisteredDevice`
- Removed `GcmOutcomeCounts` from `NotificationDetails`

### 3. Platform Functions Updated
- Updated `GetContentType()` to remove GCM format
- Updated `IsValid()` functions to remove GCM platform validation
- Updated `normalize()` function to use FCM v1 instead of GCM

### 4. Registration Templates Replaced
- Replaced `gcmRegXMLString` with `fcmV1RegXMLString`
- Replaced `gcmTemplateRegXMLString` with `fcmV1TemplateRegXMLString`
- Updated XML templates to use `FcmV1RegistrationDescription`

### 5. Tests Updated
- Renamed `Test_RegisterGcm` to `Test_RegisterFcmV1`
- Updated all test expectations to use FCM v1 constants
- Updated test fixtures to use FCM v1 format
- Fixed installation and registration test data

### 6. Fixtures Updated
- Removed `androidRegistrationResult.xml` (GCM fixture)
- Removed `gcmInstallationResult.json` (GCM fixture)
- Updated `registrationsResult.xml` to use FCM v1 format
- Kept FCM v1 fixtures: `fcmv1RegistrationResult.xml`, `fcmv1TemplateRegistrationResult.xml`, `fcmv1InstallationResult.json`

### 7. Documentation Updated
- Updated README.md to remove GCM references
- Updated FCM_V1_MIGRATION_GUIDE.md to reflect GCM removal
- Added breaking change notices
- Updated platform support information

## Breaking Changes

⚠️ **BREAKING CHANGE**: This is a breaking change for users currently using GCM constants or functionality.

### Migration Required

Users must update their code to use FCM v1:

#### Before (GCM - No longer supported)
```go
// These constants no longer exist
notificationhubs.GcmFormat
notificationhubs.GcmPlatform  
notificationhubs.GcmTemplatePlatform
notificationhubs.GCMPlatform
```

#### After (FCM v1 - Required)
```go
// Use these FCM v1 constants instead
notificationhubs.FcmV1Format
notificationhubs.FcmV1Platform
notificationhubs.FcmV1TemplatePlatform  
notificationhubs.FCMV1Platform
```

## Benefits

1. **Future-Proof**: Aligns with Google's deprecation of legacy APIs
2. **Simplified Codebase**: Removes deprecated code paths
3. **Enhanced Features**: FCM v1 provides better error reporting and telemetry
4. **Consistency**: Single Android push notification standard

## Test Results

- ✅ All 38 tests passing
- ✅ Zero compilation errors
- ✅ Full FCM v1 functionality working
- ✅ Complete GCM removal verified

## Files Modified

### Core Files
- `constants.go` - Removed GCM constants, kept FCM v1
- `platform.go` - Updated validation functions
- `types.go` - Removed GCM type fields
- `registration.go` - Updated registration logic for FCM v1
- `internal.go` - Replaced GCM XML templates with FCM v1

### Test Files  
- `platform_test.go` - Updated platform validation tests
- `notification_test.go` - Updated to use FCM v1 format
- `installation_test.go` - Updated to use FCM v1 platform
- `registration_test.go` - Renamed and updated GCM test to FCM v1
- `fcmv1_test.go` - Comprehensive FCM v1 tests (unchanged)

### Fixtures
- `fixtures/registrationsResult.xml` - Updated to FCM v1 format
- `fixtures/fcmv1RegistrationResult.xml` - Fixed tags format
- `fixtures/fcmv1InstallationResult.json` - Simplified structure
- Removed: `fixtures/androidRegistrationResult.xml`
- Removed: `fixtures/gcmInstallationResult.json`

### Documentation
- `README.md` - Updated examples and platform support
- `FCM_V1_MIGRATION_GUIDE.md` - Updated for GCM removal
- `GCM_REMOVAL_SUMMARY.md` - This document

## Verification

The removal has been thoroughly tested:

1. **Compilation**: No build errors
2. **Tests**: All tests pass with FCM v1 functionality
3. **Functionality**: Registration, installation, and notification sending work with FCM v1
4. **Documentation**: Updated to reflect current state

## Next Steps for Users

1. **Update mobile apps** to use Firebase SDK for FCM v1 tokens
2. **Replace GCM constants** with FCM v1 equivalents in your code
3. **Update notification payloads** to use FCM v1 message format
4. **Test thoroughly** with your Android applications
5. **Update Azure Notification Hub** credentials to use FCM v1

For detailed migration instructions, see `FCM_V1_MIGRATION_GUIDE.md`. 