{% highlight swift %}
let tt2 = TT2(with: "your server url", apiKey: "your api key")
tt2.initialize(clientId: yourClientId, positionKitParams: .retail) { [weak self] error in
    if error != nil {
        // Show error to user in case of any exception happened during initialization including network exception
    } else {
        // Safe to do the next steps
    }
}
{% endhighlight swift %}