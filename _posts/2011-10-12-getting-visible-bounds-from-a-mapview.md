---
layout: post
title:  "Getting visible bounds from a MapView"
date:   2011-10-12 00:00:00
categories: android
summary: "MapView doesn’t have a getBounds() method, but using a couple of MapView’s other methods, it’s actually very easy to find the visible bounds..."
---
MapView doesn’t have a getBounds() method, but using a couple of MapView’s other methods, it’s actually very easy to find the visible bounds. MapView has the methods getCenter(), getLongitudeSpan(), and getLatitudeSpan(). By combining these methods we can get the visible bounds.

{% highlight java %}
private Rect getMapBounds(MapView mapView) {
    final GeoPoint mapCenter = mapView.getMapCenter();
    final int lngHalfSpan = mapView.getLongitudeSpan() / 2;
    final int latHalfSpan = mapView.getLatitudeSpan() / 2;

    return new Rect(
    mapCenter.getLongitudeE6() - lngHalfSpan, mapCenter.getLatitudeE6() - latHalfSpan,
    mapCenter.getLongitudeE6() + lngHalfSpan, mapCenter.getLatitudeE6() + latHalfSpan);
}
{% endhighlight %}
