---
layout: post
title:  "Falling back on old SDK methods without reflection"
date:   2011-10-28 00:00:00
categories: android
summary: "As Android progresses as a platform, there will inevitably be methods added to the SDK. You’ll want to add some of these great new features while still supporting the old SDKs..."
---
As Android progresses as a platform, there will inevitably be methods added to the SDK. You’ll want to add some of these great new features while still supporting the old SDKs.

We can conditionally use these methods by getting the device’s SDK version and using the appropriate method, but this won’t actually work because although it may compile, the older device will throw an exception when it loads the class that uses that unknown method.

One way to fix this is via reflection, which works great. Reflection, however, can be resource intensive, and hard to maintain. You’ll have to declare the method as a string and pass in a method signature… gross.

Another way to do this builds on the first approach. Since a method isn’t loaded until the class it’s in is loaded, we can hide the method in another class. Here’s how it works:

{% highlight java %}
public class MyActivity extends ListActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

        // Check to see if this version of Android supports
        // ListView smooth scrolling
        if(Integer.parseInt(Build.VERSION.SDK) >= 8) {
            // Smooth scroll to position
            SmoothScrollMethodHolder.smoothScrollToPosition(getListView(), 5);
        }
        else {
            // Set postion
            getListView().setSelection(5);
        }
    }

    private class SmoothScrollMethodHolder {
        // Put method in a static method of a private class. This class
        // won't be loaded until you call this statically method.
        public static void smoothScrollToPosition(ListView listView, int position) {
            listView.smoothScrollToPosition(position);
        }
    }
}
{% endhighlight %}

This code is easy to read, easy to maintain, and comes with all the benefits that reflection strips away, like code completion. Of course, you don’t want to litter your code with the above, so you’ll probably want to put it into a utility class.

{% highlight java %}
public static class CompatibilityUtils {
    private static final int SDK_VERSION = Integer.parseInt(Build.VERSION.SDK);

    public static void smoothScrollToPosition(ListView listView, int position) {
        if(SDK_VERSION >= 8) {
            API9.smoothScrollToPosition(listView, position);
        }
        else {
            listView.setPosition(position);
        }
    }

    private static class API6 {
        ...
    }

    private static class API8 {
        public static void smoothScrollToPosition(ListView listView, int position) {
            listView.smoothScrollToPosition(position);
        }
    }

    private static class API9 {
        ...
    }

    private static class API10 {
        ...
    }

    private static class API14 {
        ...
    }
}
{% endhighlight %}

You can add all your compatibility methods to this class. The original code is now just:

{% highlight java %}
CompatibilityUtils.smoothScrollToPosition(getListView(), 5);
{% endhighlight %}
