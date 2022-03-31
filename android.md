# Code samples
## Overview

- [Code samples](#code-samples)
  - [Overview](#overview)
  - [Current record time of implementation: 34 min](#current-record-time-of-implementation-34-min)
  - [Prerequisites](#prerequisites)
  - [Add SDK to your app](#add-sdk-to-your-app)
  - [Setup](#setup)
  - [Changing the floor](#changing-the-floor)
  - [Navigation](#navigation)
  - [Analytics](#analytics)
- [Map Library](#map-library)
  - [Add a the view to your layout](#add-a-the-view-to-your-layout)
    - [Example using the map view with a fragment](#example-using-the-map-view-with-a-fragment)
- [Known issues](#known-issues)
  - [Run in the background](#run-in-the-background)

## Current record time of implementation: 34 min
Integrating the SDK, initializing SDK, Store, Analytics and performing test round collecting heatmap data to be viewd in the dashboard on the CMS.

## Prerequisites
Install or update [Android Studio](https://developer.android.com/sdk) to its latest version.

Make sure that your project meets these requirements:

//Todo: check this info:
- Targets API level 19 (KitKat) or higher
- Uses Android 4.4 or higher


## Add SDK to your app

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

The latest [SDK_VERSION](https://link-to-release)
```gradle
dependencies {
    ... 
    implementation("se.virtualstores:tt2-android-sdk:${SDK_VERSION}")
    ...
}
```


## Setup

Declare the tt2 service in your app manifest file `AndroidManifest.xml`
```xml
<manifest>
    <application>

        . . .

        <service
            android:name="se.virtualstores.tt2.pub.positionkit.PositionKitService"
            android:enabled="true" />

        . . .

    </application>
</manifest>
```

1- To get the SDK ready to work first Call initialize method. This will prepare the SDK for all other purposes.

```kotlin
TT2.initialize(
            context = //applicationContext
            apiUrl = // The api url to connect to
            apiKey = // Your API key
            clientId = //Your client ID
            serviceNotificationIntent =  Intent(context, ClassToReceiveIntent::class.java) // The intent which will be used to keep the SDK alive on background, it will also handle user interaction with the notification that will be displayed in the notification center when the app is in background.
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
            floorLevelId = // The floor level to start navigation with. This is optional, if not set the SDK will init the default floorLevel configured in the CMS
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


## Navigation

Navigation handles the TT2 positioning system.
It's possible to start the positioning system in different ways.
It's recommended to start with a QR code for the most accurate positioning. A user can also go to a starting point visualized on the map and hold the device in the indicated direction and press start.
You can access the navigation functionalities by calling:

IScanLocations can be set up in the TT2 csm. Filter and find what start qr code has been scanned. 
```kotlin
TT2.activeStore.startScanLocations.find { iScanLocation ->
    iScanLocation.code == scanResult
}?.let{
    startNavigation(it)
}

fun startNavigation(startScanLocation: IScanLocation) {
    TT2.navigation.start(startScanLocation)
}
```

If a user scans a product or shelf we want to update the user position.

Get the position of the shelf and use it as input.
Position determines where on the map the user will be synced to. 
SyncRotation determines if the direction of the positioning system should be updated. Should only be true if we are sure the input direction is correct.
For example if a user scans a shelf label:
```kotlin
TT2.position.getByBarcode("<scannerInput>") { item -> 
    TT2.navigation.syncPosition(position = item.itemPosition, syncRotation = false, forceSync = true)            
}
```
When the positioning should stop just call tt2.navigation.stop()
```kotlin
TT2.navigation.stop()
```


## Analytics

Analytics is used for posting position related data to a server that later can be used for analysis and decision making.

For analytics functionalities to work a visit need to be started first.
It is recommended to start the visit when the navigation starts and to start collecting heatmap data on the start visit callback.

```kotlin
fun startNavigation(position: PointF, angle: Double) {
    TT2.navigation.start(position, angle)
    TT2.analytics.startVisit(
        deviceInformation = DeviceInformation(
            operatingSystem = "Android",
            osVersion = Build.VERSION.RELEASE,
            appVersion = BuildConfig.VERSION_NAME,
            Build.MODEL),
        tags = mapOf(
                //More tags can be added for more ways of filtering all visits. We recommended to use these tags as they are already prepared in the CMS for data filtering.
                "userID" to user.ID,
                "userGender" to user.gender,
                "userAge" to user.age 
            ),
        metaData = null
        ) {
        TT2.analytics.startCollectingHeatMapData()          
    }
}
```

During the visit the user will walk around on the map. In different scenarios a default trigger event will be built by the TT2 sdk.
A class can listen on these events. Then add additional information ot the trigger event and post it to a server.

```kotlin
import se.virtualstores.tt2.androidsdk.Listener

class MapActivity : AppCompatActivity(), Listener{

    override fun onDefaultTriggerEvent(triggerEvent: TriggerEvent) {

        // parse the trigger event to show it to the user. I.e in the event of an message event added from the CMS: 
         triggerEvent.metaData[TriggerEvent.defaultMetaData.type]?.let {
            when (it) {
                    TriggerEvent.DefaultMetaData.MessageType.SMALL.name -> {}
                    TriggerEvent.DefaultMetaData.MessageType.LARGE.name -> {}
         }

        // after receiving an event you can optionally remove it to avoid trigger it again during this visit
        TT2.triggerEventManager.removeTriggerEvent(triggerEvent)

        // For viewing analytics data of the messages we recommend that your post the trigger event to the anlaytics
        TT2.analytics.postEventData(triggerEvent.toMessageShownEvent())
    }
}
```

Trigger events can also be created and then listen on when they are triggered.

Setup trigger events by adding them to the trigger manager
In this example we set upp will trigger event that will trigger when a user comes inside a `5m radius` of the shalf named `"123"`
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
                        )
                    )
                    setTriggerEventData(
                        CoordinateTrigger(
                            x = position.point.x.toDouble(),
                            y = position.point.y.toDouble(),
                            radius = 2.0
                        )
                    )
            }.build()
        )
    }
}
```

Extend your class with TriggerEventManager.Listener to listen on the created trigger events.
Set this as a trigger listener inside the trigger event manager.

```kotlin
class MainActivity():  TriggerEventManager.Listener{
    
    TT2.triggerEventManager.setTriggerListener(this)
    
    fun onNewTriggerEvent(triggerEvent: TriggerEvent){
        //Show special offer for product on screen
        
        //Post trigger event
        TT2.analytics.postEventData(triggerEvent)
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



# Map Library

The latest [MAP_SDK_VERSION](https://link-to-release)
```gradle
dependencies {
    ... 
    implementation("se.virtualstores:lib-map:${MAP_SDK_VERSION}")
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

```kotlin


class MyFragment: Fragment(), MapListener, LifecycleListener, PathfindingController.Listener, MarkerController.OnMarkerClickListener {
    
    var mapController: MapController? = null

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        mapController = BaseMapboxController(
            binding.mapView,
            MapOptions())

        mapController.lifecycleListener = this
        TT2.setMapController(mapController)
    }

    // the map is now fully loaded and it's now safe to start using it
    override fun onMapLoaded() {
        super.onMapLoaded()
        
        // optional
        mapController.mapListener = this

        mapController.pathfinding.addListener(this)
        mapController.marker.addOnMarkerClickListener(this)
    
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
            // choose either 
            imageURL = data.imageUrl,
            text = data.label
        )

        mapController.marker.addMark(mark)
    }

    override fun onMarkClick(mark: MapMark<out Any>) {
        (mark.data as? YourData)?.let {

        }
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

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        mapController.onSaveInstanceState(outState)
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

# Known issues
## Run in the background
The application needs to do work continually and it is not acceptable for the Android OS to kill the application’s background services. Since the introduction of doze mode in Android Marshmallow we have seen a number of enterprise use cases where customers want to ensure their application continues to run even though the device wants to enter a power saving mode. So as a developer this is your responsiblity to keep your app alive and keep up with the latest changes of Android OS to make this possible. Here is a list that may help you do this but may not be all the best solutions to this problem. Let us know if you know a better way of handling this.
  - Read the android documentation about [the dose mode](https://developer.android.com/training/monitoring-device-state/doze-standby.html)
  - [Whitelist](https://stackoverflow.com/a/54982071/3248593) your app from being optimized. This is something you need to do for every manufacturer because some have [separate battery optimization](https://stackoverflow.com/a/56993037/3248593). You may use [third party libraries](https://github.com/pvsvamsi/AppKillerManager) to do so or do it yourself. If you use Flutter there is a [library](https://github.com/SachinGanesh/battery_optimization) that may help you.
  - Sensors quality differs between devices. In some devices some sensors may not be available.
