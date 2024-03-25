---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
description: This guide will help you to get started.
---

# Android Quick Start
## Overview

- [Android Quick Start](#android-quick-start)
  - [Overview](#overview)
  - [Current record time of implementation: 34 min\*](#current-record-time-of-implementation-34-min)
  - [Prerequisites](#prerequisites)
- [Add SDK to your app, latest version: `2.0.23`](#add-sdk-to-your-app-latest-version-2023)
  - [Usecases](#usecases)
  - [Setup](#setup)
  - [Changing the floor](#changing-the-floor)
  - [TT2.Navigation](#tt2navigation)
  - [Analytics](#analytics)
  - [Trigger events](#trigger-events)
- [Map SDK](#map-sdk)
  - [Map SDK version is the same as the SDK version](#map-sdk-version-is-the-same-as-the-sdk-version)
  - [Add a the view to your layout](#add-a-the-view-to-your-layout)
    - [Example using the map view with a fragment](#example-using-the-map-view-with-a-fragment)
  - [MarkerController](#markercontroller)
    - [Guide to MarkerController](#guide-to-markercontroller)
  - [PathfindingController](#pathfindingcontroller)
    - [Guide to PathfindingController](#guide-to-pathfindingcontroller)
  - [ZoneController](#zonecontroller)
    - [Guide to ZoneController](#guide-to-zonecontroller)
  - [CameraController](#cameracontroller)
    - [Guide to CameraController](#guide-to-cameracontroller)
- [Known issues](#known-issues)
  - [Kepp your app running in the background](#kepp-your-app-running-in-the-background)

## Current record time of implementation: 34 min*
  *Integrating the SDK, initializing SDK, Store, Analytics and performing test round collecting heatmap data to be viewd in the dashboard on the CMS.
<br/><br/>

## Prerequisites
Install or update [Android Studio](https://developer.android.com/sdk) to its latest version.

Make sure that your project meets these requirements:

- Targets API level 26 or higher
- Uses Android 7.0 or higher
<br/><br/>

# Add SDK to your app, latest version: `2.0.23`

Add the SDK to your app level `build.gradle` file:
//Todo: check this info:

Project level 
```gradle
allprojects {
    repositories {
        maven {
            credentials {
                username "provided by Virtual stores"
                password "provided by Virtual stores"

            }
            url = "https://nexus.aws.vs-office.se/repository/external-public/"
        }
    }
}
```

```gradle
dependencies {
    ... 
    implementation("se.virtualstores:tt2-android-sdk:${SDK_VERSION}")
    ...
}
```
<br/><br/>
## Usecases
[Shop & Go App](usecase-shop-and-go.html "Example")
<br/><br/>

## Setup

1- To get the SDK ready to work first Call initialize method. This will prepare the SDK for all other purposes.

{% include android/code-sample-tt2-initialize.md %}

When this step is done, you can get a list of the stores by calling: `TT2.activeStores` or query for the store that your user selected. A store can be either active or inactive. While setting up a store for indoor positioning the store should be inactive and when it's ready for production you activate the store in the CMS.
You can access all stores, active or not, by calling `TT2.stores` to test a store in your development phase.

You can show the list of the stores to the user and choose one.

2- To initialize your store, call initStore

When the user chose a store you can initialize the selected store by calling:

  ```kotlin
  TT2.initStore(
            context = //applicationContext,
            storeId = //Your store Id
            floorLevelId = // (optional) The floor level to start on. If not set the SDK will initiate the default floorLevel configured in the CMS
        ) { error ->
            if (error != null) {
                // Show error to user in case of any exception happened during initialization including network exception
            } else  {
                // Safe to continue
            }
        }
  ```

You can declare an external ID for each store matching your organisations store id's. Depending on your implementation this might be your prefered solution. To get the store you can query the stores on the external id property to find your match.

```kotlin
TT2.activeStores.find { it.externalId == "Your organisation store ID" }?.let { store ->
    TT2.initStore(applicationContext, store.id){ error ->
        ...
    }
}
```
<br/><br/>

## Changing the floor

If you need to change the current floor manually:
```kotlin
TT2.initiateFloorLevel(
            context = //applicationContext,
            floorLevelId = // The floor level to start navigation with
        ) { error->
            if (error != null) {
                // Show error to user in case of any exception happened during initialization including network exception
            } else  {
                // Safe to do the next steps
            }
        }
```
<br/><br/>

## TT2.Navigation

TT2.Navigation handles the TT2 positioning system position input control.
It's possible to start the positioning system in different ways.
It's recommended to start with a QR code for the most accurate positioning.
You can access the navigation functionalities by calling TT2.navigation:

IScanLocations can be set up in the TT2 CMS. Filter and find what start qr code has been scanned.
```kotlin
// using the input from scanning a qr-code 
TT2.activeStore.startScanLocations.find { scanLocation ->
    scanLocation.code == scanResult
}?.let{ startScanLocation -> 
    TT2.navigation.syncPosition(startScanLocation)
}
```

It's recommenden to sycronize the users location when possible, i.e If a user scans a product or shelf can use that information to calibrate the users position. 

For example if a user scans a an item:
```kotlin
TT2.navigation.syncPosition(
        identifier: "<scannerInput>",
        syncAngle: Boolean = false, // Scenario: You know the user scanned a shelf label set this to true.
        uncertainAngle: Boolean = true, // Scenario: The user scanned a shelf label and the device was held towards the shelf, set this to false. 
        withForce: Boolean = false, // Use only for debugging purposes
        callback: ((OperationResult<String>) -> Unit)? = null,
    )
```
When the positioning should stop just call tt2.navigation.stop()

```kotlin
TT2.navigation.stop()
```
<br/><br/>

## Analytics

Analytics is used for posting position related data to a server that later can be used for analysis and decision making.

For tracking position based analytics you need to start a Visit in the TT2.Analytics.
It is recommended to start the visit when the navigation starts and to start collecting heatmap data on the start visit callback.

```kotlin
fun startNavigation(position: PointF, angle: Double) {
    TT2.navigation.syncPosition(position, angle)
    TT2.analytics.startVisit(
        deviceInformation = DeviceInformation(
            operatingSystem = "Android",
            osVersion = Build.VERSION.RELEASE,
            appVersion = BuildConfig.VERSION_NAME,
            Build.MODEL),
        tags = mapOf(
                // More tags can be added for more ways of filtering all visits. 
                // We recommended to use these tags as they are already prepared in the CMS for data filtering.
                "userID" to user.ID,
                "userGender" to user.gender,
                "userAge" to user.age 
            ),
        metaData = mapOf(
            "customTag" to "customData"
        )
    ) {
        TT2.analytics.startCollectingHeatMapData()          
    }
}
```
<br/><br/>
## Trigger events
Trigger events can be created in the TT2 CMS or via the SDK. 
Trigger events are mostly set to trigger on user locaton, when entering/exiting a specific area or radius.
To listen for these events implement TriggerEventController.Listener interface and set the listener in TT2 after initiating the store.

```kotlin
import se.virtualstores.tt2.androidsdk.Listener

class MapActivity : AppCompatActivity(), TriggerEventController.Listener {
    fun storeInitiated(){
        TT2.triggerEvents.setTriggerListener(this)
    }

    override fun onNewTriggerEvent(triggerEvent: TriggerEvent) {

        // parse the trigger event to show it to the user. I.e in the event of an 
        // message event created in the CMS the event will contain the following metaData tags: 
         triggerEvent.metaData[TriggerEvent.defaultMetaData.id]?.let {
         
         }

         triggerEvent.metaData[TriggerEvent.defaultMetaData.type]?.let {
            when (it) {
                    TriggerEvent.DefaultMetaData.MessageType.SMALL.name -> {}
                    TriggerEvent.DefaultMetaData.MessageType.LARGE.name -> {}
         }

         triggerEvent.metaData[TriggerEvent.defaultMetaData.title]?.let {

         }

         triggerEvent.metaData[TriggerEvent.defaultMetaData.body]?.let {
             
         }

         triggerEvent.metaData[TriggerEvent.defaultMetaData.imageUrl]?.let {
             
         }

        // after receiving an event you can optionally remove it to avoid trigger it again during this visit
        TT2.triggerEvents.removeTriggerEvent(triggerEvent)

        // For viewing analytics data of the messages we recommend that you also post the trigger event to the anlaytics.
        TT2.analytics.postEventData(triggerEvent.toMessageShownEvent())
    }
}
```

Trigger events can also be created and then listen on when they are triggered.

Setup trigger events by adding them to the trigger manager
In this example we set upp will trigger event that will trigger when a user comes inside a `5m radius` of the item relating to the  `"<barcode>"`
```kotlin
TT2.position.getByBarcode("<barcode>") { item ->  // getting an IItem matching the barcode
    it?.itemPosition?.let { position ->
        TT2.triggerEvents.addTriggerEvent( //Adding the trigger event to the trigger event manager.
            TriggerEvent.Builder().apply {
                setName("Coordinate Trigger")
                // add as mouch metaData as you like
                    addMetaData(
                        listOf(
                            TriggerEvent.defaultMetaData.id to "<barcode>",
                            TriggerEvent.defaultMetaData.title to "RadiusTrigger",
                            TriggerEvent.defaultMetaData.body to "Close to kitchen",
                            "CustomTag" to "foo"
                        )
                    )
                    setTriggerEventData(
                        CoordinateTrigger(
                            x = position.point.x.toDouble(),
                            y = position.point.y.toDouble(),
                            radius = 5.0
                        )
                    )
            }.build()
        )
    }
}
```

When navigation is stopped or the user quits the app, the visit and heatmap collection should be stopped.

```kotlin
fun stopNavigation(){
    TT2.navigation.stop()
    TT2.analytics.stopVisit() 
    //stopVisit() will stop the collection heatmap data internaly. If the heatmap collection wants to be stopped alone TT2.analytics.stopCollctingHeatMapData() can be called.
}
```
<br/><br/>


# Map SDK

Current record time of implementation: Not set, will you be the first?

## Map SDK version is the same as the SDK version

```gradle

allprojects {
    repositories {
    
        maven {
            url 'https://api.mapbox.com/downloads/v2/releases/maven'
            authentication {
                basic(BasicAuthentication)
            }
            credentials {
                username = "mapbox"
                password = "provided by Virtual stores"
            }
        }
    }
}


```


```gradle
dependencies {
    ... 
    implementation("se.virtualstores:lib-map:${SDK_VERSION}")
    ...
}
```

## Add a the view to your layout

```xml
<se.virtualstores.lib_map.pub.view.MapView
    android:id="@+id/mapView"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

### Example using the map view with a fragment
Documentation: [MapListener](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-map-listener/index.html)

Documentation: [LifecycleListener](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-lifecycle-listener/index.html)

Documentation: [MapOptions](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain/-map-options/index.html)

Documentation: [MapController](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-map-controller/index.html)

Example:
```kotlin


class MyMapFragment: Fragment(), MapListener {
    
    var mapController: MapController? = null

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        mapController = BaseMapController(
            binding.mapView,
            MapOptions())

        mapController.mapListener = this
        TT2.setMapController(mapController)
    }

    // the map is now fully loaded and it's now safe to start using it
    override fun onMapLoaded() {
        super.onMapLoaded()
        // continue map setup
    }
    

    // Lifecycle callbacks for map controller
    override fun onStart() {
        super.onStart()
        mapController.onStart()
    }

    override fun onResume() {
        super.onResume()
        mapController.onResume()
    }

    override fun onPause() {
        super.onPause()
        mapController.onPause()
    }

    override fun onStop() {
        super.onStop()
        mapController.onStop()
    }

    override fun onLowMemory() {
        super.onLowMemory()
        mapController.onLowMemory()
    }

    override fun onDestroyView() {
        super.onDestroyView()
        mapController.onDestroy()
    }        
}
```
<br/><br/>

## MarkerController
Documentation: [MarkerController](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-marker-controller/index.html)

### [Guide to MarkerController](./mapsdk/marker_controller.html "Example")
<br/><br/>

## PathfindingController

Documentation: [PathfindingController](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-pathfinding-controller/index.html)

### [Guide to PathfindingController](./mapsdk/pathfinding_controller.html "Example")
<br/><br/>


## ZoneController

Documentation: [ZoneController](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-zone-controller/index.html)

### [Guide to ZoneController](./mapsdk/zone_controller.html "Example")


<br/><br/>

## CameraController

Documentation: [CameraController](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-camera-controller/index.html)

Documetation: [CameraModes](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-camera-controller/-camera-modes/index.html)

### [Guide to CameraController](./mapsdk/camera_controller.html "Example")
<br/><br/>

# Known issues
## Kepp your app running in the background
To keep the position service running even when the user switches to another application or locks the phone and puts it in the pocket it is not acceptable for the Android OS to kill the application’s background services, ofc... 

Since the introduction of doze mode in Android Marshmallow we have seen a number of enterprise use cases where customers want to ensure their application continues to run even though the device wants to enter a power saving mode. So as a developer this is your responsiblity to keep your app alive and keep up with the latest changes of Android OS to make this possible. Here is a list that may help you do this but may not be all the best solutions to this problem. Let us know if you know a better way of handling this.
- Read the android documentation about [the dose mode](https://developer.android.com/training/monitoring-device-state/doze-standby.html)
- [Whitelist](https://stackoverflow.com/a/54982071/3248593) your app from being optimized. This is something you need to do for every manufacturer because some have [separate battery optimization](https://stackoverflow.com/a/56993037/3248593). You may use [third party libraries](https://github.com/pvsvamsi/AppKillerManager) to do so or do it yourself. If you use Flutter there is a [library](https://github.com/SachinGanesh/battery_optimization) that may help you.

Other mentionable notes
- Sensor quality differs between android devices. In some devices some sensors may not be available. The quality of the sensors might affect the accuracy of the positioning.
- You can call on `TT2.validateDevice(context)` to validate the device has the required sensors, not that this is not a measurment on the quality of the sensors.
- `checkForMagnetometer` : Depending on your integration you might want to also validate that the device has a `magnetometer`. If your integration only uses StartScanLocations by QR-codes or cradleId for hardware souch as Zebra `PS20` then there is no need for the `magnetometer`. However if you intend to start the positioning on scanning items in store then the system will rely on the `magnetometer` to get the orientation of the device and therefore the `magnetometer` has to be available.

```kotlin
TT2.validateDevice(
    context
    checkForMagnetometer = false // optional depending on your use case
) { unSupportedDeviceException ->
    unSupportedDeviceException?.let {
        // This means that the device is unsupported due to missing crucial sensors
    }
}
```
