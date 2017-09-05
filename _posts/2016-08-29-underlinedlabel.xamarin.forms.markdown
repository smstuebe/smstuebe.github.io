---
layout:     post
title:      "Underlined Label Text in Xamarin.Forms"
subtitle:   "This is how you underline a Text of a Label using Effects."
date:       2016-08-29 12:00:00
author:     "Sven-Michael Stübe"
header-img: "img/fonts-bg.jpg"
tags: [Xamarin, Forms, Android, iOS, Label]
---

I saw <a href="http://stackoverflow.com/q/39199975/1489968" target="_blank">a question on stackoverflow</a> and thought it would be a nice use case for Effects.

<h2 class="section-heading">Problem</h2>

The <a href="https://developer.xamarin.com/api/type/Xamarin.Forms.FontAttributes/"  target="_blank">FontAttributes</a> contain only `None`, `Bold` and `Italic`. If you want to underline your text, you have to implement it on your own.

<h2 class="section-heading">Effects</h2>

<a href="https://developer.xamarin.com/guides/xamarin-forms/effects/creating/">Xamarin.Forms Effects</a> allow you to customize the underlying native controls without writing custom renderers and custom controls. Creating your own effects is really easy if you follow the linked tutorial.

<b>Core</b><br>
To be able to use the effect in you Xaml, you have to create a `RoutingEffect` in your Core project.

{% highlight c# %}
public class UnderlineEffect : RoutingEffect
{
	public const string EffectNamespace = "Example";

	public UnderlineEffect() : base($"{EffectNamespace}.{nameof(UnderlineEffect)}")
	{
	}
}
{% endhighlight %}

I introduced `EffectNamespace` to add some refactoring safety, because this namespace is used in the platform specific implementation, too.


Add the Effect to your Label:

{% highlight xml %}
<Label Text="Welcome to underlined Xamarin Forms!"
       VerticalOptions="Center"
       HorizontalOptions="Center">
    <Label.Effects>
        <local:UnderlineEffect />
    </Label.Effects>
</Label>
{% endhighlight %}
    
<b>Android</b><br>

On Android it's pretty easy to underline the whole text. You just have to set the paint flag for it. You should listen to some property changes (e.g. `Text` and `FormattedText`), because the renderer might reset the paint flags. You should add the flag using the OR-operator in combination with the old flags, to keep the already set flags. When you remove the effect, you should remove via the inverse bit operation.

{% highlight c# %}
[assembly: ResolutionGroupName(UnderlineLabel.UnderlineEffect.EffectNamespace)]
[assembly: ExportEffect(typeof(UnderlineEffect), nameof(UnderlineEffect))]
namespace UnderlineLabel.Droid
{
    public class UnderlineEffect : PlatformEffect
    {
        protected override void OnAttached()
        {
            SetUnderline(true);
        }

        protected override void OnDetached()
        {
            SetUnderline(false);
        }

        protected override void OnElementPropertyChanged(System.ComponentModel.PropertyChangedEventArgs args)
        {
            base.OnElementPropertyChanged(args);

            if (args.PropertyName == Label.TextProperty.PropertyName || args.PropertyName == Label.FormattedTextProperty.PropertyName)
            {
                SetUnderline(true);
            }
        }

        private void SetUnderline(bool underlined)
        {
            try
            {
                var textView = (TextView)Control;
                if (underlined)
                {
                    textView.PaintFlags |= PaintFlags.UnderlineText;
                }
                else
                {
                    textView.PaintFlags &= ~PaintFlags.UnderlineText;
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine("Cannot underline Label. Error: ", ex.Message);
            }
        }
    }
}
{% endhighlight %}

<b>iOS</b><br>

Underlining a Text on iOS is as easy as on Android. You have to add the `UnderlineStyle` attribute to the `AttributedText`.

{% highlight c# %}
[assembly: ResolutionGroupName(UnderlineLabel.UnderlineEffect.EffectNamespace)]
[assembly: ExportEffect(typeof(UnderlineEffect), nameof(UnderlineEffect))]
namespace UnderlineLabel.iOS
{
    public class UnderlineEffect : PlatformEffect
    {
        protected override void OnAttached()
        {
            SetUnderline(true);
        }

        protected override void OnDetached()
        {
            SetUnderline(false);
        }

        protected override void OnElementPropertyChanged(System.ComponentModel.PropertyChangedEventArgs args)
        {
            base.OnElementPropertyChanged(args);

            if (args.PropertyName == Label.TextProperty.PropertyName || args.PropertyName == Label.FormattedTextProperty.PropertyName)
            {
                SetUnderline(true);
            }
        }

        private void SetUnderline(bool underlined)
        {
            try
            {
                var label = (UILabel)Control;
                var text = (NSMutableAttributedString)label.AttributedText;
                var range = new NSRange(0, text.Length);

                if (underlined)
                {
                    text.AddAttribute(UIStringAttributeKey.UnderlineStyle, NSNumber.FromInt32((int)NSUnderlineStyle.Single), range);
                }
                else
                {
                    text.RemoveAttribute(UIStringAttributeKey.UnderlineStyle, range);
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine("Cannot underline Label. Error: ", ex.Message);
            }
        }
    }
}
{% endhighlight %}

<h2 class="section-heading">Conclusion</h2>

It's very easy to customize native controls with Effects.<br>
Keep in mind, that this is a solution for Labels with simple text. If you want to underline just some parts of the Text using `FormattedText`, you might have a problem :) See my [post on FormattedText]({% post_url 2016-04-03-formattedtext.xamrin.forms %}) for more information on the problem.

<h2 class="section-heading">Code Sample</h2>
The implementation of the Effect are pushed to a github project <br>(<i class="fa fa-github"></i><a href="https://github.com/smstuebe/stackoverflow-answers/tree/master/xamarin-forms-underline" target="_blank"> stackoverflow-answers/xamarin-forms-underline</a>). Feel free to use these if you want to play around or have a closer look to effects.

<center>If you like the answer, I'd appreciate an upvote ;) <br><a href="http://stackoverflow.com/a/39200923/1489968" target="_blank" title="Answer">http://stackoverflow.com/a/39200923/1489968</a><br>Thanks!</center>

<br>
<small>[Background Photo](https://flic.kr/p/fJNKB "Photo") by [Karen](https://www.flickr.com/photos/56832361@N00/ "Karen") / [CC BY](http://creativecommons.org/licenses/by/2.0/ "CC BY")</small>
<br>
<small>Found a typo? Send me a pull request!</small>
