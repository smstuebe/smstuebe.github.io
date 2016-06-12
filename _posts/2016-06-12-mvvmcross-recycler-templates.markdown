---
layout:     post
title:      "RecyclerView TemplateSelector in MvvmCross"
subtitle:   "This is how you can show different layouts for items in a RecyclerView"
date:       2016-06-12 00:00:00
author:     "Sven-Michael Stübe"
header-img: "img/recycle-bg.jpg"
tags: [Xamarin, MvvmCross, RecyclerView, template selector]
---

There is a old tutorial from Stuart Lodge (Polymorphic lists in the MvvmCross tutorials) where he explains how to show different table cells for different types of ViewModels. This post will show you how to do the same with a `MvxRecyclerView`. The example code is available on <i class="fa fa-github"></i><a href="https://github.com/smstuebe/mvvmcross-examples/tree/master/RecyclerViewDifferentTemplates" target="_blank">github</a>. 

<h3>Problem</h3>

Sometimes there is the need to layout the items of RecyclerView differently. This is mostly the case, if your `ItemsSource` stores items of different classes.

**ViewModels**

For this example, we use the following ViewModels:

{% highlight c# %}
public class FirstViewModel : MvxViewModel
{
    public ObservableCollection<PetViewModelBase> Pets { get; }

    public FirstViewModel()
    {
        Pets = new ObservableCollection<PetViewModelBase>
        {
            new CatViewModel {Age = 10, LifesLeft = 6, Name = "Paul"},
            new CatViewModel {Age = 1, LifesLeft = 9, Name = "Horst"},
            new FishViewModel {Age = 1, Fins = 2, Name = "Nemo"},
            new CatViewModel {Age = 16, LifesLeft = 4, Name = "Erna"},
            new CatViewModel {Age = 5, LifesLeft = 6, Name = "Nougat"},
            new FishViewModel {Age = 12, Fins = 2, Name = "Marlin"},
            new FishViewModel {Age = 9, Fins = 5, Name = "Sharky"},
            new CatViewModel {Age = 7, LifesLeft = 1, Name = "Tirebiter"},
            new CatViewModel {Age = 8, LifesLeft = 6, Name = "Moritz"},
            new FishViewModel {Age = 2, Fins = 5, Name = "Barsch"},
            new FishViewModel {Age = 4, Fins = 20, Name = "Finny"},
            new FishViewModel {Age = 8, Fins = 3, Name = "Klaus"},
            new FishViewModel {Age = 1, Fins = 1, Name = "Pirat"},
            new CatViewModel {Age = 3, LifesLeft = 9, Name = "Mickey"}
        };
    }
}

// Items 

public abstract class PetViewModelBase
{
    public string Name { get; set; }
    public int Age { get; set; }
}

public class CatViewModel : PetViewModelBase
{
    public int LifesLeft { get; set; }
}

public class FishViewModel : PetViewModelBase
{
    public int Fins { get; set; }
}
{% endhighlight %}

<h3>Solution</h3>

Luckily MvvmCorss includes a neat mechanism for this since version **4.1.5**. It's called `TemplateSelector` and defined in the interface `IMvxTemplateSelector`. The purpose of this interface is to map your item to a layout id based on a rule that you can define. 

**Selecting by ViewModel type**

In this scenario, we want to show different layouts for cats (`CatViewModel`) and fishes (`FishViewModel`).

{% highlight c# %}
public class TypeTemplateSelector : IMvxTemplateSelector
{
    private readonly Dictionary<Type, int> _typeMapping;

    public TypeTemplateSelector()
    {
        _typeMapping = new Dictionary<Type, int>
        {
            {typeof(CatViewModel), Resource.Layout.pet_cat},
            {typeof(FishViewModel), Resource.Layout.pet_fish}
        };
    }

    public int GetItemViewType(object forItemObject)
    {
        return _typeMapping[forItemObject.GetType()];
    }

    public int GetItemLayoutId(int fromViewType)
    {
        return fromViewType;
    }
}
{% endhighlight %}

Therefore we create a lookup table `_typeMapping` and use it in `GetItemViewType` that is called for each item of the list. The parameter `forItemObject` is the current item for which we should return the view type. We return the layout id directly, because we have one layout id per view type. If you want to know more about the purpose of view types have a look at: <a href="https://developer.android.com/reference/android/widget/Adapter.html#getItemViewType(int)" target="_blank">Adapter.getItemViewType(int)</a>.

**Selecting by a property value**

In our second scenario we want to show different layouts for old, adult and young pets using the Age property. We have a common base class and there is a generic base class `MvxTemplateSelector<T>` that allows us to implement `IMvxTemplateSelector` without casting manually.

{% highlight c# %}
public class AgeTemplateSelector : MvxTemplateSelector<PetViewModelBase>
{
    public override int GetItemLayoutId(int fromViewType)
    {
        return fromViewType;
    }

    protected override int SelectItemViewType(PetViewModelBase forItemObject)
    {
        if (forItemObject.Age <= 1)
            return Resource.Layout.pet_age_young;
        else if (forItemObject.Age <= 10)
            return Resource.Layout.pet_age_adult;
        return Resource.Layout.pet_age_old;
    }
}
{% endhighlight %}

This time we override `SelectItemViewType` that is called with a `PetViewModelBase` as parameter. We return the layout id directly as view type again. You can use the `MvxTemplateSelector<T> in the first scenario, too.

**Setting the Selector**

You can set the selector in your code behind:

{% highlight c# %}
protected override void OnCreate(Bundle bundle)
{
    base.OnCreate(bundle);
    SetContentView(Resource.Layout.FirstView);
    
    var petList = FindViewById<MvxRecyclerView>(Resource.Id.recyclerView);
    petList.ItemTemplateSelector = new TypeTemplateSelector();
}
{% endhighlight %}

And you can set it in the xml. But this code is not as refactoring safe as the code behind version and you can't initialize the selector if you create a configurable one. The attribute is called `MvxTemplateSelector` and has to be set with the full qualified class name of the selector followed by the assembly name.

{% highlight xml %}
<MvvmCross.Droid.Support.V7.RecyclerView.MvxRecyclerView
    android:id="@+id/recyclerView"
    android:layout_width="match_parent"
    android:layout_height="0dp"
    android:layout_weight="1"
    local:MvxBind="ItemsSource Pets"
    local:MvxTemplateSelector="RecyclerViewDifferentTemplates.Droid.TypeTemplateSelector,RecyclerViewDifferentTemplates.Droid">
</MvvmCross.Droid.Support.V7.RecyclerView.MvxRecyclerView>
{% endhighlight %}

It is possible to exchange the selector at the runtime as well. The items will update immediately. In the sample the two buttons `Type` and `Age` are switching between our two custom selectors. Just give it a try.

<h3>Result</h3>

The Result looks like:

<img src="/img/polyrecyclerresult.png" />

<br>
<small>[Background Photo](https://flic.kr/p/9TRC1f "Photo") by [Andy Arthur](https://www.flickr.com/photos/andyarthur/ "Andy Arthur") / [CC BY](http://creativecommons.org/licenses/by/2.0/ "CC BY")</small>
<br>

<small>Found a typo? Send me a pull request!</small>
