{% highlight swift %}
func onScannedProduct(scannerInputString: String) {
    // if we failed to initiate the analytics or user profile we did not start the navigation,
    // therefore let's ignore sync on scanned products with a quick return
    guard !letUserShopWithoutPositioning else { return }
    
    // if navigation is active sync position without affecting direction else sync with compass
    let type: SyncTypeEnum = self.tt2.navigation.isActive ? .normal(syncRotation: false) : .compass(forceSync: false)
    self.tt2.navigation.syncPosition(identifier: scannerInputString, type: type) { (result) in
        switch result {
        case .success(let item):
            // Optionally report the event to track scanning events in the analytics tool
            guard let position = item.itemPosition else { return }
            self.tt2.analytics.postScanEvents(position: position)
        case .failure(let error): // Handle error
        }
    }
}
{% endhighlight swift %}