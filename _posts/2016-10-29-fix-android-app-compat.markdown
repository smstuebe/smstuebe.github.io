---
layout:     post
title:      "Fixing Android AppCompat build errors"
subtitle:   "How to avoid or escape the NuGet cache hell that causes build errors when building Xamarin.Forms or Xamarin.Android apps"
date:       2016-10-29 12:00:00
author:     "Sven-Michael Stübe"
header-img: "img/nuget-hell-bg.jpg"
tags: [Xamarin, Android, Xamarin.Forms, AppCompat]
---

Today we had an awesome Xamarin Dev Day in Munich. Unfortunately a lot of people, especially people (and MVPs :P) that have installed Xamarin freshly, had problems building the sample apps. If you just installed Xamarin or updated the Xamarin.Forms NuGet package or one of the Xamarin.Android packages and run into one of the listed error messages, reading further will probably relief your pain. 

<h2 class="section-heading">Errors</h2>

We have seen a variety of build errors. Here are a couple of them.

{% highlight text %}
Please install package: 'Xamarin.Android.Support.v4' available in SDK installer. Java library file ...\AppData\Local\Xamarin\Xamarin.Android.Support.v4\23.3.0.0\content\libs/internal_impl-23.3.0.jar doesn't exist.	
Please install package: 'Xamarin.Android.Support.Vector.Drawable' available in SDK installer. Android resource directory ...\AppData\Local\Xamarin\Xamarin.Android.Support.Vector.Drawable\23.3.0.0\content\./ doesn't exist.
Please install package: 'Xamarin.Android.Support.v7.RecyclerView' available in SDK installer. Android resource directory ...\AppData\Local\Xamarin\Xamarin.Android.Support.v7.RecyclerView\23.3.0.0\content\./ doesn't 
Please install package: 'Xamarin.Android.Support.Vector.Drawable' available in SDK installer. Java library file ...\AppData\Local\Xamarin\Xamarin.Android.Support.Vector.Drawable\23.3.0.0\content\classes.jar doesn't
Please install package: 'Xamarin.Android.Support.v4' available in SDK installer. Android resource directory ...\AppData\Local\Xamarin\Xamarin.Android.Support.v4\23.3.0.0\content\./ doesn't exist.	
Please install package: 'Xamarin.Android.Support.Design' available in SDK installer. Java library file ...\AppData\Local\Xamarin\Xamarin.Android.Support.Design\23.3.0.0\content\classes.jar doesn't exist.	
Please install package: 'Xamarin.Android.Support.v7.MediaRouter' available in SDK installer. Java library file ...\AppData\Local\Xamarin\Xamarin.Android.Support.v7.MediaRouter\23.3.0.0\content\classes.jar doesn't exist.
Please install package: 'Xamarin.Android.Support.Animated.Vector.Drawable' available in SDK installer. Java library file ...\AppData\Local\Xamarin\Xamarin.Android.Support.Animated.Vector.Drawable\23.3.0.0\content\classes.jar doesn't exist.	
Reason: File ...\AppData\Local\Xamarin\zips\2A3A8A6D6826EF6CC653030E7D695C41.zip is not a ZIP archive	
Please install package: 'Xamarin.Android.Support.v7.MediaRouter' available in SDK installer. Android resource directory ...\AppData\Local\Xamarin\Xamarin.Android.Support.v7.MediaRouter\23.3.0.0\content\./ doesn't exist.	
Unzipping failed. Please download https://dl-ssl.google.com/android/repository/android_m2repository_r29.zip and extract it to the ...\AppData\Local\Xamarin\Xamarin.Android.Support.Animated.Vector.Drawable\23.3.0.0\content directory.
Please install package: 'Xamarin.Android.Support.v7.CardView' available in SDK installer. Android resource directory ...\AppData\Local\Xamarin\Xamarin.Android.Support.v7.CardView\23.3.0.0\content\./ doesn't exist.
Please install package: 'Xamarin.Android.Support.v7.CardView' available in SDK installer. Java library file ...\AppData\Local\Xamarin\Xamarin.Android.Support.v7.CardView\23.3.0.0\content\classes.jar doesn't exist.
Please install package: 'Xamarin.Android.Support.v4' available in SDK installer. Java library file ...\AppData\Local\Xamarin\Xamarin.Android.Support.v4\23.3.0.0\content\classes.jar doesn't exist.	
Please install package: 'Xamarin.Android.Support.v7.MediaRouter' available in SDK installer. Java library file ...\AppData\Local\Xamarin\Xamarin.Android.Support.v7.MediaRouter\23.3.0.0\content\libs/internal_impl-23.3.0.jar doesn't exist.	
Please install package: 'Xamarin.Android.Support.v7.AppCompat' available in SDK installer. Android resource directory ...\AppData\Local\Xamarin\Xamarin.Android.Support.v7.AppCompat\23.3.0.0\content\./ doesn't exist.
Please install package: 'Xamarin.Android.Support.Animated.Vector.Drawable' available in SDK installer. Android resource directory ...\AppData\Local\Xamarin\Xamarin.Android.Support.Animated.Vector.Drawable\23.3.0.0\content\./ doesn't exist.
Please install package: 'Xamarin.Android.Support.v7.RecyclerView' available in SDK installer. Java library file ...\AppData\Local\Xamarin\Xamarin.Android.Support.v7.RecyclerView\23.3.0.0\content\classes.jar doesn't exist.
Please install package: 'Xamarin.Android.Support.v7.AppCompat' available in SDK installer. Java library file ...\AppData\Local\Xamarin\Xamarin.Android.Support.v7.AppCompat\23.3.0.0\content\classes.jar doesn't exist.
Please install package: 'Xamarin.Android.Support.Design' available in SDK installer. Android resource directory ...\AppData\Local\Xamarin\Xamarin.Android.Support.Design\23.3.0.0\content\./ doesn't exist.
{% endhighlight %}

or

{% highlight text %}
...\Resources\values\styles.xml
error APT0000: Error retrieving parent for item: No resource found that matches the given name 'Theme.AppCompat.Light.DarkActionBar'.
error APT0000: No resource found that matches the given name: attr 'colorAccent'.
error APT0000: No resource found that matches the given name: attr 'colorPrimary'.
error APT0000: No resource found that matches the given name: attr 'colorPrimaryDark'.
error APT0000: No resource found that matches the given name: attr 'windowActionBar'.
error APT0000: No resource found that matches the given name: attr 'windowNoTitle'.
error APT0000: Error retrieving parent for item: No resource found that matches the given name 'style/Widget.AppCompat.Light.ActionBar.Solid'.
error APT0000: No resource found that matches the given name: attr 'elevation'.
{% endhighlight %}

<h2 class="section-heading">Cause</h2>

After a package restore, Xamarin is downloading missing Android SDK packages into `%UserProfile%\AppData\Local\Xamarin\zips`. This will take a while (depending on your internet connection and the google servers) and the build seems frozen. If you are impatient and kill Visual Studio or cancel the build, you end up in a undefined state. If clean / rebuild doesn't fix it, you have to call the artillery. Unfortunately, deleting the packages, bin, obj folders isn't enough because NuGet and Xamarin are caching the packages. 


<h2 class="section-heading">Fix</h2>

- close visual studio
- open the console at the solution root (where `*.sln` and `packages` folder are located)
- Execute the following commands

{% highlight bat %}
REM ARTILLERY SCRIPT
REM delete bin and obj folders
for /d /r . %d in (obj) do @if exist "%d" rd /s /q "%d"
for /d /r . %d in (bin) do @if exist "%d" rd /s /q "%d"

REM delete packages folder
rd /s /q packages

REM delete Xamarin.Android packages from NuGet cache
for /d %d in ("%UserProfile%\.nuget\packages\Xamarin.Android.*") do @if exist "%d" rd /s /q "%d"

REM delete Xamarin.Android packages from Xamarin cache
for /d %d in ("%UserProfile%\AppData\Local\Xamarin\Xamarin.Android.*") do @if exist "%d" rd /s /q "%d"

REM delete broken downloads
rd /s /q "%UserProfile%\AppData\Local\Xamarin\zips"
{% endhighlight %}

- rebuild and wait

**Warning:** This will delete a bunch of folders silently. If you don't trust me, replace `rd /s /q` with `echo` and execute the commands and you will see the folders that will be deleted. If you want to confirm each deletion, remove the `/q` switch.


<br>
<small>[Background Photo](https://flic.kr/p/94qMp8 "Photo") by [Rob and Stephanie Levy](https://www.flickr.com/photos/robandstephanielevy/ "Rob and Stephanie Levy") / [CC BY](http://creativecommons.org/licenses/by/2.0/ "CC BY")</small>
<br>
<small>Found a typo? Send me a pull request!</small>
