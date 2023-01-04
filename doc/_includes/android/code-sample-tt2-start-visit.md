{% highlight kotlin %}
fun startVisit(user: CustomUserObject) {
    TT2.analytics.startVisit(
        deviceInformation = DeviceInformation(
            operatingSystem = "Android",
            osVersion = Build.VERSION.RELEASE,
            appVersion = BuildConfig.VERSION_NAME,
            Build.MODEL
        ),
        // More tags can be added for more ways of filtering all visits.
        // We recommended to use these tags as they are already
        // prepared in the CMS for data filtering.
        tags = mapOf(
            "userID" to user.ID,
            "userGender" to user.gender,
            "userAge" to user.age
        ),
        metaData = mapOf(
            "customTag" to "customData"
        )
    ) { exception ->
        // If exception on start visit, we can check if we can retry this request
        exception?.let {
            (it as? TT2NetworkException)?.let { networkException ->
                when (networkException) {
                    is TT2NetworkException.NetworkFailureException -> {
                        // optional retry - Create your own retry policy
                        // or - let user shop without positioning
                    }
                    else -> {
                        //no retry - let user shop without positioning
                    }
                }
            } ?: run {
                // no Retry - let user shop without positioning
            }
        } ?: run {
            // Exception is null, the user can shop with positioning in the background
            TT2.analytics.startCollectingHeatMapData()
        }
    }
}
{% endhighlight %}