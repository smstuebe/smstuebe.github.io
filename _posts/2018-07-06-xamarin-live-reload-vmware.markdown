---
layout:     post
title:      "Using Xamarin Live Reload with VMWare Fusion in offline land"
subtitle:   "Setup VMWare Fusion to enable Xamarin Live Reload using port forwarding instead of cloud services."
date:       2018-07-06 12:00:00
author:     "Sven-Michael Stübe"
card-img:   "img/live-reload-vmware-library-icon.png"
tags:       [Xamarin, iOS, Live Reload, VMWare Fusion, Visual Studio, Xamarin.Forms, macOS]
---

One of the latest big improvement to the Xamarin.Forms ecosystem is Xamarin LiveReload. It's the next step when you reach the limits of LivePlayer. It helps you quickly iterating during your UI developent.

For more information and installation intructions, see: <a href="https://docs.microsoft.com/en-us/xamarin/xamarin-forms/xaml/live-reload " target="_blank" onclick="return tol(this);">Xamarin Live Reload</a>.

<a href="https://twitter.com/ianvink " target="_blank" onclick="return tol(this);">Ian Vink</a> wrote a <a href="https://ianvink.wordpress.com/2018/07/06/live-reload-from-visual-studio-2017-for-windows-in-wmware-fusion-on-a-mac-xamarin/ " target="_blank" onclick="return tol(this);">blog post</a> about how to use Xamarin Live Reload in a MacOS + VMWare Fusion + Visual Studio 2017 on a Windows 10 VM environment. 

The major disadvantage of the described setup is: <b>It doesn't work without internet or setting up a local MQTT server</b>. 

That's why I want to describe the setup I've chosen on my machine. On my development machine I use port forwarding from MacOS to my Windows 10 VM. Here is what you have to do, if you live in a country like me where fast internet isn't available everywhere (Germany, not kidding!) or you are on a plane ✈️ or train 🚆.

<h3>Setup</h3>



1. ensure, your VM uses NAT as networking adapter type (`Virtual Machine -> Network Adapter -> NAT`)
2. get your VMs IP address (`Window -> Virtual Machine Library` or <kbd>⌘</kbd> <kbd>⇧</kbd> <kbd>L</kbd>) when your machine is running
   <img src="/img/live-reload-vmware-library.png" style="margin:0 auto; cursor: pointer;" />
3. open the terminal on your Mac
4. open the nat configuration file
```bash
sudo vim /Library/Preferences/VMware\ Fusion/vmnet8/nat.conf
```
5. find `[incomingtcp]`
6. add `1883 = 192.168.2.178:1883` (<b>replace the IP with the IP from step 2!</b>)
```bash
[incomingtcp]
# Use these with care — anyone can enter into your VM through these…
# The format and example are as follows:
#<external port number> = <VM’s IP address>:<VM’s port number>
1883 = 192.168.2.178:1883
```
7. exit vim 😜
8. restart VM networking
```bash
sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --stop
sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --start
```

You might have to restart your VM if Live Reload doesn't work the first time. 

**Bonus tip:** I'm using port forwarding for several other applications (like Windows Device Portal, local web servers, etc., too. So feel freee to add as many ports as you want :)

<br>
<small>Found a typo? Send me a pull request!</small>