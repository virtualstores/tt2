{% highlight kotlin %}
fun stopVisit() {
    TT2.analytics.stopCollectingHeatmap()
    TT2.analytics.stopVisit()
    TT2.navigation.stop()
}
{% endhighlight %}