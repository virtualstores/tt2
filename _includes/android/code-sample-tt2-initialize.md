{% highlight kotlin %}
TT2.initialize(
context = //applicationContext
apiUrl = // The api url to connect to
apiKey = // Your API key
clientId = //Your client ID
// The intent which will be used by the foreground service running the positioning logic, it will also
// handle user interaction with the notification that will be displayed in the notification center when the app is in background.
serviceNotificationIntent =  Intent(context, ClassToReceiveIntent::class.java)
) { error ->
    if (error != null) {
        // Show error to user in case of any exception happened during initialization including network exception
    } else  {
        // Safe to do the next steps
    }
}
{% endhighlight %}