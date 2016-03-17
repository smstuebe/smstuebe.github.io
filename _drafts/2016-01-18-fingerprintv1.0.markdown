---
layout:     post
title:      "Fingerprint plugin for Xamarin 1.0 released"
subtitle:   "The first release of my Fingerprint plugin is now available."
date:       2016-01-18 12:00:00
author:     "Sven-Michael Stübe"
header-img: "img/post-bg-01.jpg"
---

<p>Intro.</p>



<h2 class="section-heading">What next?</h2>
The two main future things are the following:
<h3>Samsung Pass integration</h3>
Before Android Marshmallow, Samsung had its own fingerprint technology / library called *pass*. I will create a binding for this library. I'll try to
create an modular architecture that allows to use the Android and the Samsung implementation or just one of them. 

<h3>Feedback from James Montemagno</h3>
Thanks to James Montemagno for sending over some helpful feedback.
<h4>Namespaces</h4>
I think James is right with the suggestion to adjust the namespaces to fit more into his established Plugin world and that one of MvvMCross. Even in MvvMCross 4.0 the personal namespace parts (Cirrious) from Stuart got removed, because MvvMCross got alot input from the community. And think my plugin should follow. 
<h4>Get rid of current activity resolver</h4>
The idea behind SetCurrentActivityResolver was to be independent from other plugins. If I do the namespace refactoring, I think it's a logical consequence to use James' Current Activity plugin as well. And it will reduce the setup effort as well.
