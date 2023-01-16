{% highlight swift %}
func initUser(user: CustomUserObject) {
    tt2.user.initializeUser(userId: user.ID) { (error) in
        if error != nil {
            // optional retry - Create your own retry policy
            // or - let user shop without positioning
            letUserShopWithoutPositioning = true
        } else {
            startVisit(user)
        }
    }
}
{% endhighlight swift %}