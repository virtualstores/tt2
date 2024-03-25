{% highlight swift %}
tt2.initiate(store: TT2Store) { (error) in
    if error != nil {
        // Show error to user in case of any exception happened during initialization including network exception
    } else {
        // Safe to do the next steps
    }
}
{% endhighlight swift %}