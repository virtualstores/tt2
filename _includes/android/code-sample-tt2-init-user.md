{% highlight kotlin %}
fun initUser(user: CustomUserObject){
    TT2.userSettings.initializeUserProfile(user.ID) { tt2Profile, tt2NetworkException ->
        // If exception on initialze user profile, we can check if could try this request again
        tt2NetworkException?.let { networkException ->
            // Toast message, handle errors?
            when (networkException) {
                is TT2NetworkException.NetworkFailureException -> {
                    // optional retry - Create your own retry policy
                    // or - let user shop without positioning
                }
                else -> {
                    // no Retry - let user shop without positioning
                    letUserShopWithoutPositioning = true
                }
            }
        }
        ​
        // if no exception on userProfile, continue to start visit
        tt2Profile?.let {
            startVisit(user)
        }
    }
}
{% endhighlight %}