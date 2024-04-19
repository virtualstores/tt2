---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
description: " "
---

```
@startuml
Title SDK Integration overview
hide footbox
actor Customer as cust
participant App as app
participant SDK as sdk
participant tt2Server as tt2

group Initialization [On application startup]
    cust -> app: Start app
    app --> app: TT2()
    app -> sdk: tt2.initialize()
    sdk -> tt2: getStores()
        note right: Retrieves the latest version of\n the stores regarding the client
end

group Initiate Store [When customer checks in to store]
    cust -> app: Check in to store
    app -> sdk: tt2.initiate()
end

group Start shopping trip [Can be started right after store initialization]
    app -> sdk: tt2.user.initializeUser(memberId)
    sdk -> tt2: getUser(memberId)
        note right: Retrieves the personal profile
    app -> sdk: tt2.analytics.startVisit()
    app -> sdk: tt2.analytics.startCollectingHeatmap()
end

group Scan Item
    cust -> app: Scans item
    app -> sdk: TT2.position.getByBarcode(Barcode)
    sdk --> app: itemPosition
    app -> sdk: TT2.navigation.sync(itemPosition)
        note right: The first item scanned will set the starting position
    app -> sdk: TT2.analytics.postScanEvent()
        note right: This is optional and used for statistical analysis
end

group End Visit [When customer has paid]
    app -> sdk: TT2analytics.stopCollectingHeatmap()
    app -> sdk: TT2.analytics.stopVisit()
    app -> sdk: TT2.navigation.stop()
end
@enduml
```