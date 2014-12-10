---
layout: post
title:  "Inset TextView shadows"
date:   2011-10-17 00:00:00
categories: android
summary: "MapView doesn’t have a getBounds() method, but using a couple of MapView’s other methods, it’s actually very easy to find the visible bounds..."
---
If you’re used to using Apple products, you’re probably familiar with the heavy use of inset shadows. It adds a bit of depth to the UI and can really make the screen look beautiful. Notice the white drop shadow in the title of the window.

![Finder inset text](/images/finder_inset_text.png)

I tend to do the same thing a lot for my Android apps. It’s incredibly simple to do. Here’s an example of a TextView with the same inset shadow.

{% highlight xml %}
<!-- Semi-opaque white inset shadow beneath the text -->
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    ...
    android:shadowColor="#88FFFFFF"
    android:shadowRadius="0.1"
    android:shadowDx="0"
    android:shadowDy="1" />

<!-- Semi-opaque black inset shadow above the text -->
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    ...
    android:shadowColor="#88000000"
    android:shadowRadius="0.1"
    android:shadowDx="0"
    android:shadowDy="-1" />
{% endhighlight %}

And here’s how it looks:

![Android inset shadows](/images/android_inset_text.png)

It should look great on all screen sizes. Don’t use the above code strictly, however. Mess around with the color values and shadowRadius value to get the exact effect you’re looking for.
