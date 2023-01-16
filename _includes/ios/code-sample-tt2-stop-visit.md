{% highlight swift %}
func stopVisit() {
    tt2.analytics.stopCollectingHeatMapData()
    tt2.analytics.stopVisit()
    tt2.navigation.stop()
}
{% endhighlight swift %}