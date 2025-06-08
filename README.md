# Azure Notification Hubs for Go(lang)

This library provides a Go module for Microsoft Azure Notification Hubs.

Originally a fork from [Gozure](https://github.com/onefootball/gozure) with patches
from [Martin Etnestad](https://github.com/gnawybol) @ [vippsas](https://github.com/vippsas/gozure).

Now maintained and packaged by [Daresay AB](https://daresay.co), [@daresaydigital](https://github.com/daresaydigital).

Basically a wrapper for this [Rest API](https://docs.microsoft.com/en-us/rest/api/notificationhubs/rest-api-methods)

[![Build Status](https://travis-ci.org/koreset/azure-notificationhubs-go.svg?branch=master)](https://travis-ci.org/koreset/azure-notificationhubs-go)
[![Go](https://github.com/koreset/azure-notificationhubs-go/workflows/Go/badge.svg?branch=master)](https://github.com/koreset/azure-notificationhubs-go/actions)

## Installing

Using go get

```sh
go get github.com/koreset/azure-notificationhubs-go
```

## External dependencies

No external dependencies

## Registering device

```go
package main

import (
  "context"
  "strings"
  "github.com/koreset/azure-notificationhubs-go"
)

func main() {
  var (
    hub      = notificationhubs.NewNotificationHub("YOUR_DefaultFullSharedAccessConnectionString", "YOUR_HubPath")
    template = `{
    "aps":{
      "alert":{
        "title":"$(title)",
        "body":"$(body)",
      },
      "badge":"#(badge)",
      "topic":"co.daresay.app",
      "content-available": 1
    },
    "name1":"$(value1)",
    "name2":"$(value2)"
  }`
  )

  template = strings.ReplaceAll(template, "\n", "")
  template = strings.ReplaceAll(template, "\t", "")

  reg := notificationhubs.NewTemplateRegistration(
    "ABC123",                       // The token from Apple or Google
    nil,                            // Expiration time, probably endless
    "ZXCVQWE",                      // Registration id, if you want to update an existing registration
    "tag1,tag2",                    // Tags that matches this device
    notificationhubs.ApplePlatform, // or FcmV1Platform for Android
    template,                       // The template. Use "$(name)" for strings and "#(name)" for numbers
  )

  // or hub.NewRegistration( ... ) without template

  hub.RegisterWithTemplate(context.TODO(), *reg)
  // or if no template:
  hub.Register(context.TODO(), *reg)
}
```

## Sending notification

```go
package main

import (
  "context"
  "fmt"
  "github.com/koreset/azure-notificationhubs-go"
)

func main() {
  var (
    hub     = notificationhubs.NewNotificationHub("YOUR_DefaultFullSharedAccessConnectionString", "YOUR_HubPath")
    payload = []byte(`{"title": "Hello Hub!"}`)
    n, _    = notificationhubs.NewNotification(notificationhubs.Template, payload)
  )

  // Broadcast push
  b, _, err := hub.Send(context.TODO(), n, nil)
  if err != nil {
    panic(err)
  }

  fmt.Println("Message successfully created:", string(b))

  // Tag category push
  tags := "tag1 || tag2"
  b, _, err = hub.Send(context.TODO(), n, &tags)
  if err != nil {
    panic(err)
  }

  fmt.Println("Message successfully created:", string(b))
}
```

## FCM v1 Support

This library supports FCM v1 (Firebase Cloud Messaging v1), which is the current standard for Android push notifications. FCM legacy API was deprecated in July 2024.

### Why FCM v1?

FCM v1 (Firebase Cloud Messaging v1) is the current standard for Android push notifications. Google deprecated the FCM legacy API in July 2024, making FCM v1 the only supported option for new and existing Android applications.

### FCM v1 Registration Example

```go
package main

import (
  "context"
  "strings"
  "github.com/koreset/azure-notificationhubs-go"
)

func main() {
  var (
    hub      = notificationhubs.NewNotificationHub("YOUR_DefaultFullSharedAccessConnectionString", "YOUR_HubPath")
    // FCM v1 template format
    template = `{
      "message": {
        "notification": {
          "title": "$(title)",
          "body": "$(body)"
        },
        "data": {
          "key1": "$(value1)",
          "key2": "$(value2)"
        }
      }
    }`
  )

  template = strings.ReplaceAll(template, "\n", "")
  template = strings.ReplaceAll(template, "\t", "")

  reg := notificationhubs.NewTemplateRegistration(
    "YOUR_FCM_V1_TOKEN",              // FCM v1 token from Firebase SDK
    nil,                              // Expiration time
    "YOUR_REGISTRATION_ID",           // Registration ID
    "tag1,tag2",                      // Tags
    notificationhubs.FcmV1TemplatePlatform, // FCM v1 template platform
    template,                         // FCM v1 template format
  )

  hub.RegisterWithTemplate(context.TODO(), *reg)
}
```

### FCM v1 Installation Example

```go
package main

import (
  "context"
  "github.com/koreset/azure-notificationhubs-go"
)

func main() {
  hub := notificationhubs.NewNotificationHub("YOUR_ConnectionString", "YOUR_HubPath")
  
  installation := notificationhubs.Installation{
    InstallationID: "fcmv1-installation-123",
    Platform:       notificationhubs.FCMV1Platform,
    PushChannel:    "YOUR_FCM_V1_TOKEN",
    Tags:           []string{"tag1", "tag2"},
    Templates: map[string]notificationhubs.InstallationTemplate{
      "template1": {
        Body: `{"message":{"notification":{"title":"$(title)","body":"$(message)"}}}`,
        Tags: []string{"templateTag1"},
      },
    },
  }

  hub.CreateOrUpdateInstallation(context.TODO(), &installation)
}
```

### Sending FCM v1 Notifications

```go
package main

import (
  "context"
  "github.com/koreset/azure-notificationhubs-go"
)

func main() {
  hub := notificationhubs.NewNotificationHub("YOUR_ConnectionString", "YOUR_HubPath")
  
  // FCM v1 notification payload
  payload := []byte(`{
    "message": {
      "notification": {
        "title": "Hello FCM v1!",
        "body": "This is an FCM v1 notification"
      },
      "data": {
        "key1": "value1",
        "key2": "value2"
      }
    }
  }`)

  notification, _ := notificationhubs.NewNotification(notificationhubs.FcmV1Format, payload)
  
  // Send to all FCM v1 devices
  _, _, err := hub.Send(context.TODO(), notification, nil)
  if err != nil {
    panic(err)
  }
}
```

### Platform Constants

The library provides these FCM v1 constants:

- `notificationhubs.FcmV1Format` - For notification format
- `notificationhubs.FcmV1Platform` - For target platform
- `notificationhubs.FcmV1TemplatePlatform` - For template target platform  
- `notificationhubs.FCMV1Platform` - For installation platform

### Migration from Legacy Android Push

If you're migrating from older Android push implementations:

1. **Update your mobile app** to use the Firebase SDK and obtain FCM v1 tokens
2. **Create new registrations** using `FcmV1Platform` or `FcmV1TemplatePlatform`
3. **Update notification payloads** to use FCM v1 format (wrapped in `message` object)
4. **Update installations** to use `FCMV1Platform`

**Note**: This SDK only supports FCM v1 for Android. Legacy GCM support has been removed as it's deprecated by Google.

## Tag expressions

Read more about how to segment notification receivers in [the official documentation](https://docs.microsoft.com/en-us/azure/notification-hubs/notification-hubs-tags-segment-push-message).

### Example expressions

Example devices:

```json
"devices": {
  "A": {
    "tags": [
      "tag1",
      "tag2"
    ]
  },
  "B": {
    "tags": [
      "tag2",
      "tag3"
    ]
  },
  "C": {
    "tags": [
      "tag1",
      "tag2",
      "tag3"
    ]
  },
}
```

- Send to devices that has `tag1` or `tag2`. Example devices A, B and C.

  ```go
  hub.Send(notification, "tag1 || tag2")
  ```

- Send to devices that has `tag1` and `tag2`. Device A and C.

  ```go
  hub.Send(notification, "tag1 && tag2")
  ```

- Send to devices that has `tag1` and `tag2` but not `tag3`. Device A.

  ```go
  hub.Send(notification, "tag1 && tag2 && !tag3")
  ```

- Send to devices that has not `tag1`. Device B.

  ```go
  hub.Send(notification, "!tag1")
  ```

## Changelog

### Latest Updates

- **BREAKING**: Removed deprecated GCM (Google Cloud Messaging) support
  - GCM was deprecated by Google in July 2024
  - All Android functionality now uses FCM v1 (Firebase Cloud Messaging v1)
  - Added `FcmV1Format`, `FcmV1Platform`, `FcmV1TemplatePlatform`, and `FCMV1Platform` constants
  - Updated validation functions to support FCM v1 platforms
  - Added FCM v1 registration and installation support
  - Created test fixtures and comprehensive test coverage
  - Updated documentation with migration guide
- **ENHANCEMENT**: Updated to latest Azure Notification Hubs API version 2016-07 for all operations
- **ENHANCEMENT**: Added comprehensive error handling with custom error types
- **MODERNIZATION**: Updated Go version from 1.12 to 1.21 with modern CI/CD pipeline
- **DOCUMENTATION**: Added detailed contributing guidelines, security policy, and improvement roadmap

### v0.1.4

- Fix for background notifications on iOS 13

### v0.1.3

- Pass the current context to the http request instead of using the background context, thanks to [NathanBaulch](https://github.com/NathanBaulch)
- Add support for installations, thanks to [NathanBaulch](https://github.com/NathanBaulch)
- Add support for batch send, thanks to [NathanBaulch](https://github.com/NathanBaulch)
- Add support for unregistering a device, thanks to [NathanBaulch](https://github.com/NathanBaulch)
- Add automatic testing for Go 1.13
- Two minor bug fixes

### v0.1.2

- Bugfix for reading the message id on standard hubs. Headers are always lowercase.

### v0.1.1

- Bugfix for when device registration were responding an unexpected response.

### v0.1.0

- Support for templated notifications
- Support for notification telemetry in higher tiers

### v0.0.2

- Big rewrite
- Added get registrations
- Travis CI
- Renamed the entities to use the same nomenclature as Azure
- Using fixtures for tests
- Support tag expressions

### v0.0.1

First release by Daresay. Restructured the code and renamed the API according to
Go standards.

## TODO

- Implement cancel scheduled notifications using http DELETE.
  [Find inspo from the Java SDK here.](https://github.com/Azure/azure-notificationhubs-java-backend/blob/d293da9db7564dfd2800e45899f0e2425f669c6e/NotificationHubs/src/com/windowsazure/messaging/NotificationHub.java#L646)

- Android (FCM v1) and iOS are fully supported. Other platforms (Windows, Baidu, ADM) have basic support but could be enhanced further.

## License

See the [LICENSE](LICENSE.txt) file for license rights and limitations (MIT).
