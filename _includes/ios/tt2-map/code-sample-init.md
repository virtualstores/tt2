{% highlight swift %}
let mapView = TT2MapView(frame: CGRect(x: 0, y: 0, width: 300, height: 300))
view.insertSubview(mapView, at: 0)
    
// Add constraints (optional but recommended)
mapView.translatesAutoResizingMaskIntoConstraints = false
let horizontalConstraint = NSLayoutConstraint(item: mapView, attribute: .centerX, relatedBy: .equal, toItem: view, attribute: .centerX, multiplier: 1, constant: 0)
let verticalConstraint = NSLayoutConstraint(item: mapView, attribute: .centerY, relatedBy: .equal, toItem: view, attribute: .centerY, multiplier: 1, constant: 0)
let widthConstraint = NSLayoutConstraint(item: mapView, attribute: .width, relatedBy: .equal, toItem: nil, attribute: .notAnAttribute, multiplier: 1, constant: view.frame.size.width)
let heightConstraint = NSLayoutConstraint(item: mapView, attribute: .height, relatedBy: .equal, toItem: nil, attribute: .notAnAttribute, multiplier: 1, constant: view.frame.size.height)
view.addConstraints([horizontalConstraint, verticalConstraint, widthConstraint, heightConstraint])
{% endhighlight swift %}