---
layout:     post
title:      "Fingerprint plugin for Xamarin released"
subtitle:   "The first release of my Fingerprint plugin is now available."
date:       2016-01-18 12:00:00
author:     "Sven-Michael Stübe"
header-img: "img/post-1-bg.jpg"
tags: [Xamarin, Fingerprint, Plugin]
---

I recently pushed the first stable version of my fingerprint plugin for xamarin to NuGet. It allows you to authenticate a user via the fingerprint sensor. The code is available on <i class="fa fa-github"></i>[github](https://github.com/smstuebe/xamarin-fingerprint "github"). For me it's just a project to play around with some interesting things (e.g. <i class="fa fa-github"></i>[FAKE](https://github.com/fsharp/FAKE "FAKE") build scripts) and build something useful.

<h2 class="section-heading">Give it a try</h2>
If you want to try it you have two options.
You can simply clone the repository and build and execute the sample applications, or integrate it directly into your own app via NuGet <span class="inline">[![NuGet](https://img.shields.io/nuget/v/SMS.Fingerprint.svg?label=NuGet)](https://www.nuget.org/packages/SMS.Fingerprint/) [![NuGet MvvMCross](https://img.shields.io/nuget/v/SMS.MvvmCross.Fingerprint.svg?label=NuGet MvvMCross)](https://www.nuget.org/packages/SMS.MvvmCross.Fingerprint/).</span>

<h2 class="section-heading">Testing on Simulators</h2>
You don't need to own a device with a fingerprint sensor to test this functionality. The simulators offer functionality to simulate fingerprint events.
<h3>iOS</h3>
![Controlling the sensor on the iOS Simulator](https://raw.githubusercontent.com/smstuebe/xamarin-fingerprint/master/doc/ios_simulator.png "Controlling the sensor on the iOS Simulator")

With the Hardware menu you can
<ul>
	<li>Toggle the enrolment status</li>
	<li>Trigger valid (<kbd>⌘</kbd> <kbd>⌥</kbd> <kbd>M</kbd>) and invalid (<kbd>⌘</kbd> <kbd>⌥</kbd> <kbd>N</kbd>) fingerprint sensor events</li>
</ul>

<h3>Android</h3>
* start the emulator (Android >= 6.0)
* open the settings app
* go to Security > Fingerprint, then follow the enrolment instructions
* when it asks for touch
 * open command prompt
 * `telnet 127.0.0.1 <emulator-id>` (`adb devices` prints "emulator-&lt;emulator-id&gt;")
 * `finger touch 1`
 * `finger touch 1`

Sending fingerprint sensor events for testing the plugin can be done with the telnet commands, too.

**Note for Windows users:**
You have to enable telnet: Programs and Features > Add Windows Feature > Telnet Client

<h2 class="section-heading">API</h2>
The API is described in the readme file of my <i class="fa fa-github"></i>[github repository](https://github.com/smstuebe/xamarin-fingerprint "github repository").

<h2 class="section-heading">Support & Limitations</h2>

The supported platforms are:
<ul>
	<li>Android >= 6.0</li>
	<li>iOS >= 8.0</li>
	<li>Windows UWP >= 10.0</li>
</ul>

The dialog on iOS is programmatically cancelable since iOS 9.0. All other platforms are "supported" with a default implementation where <code>IsAvailable</code> returns false and <code>AuthenticateAsync</code> returns a failed authentication result.

<h2 class="section-heading">What next?</h2>
The two main future things are the following:
<h3>Samsung Pass integration</h3>
Before Android Marshmallow, Samsung had its own fingerprint technology / library called *pass*. I will create a binding for this library. I'll try to
create an modular architecture that allows to use the Android and the Samsung implementation or just one of them. 

<h3>Feedback from James Montemagno</h3>
Thanks to James Montemagno for sending over some <i class="fa fa-comment-o"></i>[helpful feedback](https://github.com/smstuebe/xamarin-fingerprint/issues/8 "helpful feedback").
<h4>Namespaces</h4>
I think James is right with the suggestion to adjust the namespaces to fit more into his established Plugin world and that one of MvvMCross. Even in MvvMCross 4.0 the personal namespace parts (Cirrious) from Stuart got removed, because MvvMCross got alot input from the community. And think my plugin should follow. 
<h4>Get rid of current activity resolver</h4>
The idea behind <code>SetCurrentActivityResolver</code> was to be independent from other plugins. If I do the namespace refactoring, I think it's a logical consequence to use James' Current Activity plugin as well. And it will reduce the setup effort as well.
<h4>More properties</h4>
It may be useful to implement more properties like <code>HasSensor</code>, <code>IsEnrolled</code>, that tell you the state of the fingerprint sensor in more detail than <code>IsAvailable</code>.

<br>
<small>[Background Photo](https://flic.kr/p/DW1sP "Photo") by [c0t0s0d0](https://www.flickr.com/photos/c0t0s0d0/ "c0t0s0d0") / [CC BY](http://creativecommons.org/licenses/by/2.0/ "CC BY")</small>
<br>
<small>Found a typo? Send me a pull request!</small>
