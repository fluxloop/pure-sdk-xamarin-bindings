# Pure SDK – Xamarin Bindings
## Getting started
> nuget sources Add -Name "PureSDK" -Source "https://pkgs.dev.azure.com/fluxloop-devops/_packaging/Pure/nuget/v3/index.json"

Credentials will be provided.

## iOS Bindings
Add the package `Pure.iOS.Sdk` *or* `Pure.iOS.Sdk.Bluetooth`.

### Permissions
Install the bluetooth package if support for Eddystone is required.  

Some of the SDK's data collection sources require extra keys in `Info.plist`. The following keys are required for location tracking:

* `NSLocationWhenInUseUsageDescription`: Describes how your app will use location services in the foreground. You must include this key if you wish to collect location events.
* `NSLocationAlwaysAndWhenInUseUsageDescription`: Describes how your app will use location services and explains what extra features you can provide if the always option is selected. The presented alert gives the user the option between "Only when in use", "Always", and "Never". Only used on iOS 11 or later.
* `NSLocationAlwaysUsageDescription`: Describes how your app will use location services, both in the foreground and background. The presented alert gives the user the option between "Always", and "Never". If your deployment target is at iOS 11 or higher, then you don't need this key even if you wish to collect location events.

If targeting iOS 11 or higher, only `NSLocationWhenInUseUsageDescription` and `NSLocationAlwaysAndWhenInUseUsageDescription` is required.

If the bluetooth package is added, the `NSBluetoothPeripheralUsageDescription` key has to be added to the application's `Info.plist`. Failure to include this key will cause your app to be rejected on App Store Connect upload.

### Usage
Two method calls are required to initialize the SDK and start tracking.

Call the following method before the return statement in `FinishedLaunching` residing in `AppDelegate.cs` to initialize the SDK:

```cs
Pure.InitializeWithLaunchOptions(launchOptions);
```

The second call is used to start tracking, and call be added anywhere of your choosing:

```cs
using PureSDK;
…
Pure.StartTracking();
```

### Sending custom data
#### Event
Creating an event:

```cs
using PureSDK;
…
var eventType = "ORDER";
var payload = new NSDictionary(
    new NSString("userId"), new NSNumber(1234567),
    new NSString("orderId"), new NSNumber(999999),
    new NSString("timestamp", new NSString("2019-05-24T11:49:31+00:00")))
);

Pure.CreateEventWithType(
    eventType,
    payload,
    () => {
        // nullable argument
        // handle success
    },
    (error) => {
        // nullable argument
        // handle error
    }
);
…
```
The `type` argument describes the event. All calls to `CreateEventWithType` are handled as unique, and subsequent calls with the same arguments will __not__ be de-duplicated. This method requires that tracking has been started and will return an error if tracking is not started. To override this behaviour, use the overloaded method and pass `true` to the `force` argument.

#### Metadata
Associating metadata for the current user:
```cs
using PureSDK;
…
var eventType = "USER_INFO";
var payload = new NSDictionary(
    new NSString("userId"), new NSNumber(1234567),
    new NSString("gender"), new NSString("male"),
    new NSString("name", new NSString("Ola Nordmann")))
);

Pure.AssociateMetadataWithType(
    eventType,
    payload,
    () => {
        // nullable argument
        // handle success
    },
    (error) => {
        // nullable argument
        // handle error
    }
);
…
```
The `type` argument describes the event, and has to be unique for each payload. Subsequent calls to `AssociateMetadataWithType` with the same `type` will **override** the previous payload. This method requires that tracking has been started and will return an error if tracking is not started. To override this behaviour, use the overloaded method and pass `true` to the `force` argument.

## Android Bindings
Add the package `Pure.Android.Sdk`.
### Permissions
The following permissions are required to collect and report data, and will be automatically merged into your `AndroidManifest.xml`:

- `ACCESS_COARSE_LOCATION`
- `ACCESS_FINE_LOCATION`
- `ACCESS_NETWORK_STATE`
- `ACTIVITY_RECOGNITION`
- `ACCESS_WIFI_STATE`
- `BLUETOOTH`
- `BLUETOOTH_ADMIN`
- `CHANGE_WIFI_STATE`
- `INTERNET`
- `RECEIVE_BOOT_COMPLETED`

If you need to limit the number of permissons, you can use the following syntax in your `AndroidManifest.xml` to remove a permission:

```xml
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" tools:node="remove" />
```

Android 6.0 and later requires you to explicitly request permission to use the required permissions. You need to request `ACCESS_FINE_LOCATION` programatically.
Example:
```cs
const int LOCATION_REQUEST_CODE = 99;
…
if (ContextCompat.CheckSelfPermission(Application.Context, Manifest.Permission.AccessFineLocation) != PermissionChecker.PermissionGranted) 
{
    RequestPermissions(
        new string[] { Manifest.Permission.AccessFineLocation },
        LOCATION_REQUEST_CODE
    )
}
```

### Usage
The SDK is initialized on launch by default, but will not gather any data until the required location permission is granted and you explicitly start tracking movement.

To start tracking:
```cs
using Com.Pure.Sdk;
…
Pure.startTracking();
```
The SDK stores the tracking state, as subsequent calls to `Pure.startTracking()` is not required for each app launch.

To start tracking immediately after location permission is granted, call the `Pure.startTracking()` method as soon as the permission is granted.

It is recommended to give the user a way to opt out at a later stage.
To stop tracking:
```cs
using Com.Pure.Sdk;
…
Pure.stopTracking();
```

You can check if tracking by accessing the property `Pure.isTracking`.

#### Event
Creating an event:
```cs
using Com.Pure.Sdk;
…
var eventType = "ORDER";
var payload = new Org.Json.JSONObject();
payload.Put("userId", 1234567);
payload.Put("orderId", 999999);
payload.Put("timestamp", "2019-05-24T11:49:31+00:00");

Pure.CreateEvent(
    eventType,
    payload,
    null // You can pass an implementation of IPureCallback here to handle success and error if required
);
…
```
The `type` argument describes the event. All calls to `CreateEvent` are handled as unique, and subsequent calls with the same arguments will __not__ be de-duplicated. This method requires that tracking has been started. To override this behaviour, use the overloaded method and pass `true` to the `force` argument.

#### Metadata
Creating an event:
```cs
using Com.Pure.Sdk;
…
var eventType = "USER_INFO";
var payload = new Org.Json.JSONObject();
payload.Put("userId", 1234567);
payload.Put("gender", "mald");
payload.Put("name", "Ola Nordmann");

Pure.AssociateMetadata(
    eventType,
    payload,
    null // You can pass an implementation of IPureCallback here to handle success and error if required
);
…
```
The `type` argument describes the event, and has to be unique for each payload. Subsequent calls to `AssociateMetadata` with the same `type` will **override** the previous payload. This method requires that tracking has been started. To override this behaviour, use the overloaded method and pass `true` to the `force` argument.



