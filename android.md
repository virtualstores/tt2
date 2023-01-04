---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
description: This is the android description
---

# Code samples
## Overview

- [Code samples](#code-samples)
  - [Overview](#overview)
  - [Current record time of implementation: 34 min\*](#current-record-time-of-implementation-34-min)
  - [Prerequisites](#prerequisites)
- [Add SDK to your app, latest version: `1.4.1`](#add-sdk-to-your-app-latest-version-141)
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
  - [PathfindingController](#pathfindingcontroller)
  - [ZoneController](#zonecontroller)
  - [CameraController](#cameracontroller)
- [Known issues](#known-issues)
  - [Kepp your app running in the background](#kepp-your-app-running-in-the-background)

## Current record time of implementation: 34 min*
  *Integrating the SDK, initializing SDK, Store, Analytics and performing test round collecting heatmap data to be viewd in the dashboard on the CMS.

## Prerequisites
Install or update [Android Studio](https://developer.android.com/sdk) to its latest version.

Make sure that your project meets these requirements:

//Todo: check this info:
- Targets API level 24 or higher
- Uses Android 7.0 or higher


# Add SDK to your app, latest version: `1.4.1`

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
## Usecases
[Shop & Go App](usecase-shop-and-go.html "Example")


## Setup

1- To get the SDK ready to work first Call initialize method. This will prepare the SDK for all other purposes.



```kotlin
TT2.initialize(
            context = //applicationContext
            apiUrl = // The api url to connect to
            apiKey = // Your API key
            clientId = //Your client ID
            serviceNotificationIntent =  Intent(context, ClassToReceiveIntent::class.java) // The intent which will be used by the foreground service running the positioning logic, it will also handle user interaction with the notification that will be displayed in the notification center when the app is in background.
        ) { error ->
            if (error != null) {
                // Show error to user in case of any exception happened during initialization including network exception
            } else  {
                // Safe to do the next steps
            }
        }
```

When this step is done, you can get a list of the stores by calling: `TT2.activeStores`

You can show the list of the stores to the user and choose one.

2- To initialize your store, call initStore

When the user chose a store you can initialize the selected store by calling:

  ```kotlin
  TT2.initStore(
            context = //applicationContext,
            storeId = //Your store Id
            floorLevelId = // (optional) The floor level to start on. If not set the SDK will init the default floorLevel configured in the CMS
        ) { error ->
            if (error != null) {
                // Show error to user in case of any exception happened during initialization including network exception
            } else  {
                // Safe to do the next steps
            }
        }
  ```


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

Get the position of the product/shelf and use it as input to the system.
Position determines where on the map the user will be synced to.
For example if a user scans a shelf label:
```kotlin
TT2.position.getByBarcode("<scannerInput>") { item -> 
    TT2.navigation.syncPosition(
        position = item.itemPosition,
        syncAngle = true, // Scenario: You know the user scanned a shelf label.
        uncertainAngle = false // Scenario: The user scanned a shelf label and the device was held towards the shelf. 
    )                   
}
```
When the positioning should stop just call tt2.navigation.stop()

```kotlin
TT2.navigation.stop()
```


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
## Trigger events
Trigger events can be created in the TT2 CMS or via the SDK. 
Trigger events are mostly set to trigger on user locaton, when entering/exiting a specific area or radius.
To listen for these events implement TriggerEventManager.Listener interface and set the listener in TT2 after initiating the store.

```kotlin
import se.virtualstores.tt2.androidsdk.Listener

class MapActivity : AppCompatActivity(), TriggerEventManager.Listener {
    fun storeInitiated(){
        TT2.triggerEventManager.setTriggerListener(this)
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
        TT2.triggerEventManager.removeTriggerEvent(triggerEvent)

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
        TT2.triggerEventManager.addTriggerEvent( //Adding the trigger event to the trigger event manager.
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


## MarkerController
Documentation: [MarkerController](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-marker-controller/index.html)
Example:
```kotlin
// Implement interface MarkerController.Listener
class MyMapFragment: Fragment(), MapListener, MarkerController.Listener {
    
    // the map is now fully loaded and it's now safe to start using it
    override fun onMapLoaded() {
        super.onMapLoaded()
        
        mapController.marker.addListener(this)
    }

    // using map marks
    fun addMarkToMap(data: YourData, itemPosition: IItemPosition) {
        // the BaseMapMark class that are available in the SDK can show two different types of information, image or text
        // You can design your own map marks by extending :MapMark<T>, Comparable<MapMark<T>>
        val mark = BaseMapMark(
            id = data.shelfId,
            position = itemPosition.point,
            floorLevelId = itemPosition.floorLevelId,
            data = data,
            // choose either mark with image:
            imageURL = data.imageUrl,
            // or mark with text:
            text = data.label
        )

        mapController.marker.addMark(mark)
    }

        
    override fun onMarkClick(mark: MapMark<out Any>) {
        (mark.data as? YourData)?.let {

        }
    }

    override fun onClusterClicked(marks: List<MapMark<out Any>>) {

    }

    override fun onMarkTriggerEnter(mark: MapMark<out Any>) {
        
    }

    override fun onMarkTriggerExit(mark: MapMark<out Any>) {
        
    }

```



## PathfindingController
Documentation: [PathfindingController](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-pathfinding-controller/index.html)

Example:
```kotlin
// Implement interface PathfindingController.Listener
class MyMapFragment: Fragment(), MapListener, PathfindingController.Listener {
    
    // the map is now fully loaded and it's now safe to start using it
    override fun onMapLoaded() {
        super.onMapLoaded()

        mapController.pathfinder.addListener(this)
    }

    // Using the pathfinder 
    fun addPathfinderGoalToMap(data: YourData, itemPosition: IItemPosition) {        
        val goal = BasePathfindingGoal(
            id = data.id,
            position =  itemPosition.point,
            floorLevelId = itemPosition.floorLevelId,
            data = data
        )

        mapController.pathfinding.addGoal(goal)
    }

    // listening for pathfinder updates
    override val pathfindingListenerId: String
        get() = <choose an id for this listener>

    override fun onCurrentGoalChange(goal: PathfindingController.PathfindingGoal<out Any>?) {
         (goal.data as? YourData)?.let {
            
        }   
    }

    override fun onSortedGoalChange(goals: List<PathfindingController.PathfindingGoal<out Any>>) {
            
    }

    
```


## ZoneController

Documentation: [ZoneController](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-zone-controller/index.html)

Example:
```kotlin

class MyMapFragment: Fragment(), MapListener, ZoneController.Listener {
    
    override fun onMapLoaded() {
        super.onMapLoaded()
        
        // display all zones on the map
        mapController.zones?.showAll()

        // select a zone to change the apearance of the zone 
        mapController.zones?.zones.first.let {
            mapController.zones?.select(it)
        }
    }

    override fun onEnter(zone: TT2Zone) {
        
    }

    override fun onExit(zone: TT2Zone) {
        
    }


```


## CameraController
Documentation: [CameraController](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-camera-controller/index.html)

Documetation: [CameraModes](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-camera-controller/-camera-modes/index.html)

Example:
```kotlin

class MyMapFragment: Fragment(), MapListener, LifecycleListener {
    
    override fun onMapLoaded() {
        super.onMapLoaded()

        mapController.camera.updateCameraMode(mapController.camera.modes.containMap2D())
    } 
```

# Known issues
## Kepp your app running in the background
To keep the position service running even when the user switches to another application or locks the phone and puts it in the pocket it is not acceptable for the Android OS to kill the application’s background services, ofc... 

Since the introduction of doze mode in Android Marshmallow we have seen a number of enterprise use cases where customers want to ensure their application continues to run even though the device wants to enter a power saving mode. So as a developer this is your responsiblity to keep your app alive and keep up with the latest changes of Android OS to make this possible. Here is a list that may help you do this but may not be all the best solutions to this problem. Let us know if you know a better way of handling this.
- Read the android documentation about [the dose mode](https://developer.android.com/training/monitoring-device-state/doze-standby.html)
- [Whitelist](https://stackoverflow.com/a/54982071/3248593) your app from being optimized. This is something you need to do for every manufacturer because some have [separate battery optimization](https://stackoverflow.com/a/56993037/3248593). You may use [third party libraries](https://github.com/pvsvamsi/AppKillerManager) to do so or do it yourself. If you use Flutter there is a [library](https://github.com/SachinGanesh/battery_optimization) that may help you.

Other mentionable notes
- Sensor quality differs between android devices. In some devices some sensors may not be available. The quality of the sensors might affect the accuracy of the positioning.
- You can call on `TT2.validateDevice(context)` to validate the device has the required sensors, not that this is not a measurment on the quality of the sensors.
```kotlin
TT2.validateDevice(context)) { error ->
    error?.let {
        // This means that the device is unsupported due to missing crucial sensors
    }
}
```
