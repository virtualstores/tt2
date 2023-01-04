{% highlight kotlin %}
TT2.initStore(
context = //applicationContext,
storeId = //Your store Id
floorLevelId = // (optional) The floor level to start on. If not set the SDK will init the default floorLevel configured in the CMS
) { error ->
    if (error != null) {
        // Show error to user in case of any exception happened during initialization including network exception
    } else  {
        // Safe to do the next steps
    }
}
{% endhighlight %}