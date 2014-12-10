---
layout: post
title:  "Repeating Bitmaps inside LayerLists"
date:   2011-10-15 00:00:00
categories: android
summary: "I’ve run into this particular issue several times now. It’s a bug in the Android SDK, which apparently has been fixed as of ICS (Ice Cream Sandwich)..."
---
I’ve run into this particular issue several times now. It’s a bug in the Android SDK, which apparently has been fixed as of ICS (Ice Cream Sandwich). When you place a <bitmap> inside a <layer-list>, it tends to do whatever it feels like doing in regards to repeating the bitmap. Sometimes it will follow your instructions, and sometimes it won’t.

To fix this, just set the repeat mode in code. Here’s a snippet that will set all your Bitmaps repeating in a LayerDrawable.

{% highlight java %}
private void setLayerDrawableBitmapsRepeating(LayerDrawable layerDrawable) {
    final int size = layerDrawable.getNumberOfLayers();
    for(int i = 0; i < size; i++) {
        Drawable drawable = layerDrawable.getDrawable(i);
        if(drawable instanceof BitmapDrawable) {
            ((BitmapDrawable) drawable).setTileModeXY(TileMode.REPEAT, TileMode.REPEAT);
        }
    }
}
{% endhighlight %}
