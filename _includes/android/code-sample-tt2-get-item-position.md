{% highlight kotlin %}
fun getItemPosition(barcode: String) {
    
    // items can have either a itemPosition or a  zonePosition.
    // if it has an itemPosition that means the item has a known location on a specific shelf in the store.
    // if it has an zonePosition that means the item is known but does not have a specific shelf location in the store 
    // however it should be located wihtin the zone.

    // if both itemPosition and zonePosition is null the item has no known location.

    // callback version 
    TT2.position.getItemByBarcode(barcode) { item -> 
        item?.let {
            it.itemPosition
            it.zonePosition
        }
    }

    // suspended version 
    scope.lauch {
        TT2.position.getItemByBarcode(barcode)?.let { item ->
            item.itemPosition
            item.zonePosition
        }
    }
}
{% endhighlight %}