# FCM v1 Migration Guide

This guide helps you migrate to FCM v1 (Firebase Cloud Messaging v1) in your Azure Notification Hubs Go SDK implementation.

## Overview

Google deprecated the FCM legacy API in July 2024. This SDK has removed all legacy GCM support and now exclusively uses FCM v1 for Android push notifications, ensuring future compatibility and enhanced features.

## Key Differences

| Aspect | Legacy Android Push | FCM v1 |
|--------|---------------------|--------|
| **API Endpoint** | Various legacy endpoints | `fcm.googleapis.com` |
| **Authentication** | Server Key | Service Account JSON |
| **Message Format** | Direct JSON payload | Wrapped in `message` object |
| **Platform Constant** | Legacy constants | `FcmV1Format`, `FcmV1Platform` |
| **Azure NH Platform** | Legacy platforms | `fcmv1` |

## Migration Steps

### Step 1: Update Your Mobile App

Before updating your backend, ensure your mobile app is using the Firebase SDK:

#### Android (Kotlin/Java)
```kotlin
// Add to build.gradle (app level)
implementation 'com.google.firebase:firebase-messaging:23.0.0'

// Get FCM v1 token
FirebaseMessaging.getInstance().token.addOnCompleteListener { task ->
    if (!task.isSuccessful) {
        Log.w(TAG, "Fetching FCM registration token failed", task.exception)
        return@addOnCompleteListener
    }

    // Get new FCM v1 token
    val token = task.result
    Log.d(TAG, "FCM Registration Token: $token")
    
    // Send token to your server
    sendTokenToServer(token)
}
```

#### Flutter
```dart
// Add to pubspec.yaml
firebase_messaging: ^14.0.0

// Get FCM v1 token
String? token = await FirebaseMessaging.instance.getToken();
print('FCM v1 Token: $token');
```

### Step 2: Update Azure Notification Hub Credentials

In the Azure Portal:

1. Go to your Notification Hub
2. Navigate to **Settings** > **Google (FCM v1)**
3. Upload your Firebase service account JSON file
4. Save the configuration

**Important**: Only FCM v1 credentials are supported in this SDK version.

### Step 3: Update Your Go Code

#### FCM v1 Implementation
```go
// New FCM v1 registration
reg := notificationhubs.NewRegistration(
    "fcmv1_token_here",
    nil,
    "registration-id",
    "tag1,tag2",
    notificationhubs.FcmV1Format,
)

// New FCM v1 template - note the "message" wrapper
template := `{
    "message": {
        "notification": {
            "title": "$(title)",
            "body": "$(body)"
        },
        "data": {
            "customKey": "$(customValue)"
        }
    }
}`

templateReg := notificationhubs.NewTemplateRegistration(
    "fcmv1_token_here",
    nil,
    "template-registration-id",
    "tag1,tag2", 
    notificationhubs.FcmV1TemplatePlatform,
    template,
)

// New FCM v1 notification - note the "message" wrapper
payload := []byte(`{
    "message": {
        "notification": {
            "title": "Hello",
            "body": "World"
        },
        "data": {
            "key1": "value1"
        }
    }
}`)
notification, _ := notificationhubs.NewNotification(notificationhubs.FcmV1Format, payload)
```

### Step 4: Update Installations
```go
installation := &notificationhubs.Installation{
    InstallationID: "installation-123", 
    Platform:       notificationhubs.FCMV1Platform,
    PushChannel:    "fcmv1_token_here",
    Tags:           []string{"tag1", "tag2"},
    Templates: map[string]notificationhubs.InstallationTemplate{
        "default": {
            Body: `{"message":{"notification":{"title":"$(title)","body":"$(body)"}}}`,
        },
    },
}
```

## Message Format Changes

### FCM v1 Format
```json
{
    "message": {
        "notification": {
            "title": "News Update", 
            "body": "Check out the latest news"
        },
        "data": {
            "articleId": "12345"
        },
        "android": {
            "priority": "high",
            "notification": {
                "click_action": "FLUTTER_NOTIFICATION_CLICK"
            }
        }
    }
}
```

## Constants Reference

### Notification Formats
- **FCM v1**: `notificationhubs.FcmV1Format` ("fcmv1")

### Target Platforms  
- **FCM v1**: `notificationhubs.FcmV1Platform` ("fcmv1") 
- **FCM v1 Template**: `notificationhubs.FcmV1TemplatePlatform` ("fcmv1template")

### Installation Platforms
- **FCM v1**: `notificationhubs.FCMV1Platform` ("fcmv1")

## Migration Strategy

### Migration Approach

1. **Update mobile apps** to use Firebase SDK
2. **Update Azure Notification Hub** credentials to FCM v1
3. **Replace all registrations** with FCM v1 registrations
4. **Update all notification sending code** to use FCM v1 format
5. **Test thoroughly** on various Android versions and devices

**Note**: This SDK version only supports FCM v1, so migration is required for continued Android push notification support.

## Testing Your Migration

### Test FCM v1 Registration
```go
func testFcmV1Registration() {
    hub := notificationhubs.NewNotificationHub(connectionString, hubName)
    
    reg := notificationhubs.NewRegistration(
        "test_fcmv1_token",
        nil,
        "test-fcmv1-reg",
        "test",
        notificationhubs.FcmV1Format,
    )
    
    _, err := hub.Register(context.Background(), *reg)
    if err != nil {
        log.Printf("FCM v1 registration failed: %v", err)
    } else {
        log.Println("FCM v1 registration successful!")
    }
}
```

### Test FCM v1 Notification
```go
func testFcmV1Notification() {
    hub := notificationhubs.NewNotificationHub(connectionString, hubName)
    
    payload := []byte(`{
        "message": {
            "notification": {
                "title": "Test FCM v1",
                "body": "Migration test successful"
            }
        }
    }`)
    
    notification, _ := notificationhubs.NewNotification(notificationhubs.FcmV1Format, payload)
    tags := "test"
    
    _, telemetry, err := hub.Send(context.Background(), notification, &tags)
    if err != nil {
        log.Printf("FCM v1 send failed: %v", err)
    } else {
        log.Printf("FCM v1 send successful! ID: %s", telemetry.NotificationMessageID)
    }
}
```

## Common Issues and Solutions

### Issue 1: Invalid Message Format
**Error**: `Invalid notification format`
**Solution**: Ensure FCM v1 payloads are wrapped in a `message` object.

### Issue 2: Wrong Platform Type
**Error**: `Platform mismatch` 
**Solution**: Use `FcmV1Platform` for FCM v1 registrations.

### Issue 3: Token Format Issues
**Error**: `Invalid registration token`
**Solution**: Ensure you're using FCM v1 tokens from the Firebase SDK.

### Issue 4: Template Validation Errors
**Error**: `Template validation failed`
**Solution**: Update templates to use FCM v1 message structure with `message` wrapper.

## Verification

### Check Registration Type
```go
// In registration results, look for:
// FcmV1RegistrationDescription for FCM v1 registrations
```

### Monitor Telemetry
```go
// FCM v1 notifications will show in telemetry as:
// - FcmV1OutcomeCounts for FCM v1 delivery statistics
```

### Azure Portal Verification
1. Go to Azure Portal > Notification Hubs > Your Hub
2. Check **Metrics** for FCM v1 vs GCM legacy sends
3. Verify **Google (FCM v1)** credentials are configured

## Best Practices

1. **Complete Migration**: Ensure all Android devices use FCM v1 tokens
2. **Monitor Metrics**: Watch delivery rates and error rates
3. **Test Thoroughly**: Test on various Android versions and devices
4. **Update Documentation**: Keep your team informed of the changes
5. **Backup Data**: Export existing registrations before migration
6. **Validate Credentials**: Ensure FCM v1 service account credentials are properly configured

## Support and Resources

- [Azure Notification Hubs FCM v1 Documentation](https://docs.microsoft.com/en-us/azure/notification-hubs/notification-hubs-gcm-to-fcm)
- [Firebase Cloud Messaging Documentation](https://firebase.google.com/docs/cloud-messaging)
- [FCM v1 API Reference](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages)

For issues specific to this Go SDK, please check the [project issues](https://github.com/koreset/azure-notificationhubs-go/issues) or create a new one. 