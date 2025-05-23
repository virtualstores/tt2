---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
description: This guide will help you to get started.
---

# iOS Quick Start
## Overview
- [iOS Quick Start](#ios-quick-start)
  - [Overview](#overview)
- [TT2 SDK](#tt2-sdk)
  - [Usecases](#usecases)
  - [Installation](#installation)
      - [Using as a dependency](#using-as-a-dependency)
  - [Setup](#setup)
  - [Navigation](#navigation)
  - [Position](#position)
  - [Analytics](#analytics)
- [Map SDK](#map-sdk)
  - [Installation](#installation-1)
    - [1. Create .netrc file](#1-create-netrc-file)
    - [2. Add dependency on ios-map package](#2-add-dependency-on-ios-map-package)
  - [Setup](#setup-1)
  - [MarkerController](#markercontroller)
  - [PathfindingController](#pathfindingcontroller)
    - [Guide to PathfindingController](#guide-to-pathfindingcontroller)
  - [ZoneController](#zonecontroller)
  - [CameraController](#cameracontroller)
  - [Demo apps](#demo-apps)

# TT2 SDK

## Usecases

[Shop & Go App](usecase-shop-and-go.html)

## Installation

#### Using as a dependency

`Using Swift 5.6`

1. Using Xcode 15 go to File > Add Package Dependencies...
2. Paste the project URL in search bar: [https://github.com/virtualstores/ios-sdk](https://github.com/virtualstores/ios-sdk)
3. Use release version `2.4.0`
4. Click on next and select the project target

If you have doubts, please, check the following links:

[How to use](https://developer.apple.com/videos/play/wwdc2019/408/)

[Creating Swift Packages](https://developer.apple.com/videos/play/wwdc2019/410/)

Make sure to import the SDK wherever you need to use it by: `import VSTT2`

Make sure to add the following row to your .plist file so background access is working. We need it for being able to run in the background otherwise the app will be suspended once the user exits or locks the phone. Which then will result in losing user position.

* For “Privacy - Location When In Use Usage Description"

In "Signing & Capabilities" tab of your project add "Background Modes" capability and select "Location updates"

## Setup
The following parameters needed for setup will be provided by Virtual Stores

&nbsp;1. Create TT2 object with serverUrl and apiKey. Intialize TT2 with clientId and `ParameterPackage.retail`

  {% include ios/code-sample-tt2-initialize.md %}
	
&emsp;&emsp;&ensp;When this step is done, you can get a list of the stores by calling: tt2.activeStores
    
&emsp;&emsp;&ensp;You can show the list of the stores to the user and choose one.

&nbsp;2. To initialize your store, call `tt2.initiateStore(store:, completion:)`

&emsp;&emsp;&ensp;When the user chose a store you can initialize the selected store by calling:
	
  {% include ios/code-sample-tt2-init-store.md %}

## Navigation
Navigation handles the TT2 positioning system. It's possible to start the positioning system in different ways. It's recommended to start with a QR code for the most accurate positioning. A user can also go to a starting point visualized on the map and hold the device in the indicated direction and press start. You can access the navigation functionalities by calling:

ScanLocations can be set up in the TT2 cms. Filter and find what start QR code has been scanned in Store object.

```swift
do {
  guard 
    let code = self.tt2.activeFloor?.scanLocations?.filter({ $0.type == .start }).first(where: { $0.code == "scanResult"})
  else { return }
  try self.tt2.navigation.start(code: code)
} catch {
  // Handle error
}
```

If a user scans a product or shelf we want to update the user position.

Get the position of the shelf and use it as input. Position determines where on the map the user will be synced to. SyncRotation determines if the direction of the positioning system should be updated. Should only be true if we are sure the input direction is correct. For example if a user scans a shelf label:

```swift
tt2.navigation.syncPosition(position: /*product or shelf position*/, syncRotation: false, forceSync: true)
```

When the positioning should stop just call tt2.navigation.stop()

```swift
tt2.navigation.stop()
```

## Position
To get the position of a item or a shelf you can use one of the following functions

For getting position by shelf call the following:

```swift
tt2.position.getBy(shelfName: "<shelf name>") { (itemPosition) in
  do {
    try self.tt2.navigation.syncPosition(position: itemPosition)
  } catch {
    // Handle error
  }
}
```
For getting position by barcode call the following:

```swift
tt2.position.getBy(barcode: "<scanned barcode>") { (item) in
  guard let itemPosition = item?.itemPosition else { return }
  do {
    try self.tt2.navigation.syncPosition(position: itemPosition)
  } catch {
    // Handle error
  }
}
```
For getting multiple positions for barcodes call the following:

```swift
tt2.position.getBy(barcodes: [<scanned barcodes>]) { (items) in
  // Use item postions
}
```

## Analytics

Analytics is used for posting position related data to a server that later can be used for analysis and decision making.

For analytics functionalities to work a visit need to be started first. It is recommended to start the visit when the navigation starts and to start collecting heatmap data on the start visit callback.



```swift
let device = UIDevice.current
let deviceInformation = DeviceInformation(id: device.name,
                                          operatingSystem: device.systemName,
                                          osVersion: device.systemVersion,
                                          appVersion: "1.0",
                                          deviceModel: device.modelName)
    
let tags: [String : String] = ["age": String, "gender": String, "userId": String]
    
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

During the visit the user will walk around on the map. In different scenarios a default trigger event will be built by the TT2 SDK. A class can listen to these events. Then add additional information to the trigger event and post it to a server.

* Subscribe to EventTrigger for getting events

  ```swift
  analyticsMessgeCancellable = tt2.events.messageEventPublisher
    .compactMap { $0 }
    .sink { [weak self] event in
      // Handle event as you want to display it
    }
  ```

After getting an event you need to call `tt2.analytics.addTriggerEvent(for: event)`

* Create your own events

  ```swift
  let trigger = TriggerEvent.CoordinateTrigger(point: CGPoint(x: 5.0, y: 10.0), radius: 5, type: .enter)
  let event = TriggerEvent(rtlsOptionsId: /*floor level id*/,
                          name: "Testing",
                          description: "Test description",
                          eventType: TriggerEvent.EventType.coordinateTrigger(trigger))
  self.tt2.events.add(event: event)
  ```

# Map SDK

## Installation

### 1. Create .netrc file

In this package we are using Mapbox v10. To be able to use this you need to create and add the following file in your home folder: `~/.netrc`, and edit the file to add your Mapbox Secret Token. It is used by Mapbox to authenticate your account.

You can find an example of this file [here](https://github.com/virtualstores/tt2/blob/main/ios/.netrc)

Contents of the `.netrc` file should match this
```
machine api.mapbox.com
login mapbox
password <secretkey> // Provided by Virtual Stores
```

### 2. Add dependency on ios-map package 

1. Using Xcode 15 go to File > Add Package Dependencies...
2. Paste the project URL in search bar: [https://github.com/virtualstores/ios-map](https://github.com/virtualstores/ios-map)
3. Use release version `2.1.0`
4. Click on next and select the project target

Make sure to import the SDK wherever you need to use it by: `import VSMap`

Import of `VSFoundation` is also required in order for `VSTT2` and `VSMap` to be able to communicate with each other.

## Setup
* Create a TT2Map and connect it to TT2
  ```swift
  let map = TT2Map()
  tt2.set(mapManager: map.manager)
  ```
* Create a TT2MapView and add it to your view
    
    {% include ios/tt2-map/code-sample-init.md %}

* Create a BaseMapController
	- Your token can be provided by Virtual Stores, otherwise you can find your access token here: [https://account.mapbox.com/access-tokens/](https://account.mapbox.com/access-tokens/)
	- Use MapOptions to customize the map

	```swift
	let mapController = BaseMapController(
      with: "<Your Public MapBox token>", 
      view: TT2MapView, 
      mapOptions: MapOptions, 
      stateOptions: StateOptions
    )
	```

* Connect Map SDK to TT2 SDK

	```swift
	tt2.set(map: mapController)
	```

## Usecases

[Single Item Wayfinding](mapsdk/usecase-single-item-wayfinding.html)
	
## MarkerController
Example:

```swift
class ViewController: UIViewController {
  private var cancellable = Set<AnyCancellable>()

  var tt2: ITT2?
  var mapController: IMapController?
	
  override func viewDidLoad() {
    super.viewDidLoad()

    bindPublishers()
  }

  func bindPublishers() {
    mapController?.marker.onMarkerClicked
      .compactMap { $0 }
      .sink(receiveValue: { (marker) in
        // Do stuff, i.e remove marker and goal from map. Display information about the marker
        self.mapController?.marker.remove(marker: marker)
        guard let goal = self.mapController?.path.sortedGoals.first(where: { $0.id == marker.id }) else { return }
        self.mapController?.path.remove(goal: goal, completion: { })
      }).store(in: &cancellable)
  }
	
  func createMarker(for barcode: String) {
    self.tt2?.position.getBy(barcode: barcode) { [weak self] (result) in
      switch result {
      case .success(let item):
        guard let itemPosition = item.itemPosition else { return }
        let id = item.externalId
        let marker = BaseMapMark(
          id: id,
          itemPosition: itemPosition,
          clusterable: false,
          defaultVisibility: true,
          focused: false,
          type: .text(id)
        )

        self?.mapController?.marker.add(marker: marker)

        // Function found in PathfindingController chapter
        self?.createGoal(id: id, itemPosition: itemPosition)
      case .failure(let error): // Handle error
      }
    }
  }
}
```

## PathfindingController

### [Guide to PathfindingController](./mapsdk/pathfinding-controller.html "Example")

## ZoneController
Example:

```swift
class ViewController: UIViewController {
  private var cancellable = Set<AnyCancellable>()

  var tt2: ITT2?
  var mapController: IMapController?

  override func viewDidLoad() {
    super.viewDidLoad()

    bindPublishers()
  }

  func bindPublishers() {
    self.mapController?.mapDataLoadedPublisher
      .sink(receiveCompletion: { error in
        Logger(verbosity: .error).log(message: "MapLoading: \(error)")
      }, receiveValue: { (loaded) in
        if loaded == true {
          self.mapController?.zone.showAll()
        }
      }).store(in: &cancellable)
  }
}
```

## CameraController
Example:

```swift
class ViewController: UIViewController {
  private var cancellable = Set<AnyCancellable>()

  var tt2: ITT2?
  var mapController: IMapController?

  override func viewDidLoad() {
    super.viewDidLoad()

    bindPublishers()
  }

  func bindPublishers() {
    self.mapController?.mapDataLoadedPublisher
      .sink(receiveCompletion: { error in
        Logger(verbosity: .error).log(message: "MapLoading: \(error)")
      }, receiveValue: { (loaded) in
        if loaded == true {
          self.mapController?.camera.updateCameraMode(with: .containMap)
        }
      }).store(in: &cancellable)
  }
}
```

## Demo apps

- [Passive Demo](https://github.com/virtualstores/ios-passive-demo-app)
- [Active Demo](https://github.com/virtualstores/ios-active-demo-app)
