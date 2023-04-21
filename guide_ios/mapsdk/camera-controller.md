---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
description: Guide to TT2 ios CameraController.
---

# Camera Controller
### Table of Contents
- [Camera Controller](#camera-controller)
    - [Table of Contents](#table-of-contents)
  - [Summary](#summary)
  - [Camera modes:](#camera-modes)
    - [ContainMap2D](#containmap2d)
    - [FollowUser3D](#followuser3d)
    - [CameraAutoReset](#cameraautoreset)

## Summary
The CameraController handles the controls for the camera modes in the map view.

Documentation: [CameraController](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-camera-controller/index.html)

Documetation: [CameraModes](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-camera-controller/-camera-modes/index.html)
<br/><br/>

## Camera modes:

<br/>

### ContainMap2D

<img align="top" src="../../res/ios/cameracontroller/cameramode-contain-map-2D.png" height="500" >

Example:
```swift
```
<br/><br/>

### FollowUser3D

<img align="top" src="../../res/ios/cameracontroller/cameramode-follow-user-3D-default.png" height="500" >
<img align="top" src="../../res/ios/cameracontroller/cameramode-follow-user-3D-custom.png" height="500" >

Example:
```swift
```
<br/><br/>

### CameraAutoReset
Setting this value in the camera will automatically reset to the last camera mode after x amount of milliseconds passed in to the function.

Docs: [`setAutoCameraResetDelay(delay: Long)`](https://virtualstores.github.io/tt2/android/tt2-domain/se.virtualstores.tt2_domain.map/-camera-controller/index.html#-1588215429%2FFunctions%2F-1461421708)

<video  height="500" preload="auto" controls autoplay muted loop>
  <source src="../../res/ios/cameracontroller/contain-map-2D-camera-reset.mp4" type="video/mp4">
</video>
<video  height="500" preload="auto" controls autoplay muted loop>
  <source src="../../res/ios/cameracontroller/follow-user-3D-camera-reset.mp4" type="video/mp4">
</video>

```swift
```