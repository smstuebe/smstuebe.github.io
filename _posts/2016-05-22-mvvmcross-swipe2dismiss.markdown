---
layout:     post
title:      "Swipe to dismiss with MvvmCross"
subtitle:   "This is how you can implement swipe to dismiss with MvvmCross"
date:       2016-05-22 00:00:00
author:     "Sven-Michael Stübe"
header-img: "img/home-bg.jpg"
tags: [Xamarin, MvvmCross, RecyclerView, UITableView, swipe to dismiss]
---

I recently had to implement swipe to dismiss (or delete). The items should get dismissed immediately without a delete button. For Android I used a `MvxRecyclerView` and on iOS the good old `UITableView`. The example code is available on <i class="fa fa-github"></i><a href="https://github.com/smstuebe/mvvmcross-examples/tree/master/Swipe2Dismiss" target="_blank">github</a>. 

The impulse for writing this blog post was a tweet by @waniste, who thought it is uncomfortable to implement it with a `MvxRecyclerView`. I had to convince him of the opposite, because MvvmCross is awsome and not uncomfortable :P

<blockquote class="twitter-tweet" data-lang="de"><p lang="en" dir="ltr">Seems like implementing &quot;SwipeToDismiss&quot; functionality for MvxRecyclerView isn&#39;t very comfortable. Will dig into it another day. <a href="https://twitter.com/hashtag/MvvmCross?src=hash">#MvvmCross</a></p>&mdash; Stefan (@waniste) <a href="https://twitter.com/waniste/status/734418485875507204">22. Mai 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<h3>Core</h3>
**FirstViewModel**

The ViewModel for our view creates the `MyItemViewModel`s and passes a `deleteCommand` that calls `DeleteItem`. `deleteCommand` is a command that takes a `MyItemViewModel` as parameter. This is the item that should get deleted when the command gets executed. `InvokeOnMainThread` is needed, because of iOS.

{% highlight c# %}
public class FirstViewModel 
    : MvxViewModel
{
    public ObservableCollection<MyItemViewModel> MyItems { get; } = new ObservableCollection<MyItemViewModel>();

    public FirstViewModel()
    {
        var deleteCommand = new MvxCommand<MyItemViewModel>(DeleteItem);
        MyItems.Add(new MyItemViewModel(deleteCommand) {Text = "Swipe me please!"});
    }

    private void DeleteItem(MyItemViewModel item)
    {
        InvokeOnMainThread(() => MyItems.Remove(item));
    }
}
{% endhighlight %}

**MyItemViewModel**

The `MyItemViewModel` is simple, too. It contains only a Text property, and a `Delete()` method, that should be called when the swipe gesture has been executed.

{% highlight c# %}
public class MyItemViewModel
{
    public MvxCommand<MyItemViewModel> DeleteCommand { get; }
    public string Text { get; set; }

    public MyItemViewModel(MvxCommand<MyItemViewModel> deleteCommand)
    {
        DeleteCommand = deleteCommand;
    }

    public void Delete()
    {
        DeleteCommand?.Execute(this);
    }
}
{% endhighlight %}


<h3>Android</h3>
Given you have setup a MvxRecyclerView, then it's pretty easy to implement it. All you need is to inherit a class from `ItemTouchHelper.SimpleCallback` and call the `Delete` method of your ItemViewModel.

{% highlight c# %}
public class Swipe2DismissTouchHelperCallback : ItemTouchHelper.SimpleCallback
{
    public Swipe2DismissTouchHelperCallback(IntPtr javaReference, JniHandleOwnership transfer) : base(javaReference, transfer)
    {
    }

    public Swipe2DismissTouchHelperCallback() : base(0, ItemTouchHelper.Left | ItemTouchHelper.Right)
    {
    }

    public override bool OnMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, RecyclerView.ViewHolder target)
    {
        return false;
    }

    public override void OnSwiped(RecyclerView.ViewHolder viewHolder, int direction)
    {
        var holder = (MvxRecyclerViewHolder)viewHolder;
        var vm = (MyItemViewModel)holder.DataContext;
        vm.Delete();
    }
}
{% endhighlight %}

In the constructor we call the base constructor with the allowed directions (`ItemTouchHelper.Left | ItemTouchHelper.Right`). In this example you can move the items left and right and you are not allowed to drag it. For the same reason, we simply return `false` in `OnMove`. 

In `OnSwiped`, the callback that is called, when the swipe gesture has been recognized, we have to call `Delete()` on the currently swiped item. The `MvxRecyclerView` is implementing the ViewHolder pattern. This means the `DataContext` of the holder is our ViewModel.

Finally, we have to attach it to our recycler view via an `ItemTouchHelper`.

{% highlight c# %}
var cardView = FindViewById<MvxRecyclerView>(Resource.Id.recyclerView);
var itemTouchHelper = new ItemTouchHelper(new Swipe2DismissTouchHelperCallback());
itemTouchHelper.AttachToRecyclerView(cardView);
{% endhighlight %}

<h3>iOS</h3>
iOS has a built in swipe to dismiss mechanism (`UITableView.CommitEditingStyle(...)`). Unfortunately it shows a delete button when swiped, that has to be pressed to actually delete the item. But we want to delete the item immediately. We have to implement our own `UIPanGestureRecognizer` for it.

Somewhere in our `UITableViewCell`, we attach it to our cell.

{% highlight c# %}
var deleteRecognizer = new UIPanGestureRecognizer(SwipeHandler) { ShouldBegin = ShouldBegin };
AddGestureRecognizer(deleteRecognizer);
{% endhighlight %}

Then we need to implement `SwipeHandler` that gets called for each swipe interaction of the recognizer.

{% highlight c# %}
private void SwipeHandler(UIPanGestureRecognizer recognizer)
{
    if (recognizer.State == UIGestureRecognizerState.Began)
    {
        _originalCenter = Center;
    }
    else if (recognizer.State == UIGestureRecognizerState.Changed)
    {
        var translation = recognizer.TranslationInView(this);
        var x = Math.Min(_originalCenter.X + translation.X, _originalCenter.X);
        Center = new CGPoint(x, Center.Y);
    }
    else if (recognizer.State == UIGestureRecognizerState.Ended)
    {
        if (Center.X > _originalCenter.X * DeleteThreshold)
        {
            Animate(0.2, () =>
            {
                Center = _originalCenter;
            });
        }
        else
        {
            AnimateAsync(0.2, () =>
            {
                Center = new CGPoint(-Frame.Width, Center.Y);
            }).ContinueWith(_ => ViewModel.Delete());
        }
    }
}
{% endhighlight %}

- When the swipe begins (`UIGestureRecognizerState.Began`) we save the original center of our cell.
- When its in between the gesture (`UIGestureRecognizerState.Changed`), we update the center of our cell. We translate it.
- When the gesture finishes, we have two options:
  - if the cell has not moved more than a defined way, we animate it back 
  - if the cell has moved more than the defined way, we animate it away and call `Delete()` afterwards

**Note:** `ViewModel` is the casted `DataContext` of the cell.

<small>Found a typo? Send me a pull request!</small>
