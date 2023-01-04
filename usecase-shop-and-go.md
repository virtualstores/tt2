---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

# Use case Mobile Shop & Go
### Table of Contents
- [Use case Mobile Shop \& Go](#use-case-mobile-shop--go)
    - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [TT2 Initialization](#tt2-initialization)
  - [Initiate Store](#initiate-store)
  - [Start Shopping Trip](#start-shopping-trip)
    - [Init User Profile](#init-user-profile)
    - [Start Visit](#start-visit)
  - [Scan Item](#scan-item)

## Overview
<img align="top" src="res/usecases/Integration%20Overview.svg">

## TT2 Initialization
<img align="top" src="res/usecases/Initialization.svg">

To get the SDK ready to work first Call initialize method. This will prepare the SDK for all other purposes.

{% include android/code-sample-tt2-initialize.md %}

## Initiate Store
<img align="top" src="res/usecases/Initiate%20Store.svg">

When the user chooses a store you can initialize the selected store by calling:

{% include android/code-sample-tt2-init-store.md %}

## Start Shopping Trip
<img align="top" src="res/usecases/Start%20Shopping%20Trip.svg">

### Init User Profile
{% include android/code-sample-tt2-init-user.md %}

### Start Visit
{% include android/code-sample-tt2-start-visit.md %}

## Scan Item
<img align="top" src="res/usecases/Scan%20Item.svg">

{% include android/code-sample-tt2-on-scan-item.md %}