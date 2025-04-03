---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
description: This use case shows the TT2 integration flow for a Scan & Go moblie app.
---

# Use case - Mobile Shop & Go
### Table of Contents
- [Use case Mobile Shop & Go](#use-case-mobile-shop--go)
    - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [TT2 Initialization](#tt2-initialization)
  - [Initiate Store](#initiate-store)
  - [Start Shopping Trip](#start-shopping-trip)
    - [Init User Profile](#init-user-profile)
    - [Start Visit](#start-visit)
  - [Scan Item](#scan-item)
  - [End Shopping Trip](#end-shopping-trip)

## Overview
<img align="top" src="../res/ios/usecases/shop&go/integration-overview.svg">

## TT2 Initialization
<img align="top" src="../res/ios/usecases/shop&go/initialization.svg">

To get the SDK ready to work first initialize `TT2()` and then call initialize method. This will prepare the SDK for all other purposes. Recomended to save TT2 variable in a singleton so that it will be easily accesible all over the app.

{% include ios/code-sample-tt2-initialize.md %}

## Initiate Store
<img align="top" src="../res/ios/usecases/shop&go/initiate-store.svg">

When the user chooses a store you can initialize the selected store by calling:

{% include ios/code-sample-tt2-init-store.md %}

## Start Shopping Trip
<img align="top" src="../res/ios/usecases/shop&go/start-shopping-trip.svg">

### Init User Profile
{% include ios/code-sample-tt2-init-user.md %}

### Start Visit
{% include ios/code-sample-tt2-start-visit.md %}

## Scan Item
<img align="top" src="../res/ios/usecases/shop&go/scan-item.svg">
{% include ios/code-sample-tt2-on-scan-item.md %}

## End Shopping Trip
<img align="top" src="../res/ios/usecases/shop&go/end-visit.svg">
{% include ios/code-sample-tt2-stop-visit.md %}

Alternatively call the convenience method `tt2.stop()` which will do the same as above function
