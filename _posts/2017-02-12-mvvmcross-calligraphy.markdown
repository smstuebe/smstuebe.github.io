---
layout:     post
title:      "Calligraphy with MvvmCross"
subtitle:   "Calligraphy isn't working out of the box with MvvmCross. My new NuGet package MvvmCross.Calligraphy brings the simplicity back."
date:       2017-02-12 12:00:00
author:     "Sven-Michael Stübe"
header-img: "img/fonts-bg.jpg"
tags: [Xamarin, MvvmCross, Android, Calligraphy, Fonts, Label]
---

Calligraphy is a nice library that provides custom font handling in Android. I once had issues with the MvvmCross layout inflation and knew that using Calligraphy with MvvmCross will cause some problems and I pushed it onto my Blog todo list - but I was lazy. But this week I saw <a href="http://stackoverflow.com/questions/42105583/callygraphyxamarin-not-working-in-mvxappcompatactivity" target="_blank" onclick="return tol(this);">a question on stackoverflow</a>.

<h2 class="section-heading">TL;DR</h2>
If you just want to get Calligraphy working.

    Install-Package MvvmCross.Calligraphy
	
and modify your Setup.cs

{% highlight c# %}
public class Setup : MvxAndroidSetup
{
    protected override MvxAndroidBindingBuilder CreateBindingBuilder()
    {
        return new CalligraphyMvxAndroidBindingBuilder();
    }
} 
{% endhighlight %}

done. You **don't** need to override `AttachBaseContext`. And you can setup calligraphy via `CalligraphyConfig.InitDefault` as documented.


<h2 class="section-heading">Problem</h2>

MvvmCross and Calligraphy are using the same approach for two different things. MvvmCross uses a custom LayoutInflater to provide xml based data binding and Calligraphy uses a custom LayoutInflater that sets the font to TextViews at inflation time. And if you use MvvmCross, the MvvmCross inflater wins.

<h2 class="section-heading">Why not use Calligraphy Xamarin?</h2>

There is already the NuGet package / Xamarin component called <a href="https://developer.xamarin.com/guides/xamarin-forms/effects/creating/" target="_blank" onclick="return tol(this);">Calligraphy Xamarin</a>. But it doesn't offer the non public class `CalligraphyFactory` that you need for the fix. If you install it and follow the setup instructions, you will notice, that it won't have any effect.


<h2 class="section-heading">How does it work?</h2>

The package provides a custom `MvxAndroidViewBinder` called `CalligraphyMvxAndroidViewBinder` that applies calligraphy changes for every layout that is inflated with `BindingInflate` (which should be basically every view). All what's left, is to put the pieces together. Use `CalligraphyMvxAndroidViewFactory` to construct it and `CalligraphyMvxAndroidBindingBuilder` to return the factory in `CreateAndroidViewBinderFactory()`. In your code you only have to return a new `CalligraphyMvxAndroidBindingBuilder` in your Setup's `CreateBindingBuilder()` method.  

{% highlight c# %}
public class Setup : MvxAndroidSetup
{
    protected override MvxAndroidBindingBuilder CreateBindingBuilder()
    {
        return new CalligraphyMvxAndroidBindingBuilder();
    }
} 
{% endhighlight %}

<img src="/img/mvvmcross-calligraphy.jpg" style="margin:0 auto; cursor: pointer;"/>

<h2 class="section-heading">Conclusion</h2>

There is a nice side effect, using the factory directly. You don't have to wrap every context (Activity, Fragment) with the `CalligraphyContextWrapper`, because the binding inflater is used globally :) Yay! <a href="https://www.nuget.org/packages/MvvmCross.Calligraphy/" target="_blank" onclick="return tol(this);">MvvmCross.Calligraphy on NuGet</a>

<b>Code</b>

The code is available on github <i class="fa fa-github"></i><a href="https://github.com/smstuebe/mvvmcross-calligraphy" target="_blank" onclick="return tol(this);">mvvmcross-calligraphy</a>. It includes a sample project, too.

<center>If you like the answer, I'd appreciate an upvote ;) <br><a href="http://stackoverflow.com/a/42184315/1489968" target="_blank" title="Answer" onclick="return tol(this);">http://stackoverflow.com/a/42184315/1489968</a><br>Thanks!</center>

<br>
<small>[Background Photo](https://flic.kr/p/fJNKB "Photo") by [Karen](https://www.flickr.com/photos/56832361@N00/ "Karen") / [CC BY](http://creativecommons.org/licenses/by/2.0/ "CC BY")</small>
<br>
<small>Found a typo? Send me a pull request!</small>
