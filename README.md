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
}
```
And then Naurt can be added as a dependency.

```groovy
dependencies {
    implementation "com.github.Naurt-Ltd-Public:naurt-android-sdk:2.0.0"
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
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
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

This quickstart has hopefully got you up and running with Naurt; however, most apps will require more than just this basic setup. So read on to learn more about how Naurt can help.





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

Continue below for the full API documentation.





## Naurt class


### Initialisation

To use Naurt, first make an object using the Naurt class. Only one of these objects should ever be created and therefore a single instance should be passed throughout the app.

#### Import
```kotlin
import com.naurt.Naurt
```
#### Constructor signature

```kotlin
Naurt(
	apiKey: String,
	context: Context,
	runType: NaurtEngineType,
	customLocationManager: LocationManager? = null,
	customSensorManager: SensorManager? = null,
)
```

#### Parameters

- `apiKey`: The API key assigned to you from Naurt.
- `context`: The context of your application.
- `engineType`: The engine type as discussed in the [background tracking](#background_tracking) section.
- `customLocationManager`: An **optional** custom location manager as discussed in the [manual managers](#manual_managers) section.
- `customSensorManager`: An **optional** custom sensor manager as discussed in the [manual managers](#manual_managers) section.


#### Returns
As this is a constructor it technically does not return. You should expect an object of type `Naurt` though.

#### Throws
Throws an `Exception` when cannot be initialised correctly. This will be made more specific in future. Successful initialisation can be check with [getIsInitialised](#getisinitialised).




### onDestroy

Called within the onDestroy method of your app. Cleans up Naurt listeners and processes.

#### Signature
```kotlin
naurt.onDestroy()
```
#### Parameters

None

#### Returns

None 

#### Throws
Does not throw.




### startAnalyticsSession

To start an analytics session, use the following method. Either a `JsonObject` from the kotlinx-serialization library can be used or a JSON string. A short explanation of what an analytics session is can be found [here](#analytics_sessions).

#### Signatures
```kotlin
naurt.startAnalyticsSession(metadata: JsonObject)
```
```kotlin
naurt.startAnalyticsSession(metadata: String)
```
#### Parameters

- `metadata`: A `JsonObject` or a string containing the metadata associated with the analytics session.

#### Returns
Nothing. To check whether you are currently in an analytics session please use the method [getInAnalyticsSession](#getinanalyticssession).

#### Throws
Throws an `Exception` when already in an analytics session, or string cannot be converted to a valid JSON.




### restartAnalyticsSession

Restarts the current analytics session. This function prevents the need of calling both stop and start again.
#### Signatures
```kotlin
naurt.restartAnalyticsSession(newMetadata: JsonObject)
```
```kotlin
naurt.restartAnalyticsSession(newMetadata: String)
```
#### Parameters

- `metadata`: A `JsonObject` or a string containing the metadata associated with the new analytics session.

#### Returns
Nothing. To check whether you are currently in an analytics session please use the method [getInAnalyticsSession](#getinanalyticssession).

#### Throws
Throws an `Exception` when not currently in an analytics session, or string cannot be converted to a valid JSON.




### endAnalyticsSession

Ends the current analytics session.
#### Signature
```kotlin
naurt.endAnalyticsSession()
```
#### Parameters

None

#### Returns
Nothing. To check whether you are currently in an analytics session please use the method [getInAnalyticsSession](#getinanalyticssession).

#### Throws
Throws an `Exception` when not currently in an analytics session.




### pointOfInterest

Attaches the provided metadata to the current Naurt location, dispatches it to Naurt, and adds it to a local list of POIs.

#### Signatures
```kotlin
naurt.pointOfInterest(metadata: JsonObject)
```
```kotlin
naurt.pointOfInterest(metadata: String)
```
#### Parameters

- `metadata`: A `JsonObject` or a string containing the metadata associated with the point of interest.


#### Returns
Nothing.

#### Throws
Does not throw.




### getPointsOfInterest

Returns a list of the points of interest recorded since Naurt's last initialisation. 

#### Signature
```kotlin
naurt.getPointsOfInterest(): List<PointOfInterest>
```
#### Parameters

None

#### Returns

- `List<PointOfInterest>`: A list of [PointOfInterest](#pointofinterest_class) recorded so far this initialisation period. See [points of interest](#points_of_interest) for more information.

#### Throws
Does not throw.




### on

Registers a callback for specific Naurt events. An example of how this should be used can be found in the [quickstart](#register_for_location_updates).

#### Signature
```kotlin
<E : NaurtEvent> naurt.on(event: NaurtEvents, listenerNaurt: NaurtEventListener<E>)
```
#### Parameters

- `event`: A [NaurtEvent](#naurt_events) e.g., `NEW_LOCATION` or `IS_VALIDATED`.
- `listenerNaurt`: A Naurt listener event e.g., `NaurtEventListener<NaurtIsValidatedEvent>`

#### Returns

Nothing.

#### Throws
Does not throw.




### removeListeners

Remove a specific listener/callback.
#### Signature
```kotlin
naurt.removeListeners(event: String)
```
#### Parameters
- `event`: The event you want to remove the listener for.

#### Returns

Nothing.

#### Throws
Does not throw.





### removeAllListeners

Remove all listeners/callbacks associated with Naurt.

#### Signature
```kotlin
naurt.removeAllListeners()
```
#### Parameters

None.

#### Returns

Nothing.

#### Throws
Does not throw.





### getLocation

Gets the last available Naurt location. 

#### Signature
```kotlin
naurt.getLocation(): NaurtLocation?
```
#### Parameters

None.

#### Returns

- `NaurtLocation?`: The last Naurt location. If one is not available, this will be null.

#### Throws
Does not throw.




### getLastGnssLocation

Returns the last available Android location. This is useful for comparing outputs of Naurt and GNSS (GPS).

#### Signature
```kotlin
naurt.getLastGnssLocation(): Location?
```
#### Parameters

None.

#### Returns

- `Location?`: The last GNSS location. If not available, returns null.

#### Throws
Does not throw.





### getIsInitialised

Check to see if Naurt has been initialised.

#### Signature
```kotlin
naurt.getIsInitialised(): Boolean
```
#### Parameters

None.

#### Returns

- `Boolean`: Naurt has or hasn't been initialised successfully

#### Throws
Does not throw.




### getIsValidated

Check to see if Naurt has been validated. Validation is not an instant process and can take a long time depending on the internet connection of the user. For more information, see our [validation explainer](#api_key_validation).

 This can take some time depending upon the internet connection

#### Signature
```kotlin
naurt.getIsValidated(): NaurtValidationStatus
```
#### Parameters

None.

#### Returns

- `NaurtValidationStatus`: The [Naurt validation status](#naurtvalidationstatus). Can be valid, invalid, or in the process of validating. See  for more information.

#### Throws
Does not throw.




### getInAnalyticsSession

Check to see whether Naurt is currently in an analytics session. 

#### Signature
```kotlin
naurt.getInAnalyticsSession(): Boolean
```
#### Parameters

None.

#### Returns

- `Boolean`: Whether or not Naurt is currently in an analytics session.

#### Throws
Does not throw.




### getCurrentSessionID

Get the current sessionID. This will be a UUID4 ID with "internalSession" or "analyticsSession" appended onto the end depending upon whether an analytics session has been started.
This will be the ID Naurt uses to identify this session.

#### Signature
```kotlin
naurt.getCurrentSessionID(): String
```
#### Parameters

None.

#### Returns

- `String`: The session ID

#### Throws
Does not throw.




### getDeviceID

Get the unique device ID. This is the ID Naurt will associate with the device.

#### Signature
```kotlin
naurt.getDeviceID(): String
```
#### Parameters

None.

#### Returns

- `String`: The unique device ID

#### Throws
Does not throw.





### getCount

Get the current number of location fixes produced since Naurt was initialised.

#### Signature
```kotlin
naurt.getCount(): Int
```
#### Parameters

None.

#### Returns

- `Int`: The number of location fixes produced since Naurt was initialised.

#### Throws
Does not throw.




### isRooted

Check to see if the current device is rooted.

#### Signature
```kotlin
naurt.isRooted(): Boolean
```
#### Parameters

None.

#### Returns

- `Boolean`: Whether or not the device is rooted.

#### Throws
Does not throw.




### isDeveloper

Check to see if the current device is in developer mode.

#### Signature
```kotlin
naurt.isDeveloper(): Boolean
```
#### Parameters

None.

#### Returns

- `Boolean`: Whether or not the device is in developer mode.

#### Throws
Does not throw.




### inWorkProfile

Check to see if the app is running within a work profile on the device.

#### Signature
```kotlin
naurt.inWorkProfile(): Boolean
```
#### Parameters

None.

#### Returns

- `Boolean`: Whether or not the app is running in a work profile on the device.

#### Throws
Does not throw.






## Events and Listeners



### Naurt Events

An enum which contains the type of events Naurt can emit with the type that is emitted with such events. Each event will have a string ID.



```kotlin
import com.naurt.NaurtEvent

enum class NaurtEvents(val id: String) {
	NEW_LOCATION("NAURT_NEW_LOCATION"),
	IS_VALIDATED("NAURT_IS_VALIDATED"),
}
```

#### `NEW_LOCATION` event
```kotlin
import com.naurt.NaurtNewLocationEvent

NaurtNewLocationEvent(
	val id: String = NaurtEvents.NEW_LOCATION.id,
	val newPoint: NaurtLocation
)
```

#### `IS_VALIDATED` event
```kotlin
import com.naurt.NaurtIsValidatedEvent

NaurtIsValidatedEvent(
	val id: String = NaurtEvents.IS_VALIDATED.id,
	val isValidated: NaurtValidationStatus
)
```




### NaurtEventListener

The listener interface which distributes the NaurtEvents seen above.


#### Signature
```kotlin
import com.naurt.NaurtEventListener

fun interface NaurtEventListener<E : NaurtEvent> {
	fun onEvent(event: E)
}
```
#### Parameters

- `event`: A NaurtEvent.

#### Returns

Nothing.

#### Throws
Does not throw.





## NaurtLocation

The NaurtLocation class contains the location fix and additional data. It is emitted every second through the `NEW_LOCATION` event when registered with the `on` method.

### Import
```kotlin
import com.naurt.NaurtLocation
```

### Properties

```kotlin
data class NaurtLocation(
	/** WGS84 Degrees */
	val latitude: Double = Double.NaN,

	/** WGS84 Degrees */
	val longitude: Double = Double.NaN,

	/** UNIX Milliseconds since 01/01/1970. This is the device time. */
	val timestamp: Long = 0L,

	/** UNIX Milliseconds as calculated by the GNSS constellation. Not device time.*/
	val locationProviderTimestamp: Long = 0L,

	/** Metres, 1 Standard Deviation */
	val horizontalAccuracy: Double = Double.NaN,

	/** Metres per second */
	val speed: Double = Double.NaN,

	/** Degrees, North being 0, rotating clockwise */
	val heading: Double = Double.NaN,

	/** Metres per second, 1 Standard Deviation */
	val speedAccuracy: Double = Double.NaN,

	/** Degrees, North being 0, rotating clockwise, 1 Standard Deviation */
	val headingAccuracy: Double = Double.NaN,

	/** Metres Squared */
	val horizontalCovariance: Double = Double.NaN,

	/** Metres from EGS84 Geoid - Passed through from Android */
	var altitude: Double = Double.NaN,

	/** Metres - Passed through from Android */
	var verticalAccuracy: Double = Double.NaN,

	/** Metres - Cumulative distanced traveled this journey */
	var cumulativeDistance: Double = Double.NaN,

	/** Movement type. */
	var motionFlag: NaurtMovement = NaurtMovement.NA,

	/** Where the location was sourced from. GNSS, Network, Naurt Standard, Naurt Advanced. */
	var locationOrigin: NaurtLocationOrigin = NaurtLocationOrigin.GNSS,

	/** The environment the user is in. */
	var environmentFlag: NaurtEnvironmentFlag = NaurtEnvironmentFlag.NA,

	/** Whether the location fix was produced in the foreground or background */
	var backgroundStatus: NaurtBackgroundStatus = NaurtBackgroundStatus.Foreground,

	/** Whether or not the location fix has been mocked **/
	var isMocked: Boolean = false,
) 
```




## NaurtEnvironmentFlag

This enum contains information about the user's location.

### Import
```kotlin
import com.naurt.NaurtEnvironmentFlag
```

### Definition
```kotlin
enum class NaurtEnvironmentFlag {

	/**
	 * Naurt's environment flag.
	 *
	 * Outside: User is outside
	 * Inside: User is inside.
	 * NA: Flag not available, likely running on a phone below API 30.
	 */
	Outside,
	Inside,
	NA,
}
```





## NaurtMovement

This enum contains information about the user's movement type.

### Import
```kotlin
import com.naurt.NaurtMovement
```

### Definition
```kotlin
enum class NaurtMovement {
	/**
	 * Naurt's movement status.
	 *    Stationary: The device is not moving.
	 *    OnFoot: The user is running/walking etc.
	 *    VehicleMotion: The user is in a vehicle.
	 *    NA: Not yet initialised.
	 */
	Stationary,
	OnFoot,
	VehicleMotion,
	NA,
}

```





## PointOfInterest class

This class contains the metadata and the associated location fix.

### Import
```kotlin
import com.naurt.poi.PointOfInterest
```

### Definition
```kotlin
data class PointOfInterest(
	val naurtLocation: NaurtLocation,
	val metadata: JsonObject
)
```





## NaurtEngineType

This enum is used to define how Naurt should be initialised. See [x](insert link) for more information.

### Import
```kotlin
import com.naurt.NaurtEngineType
```

### Definition
```kotlin
enum class NaurtEngineType {
	/**
	 * Naurt's engine status.
	 *
	 * STANDALONE: The engine is running Naurt normally.
	 *
	 * SERVICE: The engine running within a foreground service.
	 */
	STANDALONE,
	SERVICE,
}

```





## NaurtValidationStatus

This enum contains information about the validation status of Naurt.

### Import
```kotlin
import com.naurt.NaurtValidationStatus
```


### Definition
```kotlin
enum class NaurtValidationStatus {
	/**
	 * Naurt's validation status
	 *
	 * Valid: API key has been validated.
	 *
	 * Invalid: API key is not valid.
	 *
	 * NotYetValidated: Still attempting to validate API key.
	 */
	Valid,
	Invalid,
	NotYetValidated,
}
```



## NaurtLocationOrigin

This enum contains information about how the Naurt Location object was created. For example, when an API key cannot be validated, Naurt will provide location fixes from GNSS. 

### Import
```kotlin
import com.naurt.NaurtLocationOrigin
```


### Definition
```kotlin
enum class NaurtLocationOrigin {
	/**
	 * Naurt's location origin
	 *
	 * GNSS: A pass-through point. User did not have valid API key etc.
	 * Naurt Network: Low quality network location fixes. Only occurs when starting inside.
	 * Naurt Standard: Standard version of Naurt. Android API level 29 and below.
	 * Naurt Advanced: Best location fixes. For API 30 and above. Requires internet every ~30 minutes.
	 */
	GNSS,
	NaurtNetwork,
	NaurtStandard,
	NaurtAdvanced,
}
```



## NaurtBackgroundStatus

This enum contains information on whether the location fix was produced in the background or foreground.

### Import
```kotlin
import com.naurt.NaurtBackgroundStatus
```


### Definition
```kotlin
enum class NaurtBackgroundStatus {
	/**
	 *  Naurt's background status.
	 *
	 * Foreground: Location fix was produced in the foreground.
	 * Background: Location fix was produced in the background.
	 */
	Foreground,
	Background,
}
```
