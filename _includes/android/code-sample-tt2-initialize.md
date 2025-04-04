{% highlight kotlin %}
TT2.initialize(
    context = //applicationContext,
    clientId = // your client ID,
    connectionSettings = // your TT2ConnectionSettings,
    authSettings = // your AuthSettings,
    serviceNotificationIntent = Intent(
        applicationContext,
        MyActivity::class.java
    ),
) { error ->
    if (error != null) {
        // Show error to user in case of any exception happened during initialization
    } else  {
        // SDK is now initialized.
    }
}
{% endhighlight %}