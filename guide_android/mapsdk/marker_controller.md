---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
description: Guide to TT2 Android MarkerController.
---

# Camera Controller
### Table of Contents
- [Camera Controller](#camera-controller)
    - [Table of Contents](#table-of-contents)
  - [Summary](#summary)
  - [MarkerController](#markercontroller)

## Summary
The MarkerController handles the controls for map marks in the map view.

Documentation: [MarkerController](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-marker-controller/index.html)

<img align="top" src="../../res/android/mapmark/mapmark-base-map-mark.png" height="500" >
<img align="top" src="../../res/android/mapmark/mapmark-text-box.png" height="500" >

<br/><br/>

## MarkerController

Example:
```kotlin
// Implement interface MarkerController.Listener
class MyMapFragment: Fragment(), MapListener, MarkerController.Listener {
    
    var mapController: MapController? = null

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        val mapOptions = MapOptions().apply {
            mapMark = MapOptions.MapMark().apply {
                focusSizeScale = 1.25f
            }
        }

        mapController = BaseMapController(
            binding.mapView,
            mapOptions)

        mapController.mapListener = this
        TT2.setMapController(mapController)
    }

    // the map is now fully loaded and it's now safe to start using it
    override fun onMapLoaded() {
        super.onMapLoaded()
        
        mapController.marker.addListener(this)
    }

    // using map marks
    fun addMarkToMap(data: YourData, itemPosition: IItemPosition) {
        
        // You can design and create your own map marks by extending :MapMark<T>, Comparable<MapMark<T>>
        // or use the included marks available in the map SDK.
        
        // the BaseMapMark class that are available in the SDK can display two different types of 
        // information, image from url or short text (max 2-3 characters)
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

        // the BaseTextBoxMapMark class that are available in the SDK can display longer text labels 
        val textBoxMark = BaseTextBoxMapMark(
            id = data.shelfId,
            position = itemPosition.point,
            floorLevelId = itemPosition.floorLevelId,
            data = data,
            // or mark with text:
            text = data.label
        )

        mapController.marker.addMark(mark)
    }

    // Handles the click events if a user clicks on rendered mark on the map
    override fun onMarkClick(mark: MapMark<out Any>) {
        (mark.data as? YourData)?.let {

        }
    }

    // Handles the click events if a user clicks on rendered cluster of marks on the map
    override fun onClusterClicked(marks: List<MapMark<out Any>>) {

    }

    
    // MapMarks with the MapMark.triggerRadius set will notify if the user 
    // location enters or exits the set radius. 
    override fun onMarkTriggerEnter(mark: MapMark<out Any>) {
        
    }

    override fun onMarkTriggerExit(mark: MapMark<out Any>) {
        
    }

```
<br/><br/>