---
layout:     post
title:      "Awaitable Animations on Android"
subtitle:   "A tiny extension method allows you to await Android animations"
date:       2016-09-20 00:00:00
author:     "Sven-Michael Stübe"
header-img: "img/home-bg.jpg"
tags: [Xamarin, Android, Animation, async, await]
---

Xamarin has done a great job on bringing async/await to animations on iOS (as Kerry <a href="http://kerry.lothrop.de/animation-fun/" target="_blank">shows on his blog</a>) and on <a href="https://blog.xamarin.com/creating-animations-with-xamarin-forms/" target="_blank">Xamarin.Forms</a>. For my Fingerprint Plugin I was playing around with some animations on Android and I expected to find a similar animation extension - but I did not (If I've overseen something, tweet me). But, awaiting animations is pretty easy with a small extension method.

<h3>Problem</h3>
I want to colorize a icon, animate it and remove the color after the animation. In another use case I want to animate the icon and close the dialog afterwards.

<h3>Solution</h3>

The Android way would be to implement a `IAnimatorListener`, set the color in its `OnAnimationStart` method and reset it in `OnAnimationEnd`. But with this approach the code isn't readable linearly and you would have to implement a `IAnimatorListener` for each use case. I wanted to write my Animation like this:

{% highlight c# %}
_icon.SetColorFilter(NegativeColor);
var small = ObjectAnimator.OfFloat(_icon, "translationX", -10f, 10f);
small.SetDuration(500);
small.SetInterpolator(new CycleInterpolator(5));
await small.StartAsync();
_icon.ClearColorFilter();
{% endhighlight %}

You only need these two classes and are ready to animate Android with C# style.

**Extension**

The `AnimatorExtensions` only contains the `StartAsync` method that you should use to start a animation. It adds a `TaskAnimationListener` as listener, starts the animation and returns the a Task. The method can be used with all Animator classes (e.g. `ObjectAnimator, AnimatorSet, ValueAnimator`)

{% highlight c# %}
public static class AnimatorExtensions
{
    public static Task StartAsync(this Animator animator)
    {
        var listener = new TaskAnimationListener();
        animator.AddListener(listener);
        animator.Start();
        return listener.Task;
    }
}
{% endhighlight %}


**TaskAnimationListener**

The `TaskAnimationListener` implements `IAnimatorListener` and uses a `TaskCompletionSource` to convert the events `OnAnimationCancel` and `OnAnimationEnd` to the Canceled ot Completed states of a Task.

{% highlight c# %}
public class TaskAnimationListener : Java.Lang.Object, Animator.IAnimatorListener
{
    private readonly TaskCompletionSource<int> _tcs;
    public Task Task => _tcs.Task;

    public TaskAnimationListener()
    {
        _tcs = new TaskCompletionSource<int>();
    }

    public void OnAnimationCancel(Animator animation)
    {
        _tcs.TrySetCanceled();
    }

    public void OnAnimationEnd(Animator animation)
    {
        _tcs.TrySetResult(0);
    }

    public void OnAnimationRepeat(Animator animation)
    {
    }

    public void OnAnimationStart(Animator animation)
    {
    }
}
{% endhighlight %}

**Result**

<img src="/img/android-animation.gif" />

<small>Found a typo? Send me a pull request!</small>
