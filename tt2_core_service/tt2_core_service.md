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
        - [Connection params:](#connection-params)
      - [Optional TT2Params:](#optional-tt2params)
    - [ServiceParams](#serviceparams)
    - [Other params](#other-params)
    - [ADB Command cheat sheet](#adb-command-cheat-sheet)
  - [Example: Start intent with setup params](#example-start-intent-with-setup-params)



# TT2 Core Service 


## Downloads :arrow_down: 

- [TT2_Core_Service_v1.5.5](https://virtualstores-assets.s3.eu-north-1.amazonaws.com/tt2-core-service/apks/tt2-core-service-v1.5.5.apk)
- [TT2_Core_Service_v1.5.4](https://virtualstores-assets.s3.eu-north-1.amazonaws.com/tt2-core-service/apks/tt2-core-service-v1.5.4.apk)
- [TT2_Core_Service_v1.5.3](https://virtualstores-assets.s3.eu-north-1.amazonaws.com/tt2-core-service/apks/tt2-core-service-v1.5.3.apk)
- [TT2_Core_Service_v1.5.2](https://virtualstores-assets.s3.eu-north-1.amazonaws.com/tt2-core-service/apks/tt2-core-service-v1.5.2.apk)


# How to install the TT2 Core Service

Use the Android adb tool to install the `.apk` file.

`adb install tt2-core-service-app.apk`

# How to start the TT2 Core Service

The TT2 Core Service is started using adb start command along with setup parameters.

```
adb shell 'am start -n se.tt2.tt2service/.activity.MainActivity \
< additional params >
--ez autoFinish true'
```


On devices running <= Android 12 the tt2 core service can be started directly without starting the activity, however we recommend to user the "start activity" approach since some device configurations might restrict services to start in the background:
```
adb shell "am broadcast -a se.tt2.tt2service.START -n se.tt2.tt2service/.Start" \
< additional params >
```


The TT2 Core Service is configured to automatically start itself when the device has finished
booting up (i.e device restart) but needs to have been started with the setup parameters manually once for this to work.

## Setup params

The TT2 Core Service needs to be provied with params for it to work. The params are stored in a
database within the application and there is no need to provide all of the params if the TT2 Core
Service already is setup with the right params.

The params are categorised into TT2Params and ServiceParams.

### TT2Params

- `tt2Params` type `int` nullable = `true` set to `1` to activate params

#### Mandatory TT2Params:

- `clientId` type `long` nullable = `false`
- `storeId` type `long` nullable = `true` if externalStoreId is provided
- `externalStoreId` type `long` nullable = `true` if storeId is provided

##### Connection params:
The connection parameters tells the service how to establish the connection for the tt2 system.
It's specified into two parts, the `connectionType` and the `authType`. Depending on which types are declared the service expects additional params.

Connection params (GATEWAY, DIRECT):
- Type: GATEWAY
  - `connectionType` type `string` nullable = `false`
    - Example: `--es connectionType GATEWAY`
      -  `gatewayBaseUrl` type `string` nullable = `false`
         - Example:  `--es gatewayBaseUrl "https://gateway.example.se"`
- Type: DIRECT
  - `connectionType` type `string` nullable = `false`
    - Example: `--es connectionType DIRECT`
      - `centralServerUrl` type `string` nullable = `false`
        - Example: `--es centralServerUrl "https://central.example.se"`
  
Authentication params (TOKEN_BASED, API_KEY):
- Type: TOKEN_BASED
  - `authType` type `string` nullable = `false`
    - Example:  `--es authType TOKEN_BASED`
      - `username` type `string` nullable = `false`
        - Example: `--es username "userName"`
      - `password` type `string` nullable = `false`
        - Example: `--es password "passwordXYZ"`
  
- Type: API_KEY
  - `authType` type `string` nullable = `false`
    - Example: `--es authType API_KEY`
      - `apiKey` type `string` nullable = `false`
        - Example: `--es apiKey "apiKeyXYZ"`

#### Optional TT2Params:

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

Example setup Command Using GATEWAY connection and TOKEN_BASED authentication:

``` shell
adb shell 'am start -n se.tt2.tt2service/.activity.MainActivity \
--ei tt2Params 1 \
--el clientId 1 \
--el storeId 1 \
--es connectionType GATEWAY \
--es gatewayBaseUrl "https://gateway.example.se/v1" \
--es authType TOKEN_BASED \
--es username "userName" \
--es password "Password" \
--ez debugMode true \
--ez autoFinish true'
```

Example setup Command Using DIRECT connection and API_KEY authentication
``` shell
adb shell 'am start -n se.tt2.tt2service/.activity.MainActivity \
--ei tt2Params 1 \
--el clientId 1 \
--el storeId 1 \
--es connectionType DIRECT \
--es centralServerUrl "https://central.example.se/v1" \
--es authType API_KEY \
--es apiKey "apiKeyXYZ" \
--ez debugMode true \
--ez autoFinish true'
```

<br/><br/>