# TT2 SDK

The TT2 SDK provides a powerful indoor positioning system that doesn’t need any external hardware. It contains tools for accurate live positioning, position based analytics, interactive and customizable map, zone based messages and more.

# Installation

Add the SDK to your app level `build.gradle` file:

```
dependencies {
    ...
    implementation("se.virtualstores:tt2-android-sdk:${SDK_VERSION}")
    ...
}
```


# Using the SDK

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
