
# Code samples
## Installation

Add the SDK to your app level `build.gradle` file:

```
dependencies {
    ...
    implementation("se.virtualstores:tt2-android-sdk:${SDK_VERSION}")
    ...
}
```


## Setup

1- To get the SDK ready to work first Call initialize method. This will prepare the SDK for all other purposes.

```
TT2.initialize(
            context = //applicationContext
            apiUrl = // The api url to connect to
            apiKey = // Your API key
            clientId = //Your client ID
            neuralNetFileName = // The name of the NeuralNet File name
            serviceNotificationIntent = // The intent which will be used to keep the SDK alive on background
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

  ```
  TT2.initStore(
            context = //applicationContext,
            storeId = //Your store Id
            floorLevelId = // The floor level to start navigation with
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
```
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

```

//IScanLocations can be set up in the TT2 csm. Filter and find what start qr code has been scanned. 
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

```
//Get the position of the shelf and use it as input.
//Position determines where on the map the user will be synced to. 
//SyncRotation determines if the direction of the positioning system should be updated. Should only be true if we are sure the input direction is correct.
//For example if a user scans a shelf label.
TT2.navigation.syncPosition(position = shelfPosition, syncRotation = false, forceSync = true)
 
```

When the positioning should stop just call tt2.navigation.stop()

```
TT2.navigation.stop()
```

## Analytics

Analytics is used for posting position related data to a server that later can be used for analysis and decision making.

For analytics functionalities to work a visit need to be started first.
It is recommended to start the visit when the navigation starts and to start collecting heatmap data on the start visit callback.

```
fun startNavigation(position: PointF, angle: Double) {
    TT2.navigation.start(position, angle)
    TT2.analytics.startVisit(
        deviceInformation = DeviceInformation(
            operatingSystem = "Android",
            osVersion = Build.VERSION.RELEASE,
            appVersion = BuildConfig.VERSION_NAME,
            Build.MODEL),
        tags = mapOf(
                //More tags can be added for more ways of filtering all visits. It's recommended to use these tags.
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

```
import se.virtualstores.tt2.androidsdk.Listener

class MapActivity : AppCompatActivity(), Listener{

    override fun onDefaultTriggerEvent(triggerEvent: TriggerEvent) {
        TT2.analytics.postEventData(triggerEvent)
    }
}

```

Trigger events can also be created and then listen on when they are triggered.

```
//set upp trigger events by adding them to the trigger manager
//In this example we set upp will trigger event that will trigger when a user comes inside a 5m radius of the shalf named "123"
TT2.position.getByShelfName("123") { // getting an IItemPosition from a shelf named "123"
    it?.let { position ->
        TT2.triggerEventManager.addTriggerEvent( //Adding the trigger event to the trigger event manager.
            TriggerEvent.Builder().apply { //Building the trigger event.
                setName("Coordinate Trigger")
                addTag("shelf", "123")
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
Extend your class with TriggerEventManager.Listener to listen on the created trigger events.
Set this as a trigger listener inside the trigger event manager.

```

extend your class to listen on trigger events with  TriggerEventManager.Listener
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

```

fun stopNavigation(){
    TT2.navigation.stop()
    TT2.analytics.stopVisit() 
    //stopVisit() will stop the collection heatmap data internaly. If the heatmap collection wants to be stopped alone TT2.analytics.stopCollctingHeatMapData() can be called.
}

```

