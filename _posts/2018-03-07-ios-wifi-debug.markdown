---
layout:     post
title:      "Xamarin.iOS WiFi debugging"
subtitle:   "How to free one USB port and debug your Xamarin.iOS Apps wirelessly."
date:       2018-03-07 12:00:00
author:     "Sven-Michael Stübe"
card-img:   "img/wifi-debug-paired-icon.png"
tags:       [Xamarin, iOS, WiFi, Debug, Visual Studio]
---

The latest Visual Studio update makes an Xcode iOS/tvOS debugging feature available for Xamarin developers. It's called WiFi debugging. This means that you can now debug your Xamarin.iOS Apps without having to plug your phone in via USB.

<h3>Requirements</h3>

Make sure you have installed at least these versions of the following software:

- Xcode 9.0
- macOS 10.12.4
- VS Mac 7.4 or VS 2017.1.6
- iOS 11.0 or tvOS 11.0

Your Mac and your device have to be connected to the same network.

<h3>Setup</h3>

Setting up your device for WiFi debugging is pretty easy. Just

- connect your iOS Device via USB
- open Xcode 
- navigate to the device manager (Window &gt; Devices and Simulators)
- select your device and check `Connect via network`
- disconnect the USB connection

<img src="/img/wifi-debug-paired.png" style="margin:0 auto; cursor: pointer;" />

<h3>Special to Xamarin</h3>

In its <a href="https://help.apple.com/xcode/mac/9.0/index.html?localePath=en.lproj#/devac3261a70" target="_blank" onclick="return tol(this);">trouble shooting guide</a>, Apple states, that you have to ensure that port `62078` is open. This port is used to connect the device to Xcode (make it visible in the device list). In addition to that you can configure the Debugger port in VS. The default port is `10000` and you can configure it in the project settings of your iOS project in the iOS Debug section. There, you can also force the debugger to attach via network even if your device is connected via USB.

<img src="/img/wifi-debug-ports.png" style="margin:0 auto; cursor: pointer;" />

<h3>Observations</h3>

WiFi Deploy/Debug works well. Unfortunately, the connection is flaky as soon as the device goes into sleep mode / turns off the display. 
Sometimes, I couldn't bring it back to work and had to open the device window of Xcode to trigger a new connection. The Visual Studio teams have done a good job integrating this requested feature. Unfortunately, the flaky connection currently prevents me from using it in production. But if you really need to free up your USB ports, you should have a look at this feature.

<!-- ((ip.src in {192.168.43.129 192.168.43.63}) || (ip.dst in {192.168.43.129 192.168.43.63})) && (tcp.port in {10000 53872 62078}) -->

<br>
<small>Found a typo? Send me a pull request!</small>