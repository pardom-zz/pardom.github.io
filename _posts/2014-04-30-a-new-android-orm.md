---
layout: post
title:  "A new Android ORM"
date:   2014-04-30 00:00:00
categories: android
summary: "Over the past five years I've invested a lot in making ActiveAndroid an easy and full-featured ORM for Android."
---
Over the past five years I've invested a lot in making [ActiveAndroid](http://www.activeandroid.com/) an easy and full-featured ORM for Android. I've also learned a lot over the past five years, enough to know that ActiveAndroid is not really efficient. I've tried to make it efficient in terms of use, but where it really fails is in performance.

The main problem with ActiveAndroid is that it does run-time reflection on your model classes, which is really expensive. Even worse, class members are reflected upon during load and save operations which may happen thousands of times per query. Even with all the caching that is done, performance is still limited by the massive amounts of reflection.

I've started a new project which aims to solve the run-time reflection problem, simplify features, and lean on the native Android APIs. The most important feature is the compile-time reflection which makes load and save opertaions just as fast as if you weren't using an ORM.

Introducing Ollie: [https://github.com/pardom/ollie](https://github.com/pardom/ollie)

Ollie is in alpha right now, but already includes the basic features you would need in an ORM. Give it a try and send in your feedback.
