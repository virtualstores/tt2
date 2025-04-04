{% highlight kotlin %}

// Authentication Settings
val tokenBased = AuthSettings.TokenBased(
    username = "your sdk username",
    password = "your sdk password"
)

val apiKey = AuthSettings.ApiKey(
    apiKey = "your apiKey",
)

// Connection Settings
val directConnection = TT2ConnectionSettings.Direct(
    centralServerUrl = "https://url.to.your.tt2.central.server.com",
    authType = AuthType.TOKEN_BASED or AuthType.API_KEY depending on your authentication settings.
)

val gatewayConnection = TT2ConnectionSettings.Gateway(
    baseUrl = "https://url.to.your.gateway.com",
    authType = AuthType.TOKEN_BASED or AuthType.API_KEY depending on your authentication settings.
)

{% endhighlight %}