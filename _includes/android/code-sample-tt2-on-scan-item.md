{% highlight kotlin %}
fun onScannedProduct(scannerInputString: String) {
    // if we failed to initiate the analytix or user profile we did not start the navigation,
    // therefore let's ignore sync on scanned products with a quick return
    if (letUserShopWithoutPositioning)
        return

    TT2.position.getByBarcode(scannerInputString) { item ->
        item?.itemPosition?.let { itemPosition ->
            TT2.navigation.syncPosition(
                itemPosition,
                syncAngle = false,
                syncPosition = true,
                uncertainAngle = true,
                withForce = false,
            )
            ​
            // Optionally report the event to track scanning events in the analytics.
            TT2.analytics.postScanEvent(
                ScanEvent(
                    itemPosition.identifier,
                    itemPosition.shelfId,
                    itemPosition.x,
                    itemPosition.y,
                    System.currentTimeMillis(),
                    ScanEvent.Type.SHELF
                )
            )
        }
    }
}
{% endhighlight %}