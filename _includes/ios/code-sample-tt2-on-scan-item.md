{% highlight swift %}
func onScannedProduct(scannerInputString: String) {
    // if we failed to initiate the analytics or user profile we did not start the navigation,
    // therefore let's ignore sync on scanned products with a quick return
    if letUserShopWithoutPositioning {
        return
    }
    tt2.position.getBy(barcode: scannerInputString) { (item) in
        guard let position = item?.itemPosition else { return }
        let isActive = self.tt2.navigation.isActive
        if !isActive {
            // Sync position with compass start
            try? self.tt2.navigation.syncPosition(position: position)
        } else {
            // Sync position without affecting direction
            try? self.tt2.navigation.syncPosition(position: position, syncRotation: false, forceSync: true)
        }

        // Optionally report the event to track scanning events in the analytics.
        self.tt2.analytics.postScanEvents(position: position)
    }
}
{% endhighlight swift %}