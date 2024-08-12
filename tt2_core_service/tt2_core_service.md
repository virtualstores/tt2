---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
description: This guide will help you to get started.
---

- [TT2 Core Service](#tt2-core-service)
  - [Downloads :arrow\_down:](#downloads-arrow_down)
- [How to install the TT2 Core Service](#how-to-install-the-tt2-core-service)
- [How to start the TT2 Core Service](#how-to-start-the-tt2-core-service)
  - [Setup params](#setup-params)
    - [TT2Params](#tt2params)
      - [Mandatory TT2Params:](#mandatory-tt2params)
      - [Optional TT2Params:](#optional-tt2params)
    - [ServiceParams](#serviceparams)
    - [Other params](#other-params)
    - [ADB Command cheat sheet](#adb-command-cheat-sheet)
  - [Example: Start intent with setup params](#example-start-intent-with-setup-params)



# TT2 Core Service 


## Downloads :arrow_down: 

- [TT2_Core_Service_v1.4.2](https://virtualstores-assets.s3.eu-north-1.amazonaws.com/tt2-core-service/apks/tt2-core-service-v1.4.2.apk)
- [TT2_Core_Service_v1.4.1](https://virtualstores-assets.s3.eu-north-1.amazonaws.com/tt2-core-service/apks/tt2-core-service-v1.4.1.apk)
- [TT2_Core_Service_v1.4](https://virtualstores-assets.s3.eu-north-1.amazonaws.com/tt2-core-service/apks/tt2-core-service-v1.4.apk)
- [TT2_Core_Service_v1.3](https://virtualstores-assets.s3.eu-north-1.amazonaws.com/tt2-core-service/apks/tt2-core-service-v1.3.apk)
- [TT2_Core_Service_v1.2](https://virtualstores-assets.s3.eu-north-1.amazonaws.com/tt2-core-service/apks/tt2-core-service-v1.2.apk)
- [TT2_Core_Service_v1.0](https://virtualstores-assets.s3.eu-north-1.amazonaws.com/tt2-core-service/apks/tt2-core-service-v1.0.apk)


# How to install the TT2 Core Service

Use the Android adb tool to install the `.apk` file.

`adb install tt2-core-service-app.apk`

# How to start the TT2 Core Service

The TT2 Core Service is started using adb start command along with setup parameters.

On devices running <= Android 12 the tt2 core service can be started directly:

```
adb shell "am broadcast -a se.tt2.tt2service.START -n se.tt2.tt2service/.Start" \
< additional params >
```

On devices running Android 13 and above the tt2 core service needs to be started from an activity:

```
adb shell 'am start -n se.tt2.tt2service/.activity.MainActivity \
< additional params >
--ez autoFinish true'
```

If unsure of what Android version the device is running use start the service using the activity.

The TT2 Core Service is configured to automatically start itself when the device has finished
booting up but needs to have been started with the setup parameters manually once for this to work.

## Setup params

The TT2 Core Service needs to be provied with params for it to work. The params are stored in a
database within the application and there is no need to provide all of the params if the TT2 Core
Service already is setup with the right params.

The params are categorised into TT2Params and ServiceParams.

### TT2Params

- `tt2Params` type `int` nullable = `true` set to `1` to activate params

#### Mandatory TT2Params:

- `centralServerUrl` type `string` nullable = `false`
- `centralServerApiKey` type `string` nullable = `false`\
- `clientId` type `long` nullable = `false`
- `storeId` type `long` nullable = `true` if externalStoreId is provided
- `externalStoreId` type `long` nullable = `true` if storeId is provided

#### Optional TT2Params:

TT2 can be configured to use a proxy connection for it's resources making it easier to configure
firewall whitelist.
If opting in for this the following parameters are used to configure the proxy connections:

- `dataServerUrl` type `string` nullable = `true`
- `dataServerApiKey` type `string` nullable = `true`
- `mlModelServerUrl` type `string` nullable = `true`
- `mlModelServerApiKey` type `string` nullable = `true`
- `tt2ResourceUrl` type `string` nullable = `true`
- `tt2MLResourceUrl` type `string` nullable = `true`

TT2 can be configured to use specific trained models for certain environments, overriding default settings.
If opting in for this the following params are used to configure targets and specific versions to use:

- `tt2ModelTarget` type `int` nullable = `true`
- `tt2TargetMLModelVersion` type `int` nullable = `true` (works only if `tt2ModelTarget` is also specified) 
- `tt2TargetNLModelVersion` type `int` nullable = `true` (works only if `tt2ModelTarget` is also specified)

There are some tools that can be configured for debugging and some specific environment constraints:

- `debugMode` type `boolean` defaults to `false`
- `enableWorkManager` type `boolean` defaults to `true`
- `enableProxy` type `boolean` defaults to `false`

### ServiceParams

- `serviceParams` type `int` nullable = `true` set to `1` to activate params

- `scanIntentAction` type `string` nullable = `true`\
  ScanIntentAction is used to set the intent filter for device scan events. For Zebra devices
  utilizing DataWedge with intent broadcast for scan data should provide the specified intent action
  here to also make the TT2 Core Service receive the scan data.

TT2 Core Service can be configured to automatically display image based messages created in the
Location-based communication section of the TT2 CMS.

- `messageDialogEnabled` type `boolean` defaults to `false`
- `messageVibrationEnabled` type `boolean` defaults to `false`
- `messageSoundEnabled` type `boolean` defaults to `false`

### Other params

- `autoFinish` type `boolean` !important!\
  Used for automatically finishing the MainActivity when starting the service on devices running
  Android 13 and above. The TT2 Core Service app will start up, visible on the device, and close
  itself after it has started the background service.

### ADB Command cheat sheet

```
--<flag> key value
```

- `--es` configures `string` parameter
- `--ei` configures `int` parameter
- `--el` configures `long` parameter
- `--ez` configures `boolean` parameter
- `\` new line, makes large commands more readable (make sure to remove any trailing whitespaces
  after this)

## Example: Start intent with setup params

Example start Service Command

```
adb shell "am broadcast -a se.tt2.tt2service.START -n se.tt2.tt2service/.Start" \
--ei tt2Params 1 \
--es centralServerUrl "https://example-central-server.se" \
--es centralServerApiKey "example-api-key" \
--el clientId 12 \
--el storeId 62 \
--ei serviceParams 1 \
--es scanIntentAction com.example.SCAN
```

Example start Activity Command

```
adb shell 'am start -n se.tt2.tt2service/.activity.MainActivity \
--es centralServerUrl "https://example-central-server.se" \
--es centralServerApiKey "example-api-key" \
--el clientId 12 \
--el storeId 62 \
--es scanIntentAction com.example.SCAN \ 
--ez autoFinish true'
```

<br/><br/>