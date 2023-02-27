# Naurt Android SDK

Naurt's Android SDK is a development kit that integrates our technology into your Android apps.

If you would like to enhance location fixes during dynamic journeys on Android, this product is the one for you.



## Introduction

Naurt's Android SDK offers enhanced location abilities over built-in services. Instead of using Android's Location object, Naurt provides a [NaurtLocation](#naurtlocation) class which contains additional information such as:

- Spoofing detection 
- Indoor/outdoor detection
- User motion type
- Error associated with location fix, speed, heading, altitude, etc.
- Location fixes when indoors

The final point above is a key feature of Naurt - location outputs will be available once a second every second, regardless of environment or location. Our combination of sensor fusion, Wi-FI and network localisation will provide tracking for extended indoor periods.



## Quickstart

Naurt is essentially a replacement for the standard location listener. Though before you begin, ensure you have a valid API key. If you don't, no worries. Contact our sales team [here](https://www.naurt.com/#email-form-first-one)


### Project Configuration
Naurt's minimum supported API level is 16 which is Android 4.1 (Jelly Bean). Your project's build.gradle file should contain a "minSdkVersion" of 16 or above.
```groovy
android {
    defaultConfig {
        minSdkVersion 16
    }
}
```



While you're in the build.gradle file, Naurt also needs to be added as a dependency. First add mavenCentral() as a repository.

```groovy
repositories {
    mavenCentral()
    maven { url 'https://jitpack.io' }
}
```
And then Naurt can be added as a dependency.

```groovy
dependencies {
    implementation "com.github.Naurt-Ltd-Public:naurt-android-sdk:2.0.2"
}
```

The latest version of Naurt can be found on [JitPack](https://jitpack.io/#Naurt-Ltd-Public/naurt-android-sdk) with change logs found on our [Github](https://github.com/Naurt-Ltd-Public/naurt-android-sdk/releases).

We're not quite done with dependencies yet. Naurt itself has some dependencies which will also need to be added.

```groovy
dependencies {
    implementation "androidx.core:core-ktx:1.9.0"
    implementation "org.apache.commons:commons-compress:1.21"
    implementation "com.google.android.gms:play-services-auth:20.4.1"
    implementation "org.jetbrains.kotlinx:kotlinx-serialization-json:1.4.1"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.1"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.1"
}
```

During this step watch out for any issues caused by conflicting library versions.

### App permissions

As Naurt accesses the phones network and GPS location services, you'll need to add the corresponding permissions to your AndroidManifest.xml file.

```xml
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
```

### Instantiating Naurt

First, begin by importing the Naurt class and the NaurtEngineType

```kotlin
import com.naurt.Naurt
import com.naurt.NaurtEngineType
```

Once imported, instantiate the Naurt class choosing an engine type. If you would like Naurt to run in it's own foreground service, select SERVICE, else choose STANDALONE (for more information see our [engine type explainer](#background_tracking)). Naurt will also need the application's context.

```kotlin
val naurt = Naurt(
    <YOUR NAURT API KEY HERE> as String,
    applicationContext as Context,
    NaurtEngineType.STANDALONE
)
```

Naurt will ideally be instantiated within the onCreate() method of the app, though it's important the user has granted location permissions first otherwise Naurt will crash.

### Register for location updates

Once the Naurt object has been created, you can then register for location updates by registering for a location callback with Naurt's [on](#on) method.
```kotlin
import com.naurt.NaurtEvents
import com.naurt.NaurtEventListener
import com.naurt.NaurtNewLocationEvent

naurt.on(
    NaurtEvents.NEW_LOCATION,
    NaurtEventListener<NaurtNewLocationEvent> {
        Log.d(
            "naurt",
            "${it.newPoint.timestamp} | ${it.newPoint.latitude}, ${it.newPoint.longitude}"
        )
    }
)
```
By default the callback will be triggered once a second every second and will pass out a [NaurtLocation](#naurtlocation) object. When testing this, ensure the phone can receive network or GPS location updates.


### Disabling Naurt

When you're finished with Naurt and want to stop the location updates, call Naurt's onDestroy method. This will clean up Naurt's internal processes and listeners. We recommend doing this within the onDestroy method of your app.

```kotlin
naurt.onDestroy()
```
And that's all there is to getting enhanced location fixes from Naurt!

This quickstart has hopefully got you up and running with Naurt; however, most apps will require more than just this basic setup. So read on to learn more about how Naurt can help. We also have an example app showing this basic integration in full. It can be found [here](https://github.com/Naurt-Ltd-Public/naurt-android-example-app)





## Background tracking

Naurt can be used to provide location fixes when the app is running in the background. 

To avoid adding the background tracking permission directly in the SDK (which would mean everyone would have to add it), we use different [engine types](#naurtenginetype).

### Service type

Naurt can be used to create a foreground service which will automatically track in the background by using the **SERVICE** engine type. 

```kotlin
import com.naurt.Naurt
import com.naurt.NaurtEngineType

val naurt = Naurt(
    <YOUR API KEY HERE>,
    applicationContext,
    NaurtEngineType.SERVICE
)
```

If you run Naurt in this way, you'll have to add a service definition and corresponding permission within the manifest tags of you AndroidManifest.xml.


```
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />

<application
    <service
        android:name="com.naurt.NaurtService"
        android:enabled="true"
        android:exported="true"
        android:permission="android.permission.ACCESS_FINE_LOCATION"
        android:foregroundServiceType="location|dataSync"
        />

    <receiver
        android:name="com.naurt.StopNaurtBroadcastReceiver"
        android:enabled="true"
        android:exported="true"
        android:permission="android.permission.ACCESS_FINE_LOCATION">
        <intent-filter>
            <action android:name="stopnaurtservice" />
        </intent-filter>
    </receiver>
</application>
```
<!-- html -->
<div class="callout-block callout-block-danger">
    <div class="content">
        <h4 class="callout-title">
            <span class="callout-icon-holder me-1">
                <i class="fas fa-info-circle"></i>
            </span>
            <!--//icon-holder-->
            Warning
        </h4>
        <p>Please do not use the SERVICE engine if your app already has a foreground service. An Android app can only contain one foreground service and will therefore crash.
        </p>
    </div>
    <!--//content-->
</div>
<!-- html -->



### Standalone type

Background is also possible through the **STANDALONE** engine type, though the background location permission will need to be added, or you should run Naurt from within your own foreground service.

With this mode, there is no need to worry about extra Android Manifest services/receivers.

```kotlin
import com.naurt.Naurt
import com.naurt.NaurtEngineType

val naurt = Naurt(
    <YOUR API KEY HERE>,
    applicationContext,
    NaurtEngineType.STANDALONE
)
```

While we're talking about Naurt's initialisation options, it's worth mentioning options for sensor and location listeners.

### Manual managers

The Naurt initialiser also has optional inputs for a sensor manager and a location manager. Naurt needs access to both the phone's sensors and gps module. To do this, we access the sensor and location managers and register listeners.

You may have already done this in your app and if so, you can pass them to Naurt to register listeners under the same manager. Though, we recommend running Naurt in it's default way.

```kotlin
import com.naurt.Naurt
import com.naurt.NaurtEngineType
import android.hardware.SensorManager
import android.location.LocationManager
import android.content.Context

locationManager =
    getSystemService(Context.LOCATION_SERVICE) as
            LocationManager
sensorManager =
    getSystemService(Context.SENSOR_SERVICE) as SensorManager

naurt = Naurt(
    BuildConfig.API_KEY,
    applicationContext,
    NaurtEngineType.SERVICE,
    locationManager,
    sensorManager
)

```







## API key validation

When Naurt is initialised, it will attempt to validate the API key every 24 hours. If a key cannot be validated, Android's standard GPS output will be passed through within the NaurtLocation object to ensure continuous tracking. This will be denoted within the [LocationOrigin](#naurtlocationorigin) property of [NaurtLocation](#naurtlocation).

The easiest way to check whether Naurt is validated yet is by using the [getValidated](#getisvalidated) method.


```kotlin
import com.naurt.NaurtValidationStatus


val isMyApiKeyValidated = naurt.getValidated()

when(isMyApiKeyValidated) {
    NaurtValidationStatus.Valid -> Log.d("app", "API key is valid.")
    NaurtValidationStatus.Invalid -> Log.d("app", "API key is invalid. GPS being passed through.")
    NaurtValidationStatus.NotYetValidated -> Log.d("app", "Naurt is currently attempting to validate.")
}
```

Alternatively, you can use a callback which fires when Naurt has validated the API key. The setup is the same as that of the location callback.

```kotlin

import com.naurt.NaurtEvents
import com.naurt.NaurtEventListener
import com.naurt.NaurtValidationStatus
import com.naurt.NaurtIsValidatedEvent

naurt.on(
    NaurtEvents.IS_VALIDATED,
    NaurtEventListener<NaurtIsValidatedEvent> {
        when (it.isValidated) {
            NaurtValidationStatus.Valid -> Log.d("App", "API key is valid")
            NaurtValidationStatus.Invalid -> Log.d("App", "API key is invalid")
            NaurtValidationStatus.NotYetValidated -> Log.d("App", "This will not occur")
        }
    }
)
```

<!-- html -->
<div class="callout-block callout-block-info">
    <div class="content">
        <h4 class="callout-title">
            <span class="callout-icon-holder me-1">
                <i class="fas fa-info-circle"></i>
            </span>
            <!--//icon-holder-->
            Important
        </h4>
        <p>Though users may not have internet to validate, Naurt will always provide location fixes; though, they will be unprocessed by us and not count towards your key's total usage.
        </p>
    </div>
    <!--//content-->
</div>
<!-- html -->





## Analytics sessions

Naurt collects anonymised location data from the SDK which is primarily done to improve the performance of the SDK. We do however, offer tailored reports and analytics to enterprise customers.

We do this through analytics sessions. An analytics session is a way for you to divide up user data into shorter, more useful portions. For instance, if you're a taxi company you may care more about when the driver is in-ride with a customer than waiting for a job.

Starting one of these sessions can be done using the Naurt method [startAnalyticsSession](#startanalyticssession). When calling this method, you'll also have to provide some metadata about the session as either a **JsonObject** or a **JSON string**.

The metadata should contain useful information such as the driver/vehicle/ride ID so that when we provide analytics reports we can ensure the information correctly corresponds to your services. 

```kotlin
import kotlinx.serialization.json.buildJsonObject
import kotlinx.serialization.json.put

val jsonMetadata = buildJsonObject {
    put("driverID", "exampleID213")
}

naurt.startAnalyticsSession(jsonMetadata)
```

<!-- html -->
<div class="callout-block callout-block-danger">
    <div class="content">
        <h4 class="callout-title">
            <span class="callout-icon-holder me-1">
                <i class="fas fa-info-circle"></i>
            </span>
            <!--//icon-holder-->
            Important
        </h4>
        <p>Please do not send any personal information, or data that directly identifies the user.
        </p>
    </div>
    <!--//content-->
</div>
<!-- html -->

Once the session has finished; for example, when the driver has dropped off the customer, call the [endAnalyticsSession](#endanalyticssession) method.

```kotlin
naurt.endAnalyticsSession()
```


Analytics sessions can be used in many different scenarios and increase the value Naurt brings to your company. If you're still unsure about how analytics sessions could play a part in your use case, contact our [sales team](https://www.naurt.com/#email-form-first-one) to have a chat.



## Points of interest

Another useful feature of Naurt is our points of interest (POI) system. Similar to analytics sessions, POIs enable you to send us metadata associated with the last Naurt location fix. Metadata needs to be formatted as a **JsonObject** or a **JSON string** and can be as detailed as you like.

POIs can be triggered for any reason. For example, you may be a a delivery company who wants to better understand where the front doors of buildings are for future deliveries. In this case, you'd trigger a POI each time the delivery has been completed. 

```kotlin
import kotlinx.serialization.json.buildJsonObject
import kotlinx.serialization.json.put

val jsonMetadata = buildJsonObject {
    put("driverID", "exampleID213")
    put("poiType", "exampleDoor")
}
naurt.pointOfInterest(jsonMetadata)
```

POIs utilise the enhanced Naurt location combined with post-processing on Naurt's servers to offer the best accuracy possible. They are then later accessible through a simple API. THe structure of a POI can be found [here](#pointofinterest).



## Spoofing toolkit

### Mocked location
Customers mocking, spoofing, or providing false locations can enables them to access services they wouldn't otherwise be able to.

When this occurs, the NaurtLocation object provides a simple [isMocked](#naurtlocation) property that can be used to check if the location fix is legitimate or not. For example,
```kotlin
naurt.on(
    NaurtEvents.NEW_LOCATION,
    NaurtEventListener<NaurtNewLocationEvent> {
        if (it.newPoint.isMocked){
            Log.d("App", "Latest location is mocked!")
        }
    }
)
```

Since v2.0.1, Naurt also mitigates mocked locations, when the device is running API level 30 or higher, has internet, and is outside. Though the user is mocking Naurt is able to provide authentic location fixes, though at a slightly degraded accuracy. In this case, the [isMocked](#naurtlocation) will be false, but the [isMockedPrevented](#naurtlocation) property will be true. Make sure to check both booleans for an accurate gauge of how many devices are spoofing locations.

```kotlin
naurt.on(
    NaurtEvents.NEW_LOCATION,
    NaurtEventListener<NaurtNewLocationEvent> {
        if (it.newPoint.isMockedPrevented){
            Log.d("App", "Mocking attempted but Naurt has returned authentic location fix.")
        }
    }
)
```

It's also worthing noting that when these events happen, Naurt will be sent a notification and can later provide reports.

### Phone state

Naurt also provides handy methods to check the phone for states that would suggest advanced spoofing.

To check whether the phone is in developer mode, which is needed for spoofing apps to work, the naurt method [isDeveloper](#isdeveloper) can be used. Similarly, the method [isRooted](#isrooted) checks to see if the user's phone is rooted. If both are false it is unlikely the user is spoofing their location.

You can also use Naurt to see if your app is running in a [work profile](https://support.google.com/work/android/answer/6191949?hl=en) on the user's phone.

```kotlin
val isDeveloper = naurt.isDeveloper()
val isRooted = naurt.isRooted()
val inWorkMode = naurt.inWorkProfile()
```


## Location intelligence

One of Naurt's key features is the additional information provided within the [NaurtLocation](#naurtlocation) object. Two of the most powerful fields are the [NaurtMovement](#naurtmovement) type and the [NaurtEnvironmentFlag](#naurtenvironmentflag) type. 

### Movement type
The movement type specifies if the user is **on foot, in a vehicle or stationary**. This aids decision making within your app and can also be used to validate the information being inputted to it. 

### Environment type
The environment type specifies whether a user is **outdoors or indoors**, though this feature is only available on phones running on API level 30 and up.

When these two flags are used together they can be used for parking detection, front door detection and more. They can also be used as triggers for POI within your app. Here's a short example showing how they are accessed.


```kotlin
import com.naurt.NaurtEvents
import com.naurt.NaurtEventListener
import com.naurt.NaurtEnvironmentFlag
import com.naurt.NaurtMovement

naurt.on(
    NaurtEvents.NEW_LOCATION,
    NaurtEventListener<NaurtNewLocationEvent> {

        when(it.newPoint.environmentFlag){
            NaurtEnvironmentFlag.Outside -> Log.d("App", "User is outside.")
            NaurtEnvironmentFlag.Inside -> Log.d("App", "User is inside.")
            NaurtEnvironmentFlag.NA -> Log.d("App", "This phone is unable to produce this flag.")
        }

        when (it.newPoint.motionFlag){
            NaurtMovement.Stationary -> Log.d("App", "User not currently moving.")
            NaurtMovement.OnFoot -> Log.d("App", "User is on foot.")
            NaurtMovement.VehicleMotion -> Log.d("App", "User is in a vehicle.")
            NaurtMovement.NA -> Log.d("App", "This flag has not been initialised yet.")
        }
    }
)
```

For full API documentation, please visit our [website](https://docs.naurt.net/android_two_sdk).
