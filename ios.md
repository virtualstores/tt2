# Code samples
## Overview
- [Installation](#Installation)
- [Setup](#Setup)
- [Navigation](#Navigation)
- [Position](#Position)
- [Analytics](#Analytics)

## Installation

###Using as a dependency

`Using Swift 5.5`

1. Using Xcode 11 go to File > Swift Packages > Add Package Dependency
2. Paste the project URL in search bar: [https://github.com/virtualstores/ios-sdk](https://github.com/virtualstores/ios-sdk)
3. Click on next and select the project target

If you have doubts, please, check the following links:

[How to use](https://developer.apple.com/videos/play/wwdc2019/408/)

[Creating Swift Packages](https://developer.apple.com/videos/play/wwdc2019/410/)

Make sure to import the SDK wherever you need to use it by: `import VSTT2`

Make sure to add these following rows to your .plist file so background access is working. We need it for being able to run in the background otherwise the app will be suspended once the user exits or locks the phone. Which then will result in losing user position.

* For "Privacy - Location Always and When In Use Usage Description" 
* For “Privacy - Location Always Usage Description”
* For “Privacy - Location When In Use Usage Description"

## Setup
1. Create TT2 object: `let tt2 = TT2()`
2. Initiate TT2 for user with serverUrl, apiKey, and clientId

	```
	tt2.initialize(with: /*your server url*/, apiKey: /*your api key*/, clientId: 1) { [weak self] error in
	    if error != nil {
	    	// Show error to user in case of any exception happened during initialization including network exception
	    } else {
	    	// Safe to do the next steps
	    }
	}
	```
	
	When this step is done, you can get a list of the stores by calling: tt2.activeStores
		
	You can show the list of the stores to the user and choose one.

3. To initialize your store, call tt2.initiateStore(store:, completion:)

	When the user chose a store you can initialize the selected store by calling:
	
	```
	tt2.initiateStore(store: store) { error in
		if error != nil {
	    	// Show error to user in case of any exception happened during initialization including network exception
	    } else {
	    	// Safe to do the next steps
	    }
	}
	```

## Navigation
Navigation handles the TT2 positioning system. It's possible to start the positioning system in different ways. It's recommended to start with a QR code for the most accurate positioning. A user can also go to a starting point visualized on the map and hold the device in the indicated direction and press start. You can access the navigation functionalities by calling:

ScanLocations can be set up in the TT2 cms. Filter and find what start QR code has been scanned in Store object.

```
do {
	guard let code = self.tt2.activeStore?.startScanLocations.first(where: { $0.code == "scanResult "}) else { return }
	try self.tt2.navigation.start(code: code)
} catch {
	// Handle error
}
```

If a user scans a product or shelf we want to update the user position.

Get the position of the shelf and use it as input. Position determines where on the map the user will be synced to. SyncRotation determines if the direction of the positioning system should be updated. Should only be true if we are sure the input direction is correct. For example if a user scans a shelf label:

```
tt2.navigation.syncPosition(position: /*product or shelf position*/, syncRotation: false, forceSync: true)
```

When the positioning should stop just call tt2.navigation.stop()

```
tt2.navigation.stop()
```

## Position
To get the position of a item or a shelf you can use one of the following functions

For getting position by shelf call the following:

```
tt2.position.getBy(shelfName: name) { (itemPosition) in
    do {
    	try self.tt2.navigation.start(startPosition: itemPosition.point)
    } catch {
    	// Handle error
    }
}
```
For getting position by barcode call the following:

```
tt2.position.getBy(barcode: "barcode") { (item) in
    guard let point = item.itemPosition?.point else { return }
    do {
        try self.tt2.navigation.start(startPosition: point)
    } catch {
        // Handle error
    }
}
```
For getting multiple positions for barcodes call the following:

```
tt2.position.getBy(barcodes: []) { (items) in
    // Use item postions
}
```

## Analytics

Analytics is used for posting position related data to a server that later can be used for analysis and decision making.

For analytics functionalities to work a visit need to be started first. It is recommended to start the visit when the navigation starts and to start collecting heatmap data on the start visit callback.



```
let device = UIDevice.current
let deviceInformation = DeviceInformation(id: device.name,
                                          operatingSystem: device.systemName,
                                          osVersion: device.systemVersion,
                                          appVersion: "1.0",
                                          deviceModel: device.modelName)
    
let tags: [String : String] = ["age": age, "gender": gender, "userId": userId]
    
tt2.analytics.startVisit(deviceInformation: deviceInformation, tags: tags) { (error) in
	if let error = error {
	  	// Handle error
	} else {
		do {
			// Start collecting heatmap data by calling the following:
			try tt2.analytics.startCollectingHeatMapData()
		} catch {
		  	// Handle error
		}
	}
}
```

During the visit the user will walk around on the map. In different scenarios a default trigger event will be built by the TT2 sdk. A class can listen on these events. Then add additional information ot the trigger event and post it to a server.

* Subscribe to EventTrigger for getting events

```
analyticsMessgeCancellable = tt2.analytics.evenManager.messageEventPublisher
  .compactMap({ $0 })
  .sink { [weak self] event in
  		// Handle event as you want to display it
  }
```

After getting an event you need to call `tt2.analytics.addTriggerEvent(for: event)`

* Create your own events

```
let trigger = TriggerEvent.CoordinateTrigger(point: CGPoint(x: 5.0, y: 10.0), radius: 5)
let event = TriggerEvent(rtlsOptionsId: /*floor level id*/, 
							  name: "Testing", 
							  description: "Test description", 
							  eventType: TriggerEvent.EventType.coordinateTrigger(trigger))
self.tt2.analytics.evenManager.addEvent(event: event)
```