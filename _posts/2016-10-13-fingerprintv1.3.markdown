---
layout:     post
title:      "Fingerprint plugin 1.3.0 for Xamarin released"
subtitle:   "The fingerprint plugin now supports Samsung devices"
date:       2016-10-13 12:00:00
author:     "Sven-Michael Stübe"
header-img: "img/post-1-bg.jpg"
tags: [Xamarin, Fingerprint, Plugin]
---

I recently pushed the stable version 1.3.0 of my fingerprint plugin for xamarin to NuGet. The code is available on <i class="fa fa-github"></i>[github](https://github.com/smstuebe/xamarin-fingerprint "github").

If you want to try it you have two options.
You can simply clone the repository and build and execute the sample applications, or integrate it directly into your own app via NuGet <span class="inline">[![NuGet](https://img.shields.io/nuget/v/Plugin.Fingerprint.svg?label=NuGet)](https://www.nuget.org/packages/Plugin.Fingerprint/) [![NuGet MvvmCross](https://img.shields.io/nuget/v/MvvmCross.Plugins.Fingerprint.svg?label=NuGet MvvmCross)](https://www.nuget.org/packages/MvvmCross.Plugins.Fingerprint/).</span>

<h2 class="section-heading">Changes</h2>

<h3>Breaking</h3>
The property `IsAvailable` got replaced with `GetAvailabilityAsync()` which gives you a more detailed feedback why the fingerprint authentication is not available if it isn't available. The possible options are:

- Available: Fingerprint authentication can be used.
- NoImplementation: This plugin has no implementation for the current platform.
- NoApi: Operating system has no supported fingerprint API.
- NoPermission: App is not allowed to access the fingerprint sensor.
- NoSensor: Device has no fingerprint sensor.
- NoFingerprint: Fingerprint isn't set up.
- Unknown: An unknown, platform specific error occurred. 

If you are not interested in the reason, you can easily check the availability via `IsAvailableAsync()`.

<h3>Samsung Support</h3>
Samsung devices (even > Android 6.0) are using a own API for fingerprint authentication called Samsung Pass. The plugin supports it now. Authentication on pre Marshmallow Devices is possible, now. The used pass SDK version is 1.2.0, because 1.2.1 wasn't working on a Lollipop device. If you are interested how I bound the obfuscated library, have a look at <a href="https://github.com/smstuebe/xamarin-fingerprint/blob/master/src/samsung/readme.md" target="_blank">Samsung Binding</a>. Tweet me if you have some better ideas.

<h3>Animations</h3>
The built in Authentication dialog on Android gives animated feedback to the user. 
The following example shows a failed attempt followed by requesting the fallback.

<img src="/img/fp-fallback.gif" />


<h3>Localization</h3>
The reason text and button labels are now fully customizable. But there are some <a href="https://github.com/smstuebe/xamarin-fingerprint/blob/master/README.md#limitations" target="_blank">platform specific limitations</a>.

{% highlight c# %}
var dialogConfig = new AuthenticationRequestConfiguration("Beweise, dass du Finger hast!")
{
    CancelTitle = "Abbrechen",
    FallbackTitle = "Anders!"
};

var result = await Plugin.Fingerprint.CrossFingerprint.Current.AuthenticateAsync(dialogConfig);
{% endhighlight %}

<h3>Other changes</h3>

**.NET standard support**

All PCLs are now targeting .NET standard 1.0. The nuget configuration has been adjusted.

**Bugfixes**

Of course I fixed some bugs and did some minor improvements. If you are interested in more detail have a look at the <a href="https://github.com/smstuebe/xamarin-fingerprint/blob/master/doc/changelog.md" target="_blank">changelog</a>.

<h2 class="section-heading">What next?</h2>
I could test the samsung integration only on two devices and the standard implementation only on a simulator. I'd be happy to get some feedback/bug reports/feature request from you :) 

<br>
<small>[Background Photo](https://flic.kr/p/DW1sP "Photo") by [c0t0s0d0](https://www.flickr.com/photos/c0t0s0d0/ "c0t0s0d0") / [CC BY](http://creativecommons.org/licenses/by/2.0/ "CC BY")</small>
<br>
<small>Found a typo? Send me a pull request!</small>
