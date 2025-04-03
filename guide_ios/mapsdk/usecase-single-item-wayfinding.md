---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
description: This use case shows the TT2 integration flow for a Scan & Go moblie app.
---

# Use case - Single Item Wayfinding
### Table of contents

## Overview
Use this if you want TT2 Map to use a predetermined way of how to handle the map when wayfinding to a single item.

## TT2 Map Initalization

See first page on how to setup TT2 Map properly.

[Setup](../ios.html#setup-1)

In order to use this use case, make sure the `StateOptions` are properly configured when initalizing `BaseMapController`.

```swift
let mapController = BaseMapController(
    with: "<Your Public MapBox token>", 
    view: TT2MapView, 
    mapOptions: MapOptions, 
    stateOptions: .init(preset: .singleItemWayfinding)
)
```

Example for how to setup the use case:

```swift
var mapMark: BaseMapMark? 
func createMap() {
    guard let token = tt2.activeFloor.mapBoxToken else { return }
    let mapController = BaseMapController(
      with: token,
      view: mapView,
      mapOptions: .init(),
      stateOptions: .init(preset: .singleItemWayfinding)
    )
    bindPublishers() // Make sure to bind publishers before setting map controller for tt2
    tt2.set(map: mapController)
}

// Import Combine at the top of class
var cancellable: Set<AnyCancellable> = []
func bindPublishers() {
    map.mapDataLoadedPublisher
      .sink { (result) in
        switch result {
        case .finished: break
        case .failure(_): break
        }
      } receiveValue: { [weak self] (loaded) in
        guard
          loaded,
          let self = self,
          let mark = mapMark
        else { return }

        map.marker.add(marker: mark)
        map.path.set(goals: [mark.asGoal]) {}
        map.zone.hideAllLayers()
      }.store(in: &cancellable)
}
```