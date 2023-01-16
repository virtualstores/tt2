{% highlight swift %}
func startVisit(user: CustomUserObject) {
    guard
        let appVersion = Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String,
        let buildNumber = Bundle.main.infoDictionary?["CFBundleVersion"] as? String,
        let age = user.age, 
        let gender = user.gender?.uppercased(), 
        let userId = user.userId
    else { fatalError("Missing User data") }

    tt2.analytics.startVisit(
        deviceInformation: DeviceInformation(
            id: UIDevice.current.name,
            operatingSystem: UIDevice.current.systemName,
            osVersion: UIDevice.current.systemVersion,
            appVersion: "\(appVersion) (\(buildNumber))",
            deviceModel: UIDevice.current.modelName
        ), 
        tags: ["age": String(age), "gender": gender, "userId": userId],
        metaData: ["customTag": "customData"]
    ) { (result) in
        switch result {
        case .success(let visitId):
            do {
                try tt2.analytics.startCollectingHeatMapData()
            } catch {
                // Catch error
            }
        case .failure(let error):
            // optional retry - Create your own retry policy
            // or - let user shop without positioning
        }
    }
}
{% endhighlight swift %}