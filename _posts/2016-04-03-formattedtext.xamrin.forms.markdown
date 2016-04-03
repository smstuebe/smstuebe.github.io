---
layout:     post
title:      "FormattedText with custom fonts in Xamarin.Forms"
subtitle:   "I tried to answer a question on stackoverflow and found out that using custom fonts in FormattedText in Xamarin.Forms is not as easy as I thought."
date:       2016-04-03 12:00:00
author:     "Sven-Michael Stübe"
header-img: "img/fonts-bg.jpg"
tags: [Xamarin, Forms, Android, FormattedText, iOS, Label]
---

Last weekend I saw <a href="http://stackoverflow.com/questions/36223519/custom-font-in-xamarin-android-label-with-formattedstring" target="_blank">Chet's question on stackoverflow</a> and thought it would be easy to answer. But then it took me several hours to investigate what the problem is. I want to share my findings and solutions additionally to my stackoverflow answer here on my blog.

<h2 class="section-heading">Problem</h2>
<h3>Let chet describe the problem</h3>

> I have created a custom LabelRenderer in my Android app to apply a custom font in a Xamarin Android app.
Everything works great for a normal label with the content added to the .Text property. However, if I create a label using .FormattedText property, the custom font is not applied.

<h3>Well, let's visualize it!</h3>

<b>Xaml</b><br>
We use this view that has 3 labels with different attributes and a custom font and one label, that has a custom font and a `FormattedText` with several differently styled lines.
{% highlight xml %}
<Label FontFamily="LobsterTwo" Text="Normal Label" HorizontalTextAlignment="Center"></Label>
<Label FontFamily="LobsterTwo" Text="Bold Label" FontAttributes="Bold" HorizontalTextAlignment="Center"></Label>
<Label FontFamily="LobsterTwo" Text="Bigger Italic Label" FontSize="20" FontAttributes="Italic" HorizontalTextAlignment="Center"></Label>
<Label FontFamily="LobsterTwo" HorizontalTextAlignment="Center">
  <Label.FormattedText>
    <FormattedString>
      <Span Text="FG only&#10;" ForegroundColor="Red"></Span>
      <Span Text="BG only&#10;" BackgroundColor="Lime"></Span>
      <Span Text="BG and FG&#10;" BackgroundColor="Fuchsia" ForegroundColor="White"></Span>
      <Span Text="FG and Size&#10;" ForegroundColor="Green" FontSize="40"></Span>
      <Span Text="Size and Bold&#10;" FontSize="20" FontAttributes="Bold"></Span>
      <Span Text="Custom Font&#10;" FontFamily="Quantico"></Span>
      <Span Text="All Attributes&#10;" FontFamily="Quantico" FontSize="20" FontAttributes="Bold,Italic" BackgroundColor="Black" ForegroundColor="White"></Span>
    </FormattedString>
  </Label.FormattedText>
</Label>
{% endhighlight %}
    
<b>Simple Renderer</b><br>
As described in <a href="https://developer.xamarin.com/guides/xamarin-forms/user-interface/text/fonts/#Android" target="_blank">Setting Fonts in Xamarin.Forms</a> we need a custom renderer if we want to use custom fonts in Android. I used the a simple custom renderer (like Chet did) for the first try. It uses the specified `FontFamily`, appends `-Regular.ttf` and loads this font from the assets.

{% highlight c# %}
public class SimpleLabelRenderer : LabelRenderer
{
    protected override void OnElementChanged(ElementChangedEventArgs<Label> e)
    {
        base.OnElementChanged(e);
        Control.Typeface = Typeface.CreateFromAsset(Forms.Context.Assets, $"{Element.FontFamily}-Regular.ttf");
        UpdateFormattedText();
    }
}
{% endhighlight %}

<b>Result</b><br>
<img src="/img/fonts-android-1.png" />

As we see, the custom font is applied only to the simple labels, but the bold and italic attribute is not considered. You might facepalm now and think <i>"Sven! You are loading the regular font in your custom renderer!"</i>. That's true. I did this on purpose, because its that one from Xamarin's blog post. So let's fix it with adding this method to our renderer.

{% highlight c# %}
private static string GetFontName(string fontFamily, FontAttributes fontAttributes)
{
    var postfix = "Regular";
    var bold = fontAttributes.HasFlag(FontAttributes.Bold);
    var italic = fontAttributes.HasFlag(FontAttributes.Italic);
    if (bold && italic) { postfix = "BoldItalic"; }
    else if (bold) { postfix = "Bold"; }
    else if (italic) { postfix = "Italic"; }

    return $"{fontFamily}-{postfix}.ttf";
}
{% endhighlight %}

<img src="/img/fonts-android-2.png" />

Yay, at least, the normal labels are looking as expected, now.
 
> Why don't you set the font family to "XYZ-Bold" if you want bold text?

Because I'd expect it to work similar to standard fonts. As you can see on the 3rd last line, the attributes take effect on the standard font.

<h2 class="section-heading">FormattedText</h2>
Ok, basics done, let's finally answer Chet's question! As you can see, I set the font on the formatted text label but it isn't applied to the spans. Unfortunately, there is no `SpanRenderer` that we can override. But how can we customize the interpretation of a span then!? Android is using a very similar method to allow formatted strings within a label. The sections are even called spans! We just have to find out how Xamarin.Forms translates them.

<h3>Decompile to the rescue!</h3>
I used JetBrains' dotPeek to gather some insight. The `LabelRenderer` is converting the FormattedString into a `SpannableString` using `FormattedStringExtensions.ToAttributed` which is basically iterating the spans and creating `FontSpan`s and setting them on the right positions. Ok, there are unfortunately two problems:

- You can't override something, because the functionality is in an extension method
- `FontSpan` is a private inner class of `FormattedStringExtensions` that therefore you can't inherit from and override some functions.

<img src="/img/fonts-decompiled.png" />

<h3>Extended Renderer</h3>
In our extended renderer, we need to replace all `FontSpan` with our `CustomTypefaceSpan`. The `UpdateFormattedText` function does exactly this and gets called in `OnElementChanged` and `OnElementPropertyChanged` of our renderer.

{% highlight c# %}
private void UpdateFormattedText()
{
    if (Element?.FormattedText == null)
        return;

    var extensionType = typeof(FormattedStringExtensions);
    var type = extensionType.GetNestedType("FontSpan", BindingFlags.NonPublic);
    var ss = new SpannableString(Control.TextFormatted);
    var spans = ss.GetSpans(0, ss.ToString().Length, Class.FromType(type));
    foreach (var span in spans)
    {
        var start = ss.GetSpanStart(span);
        var end = ss.GetSpanEnd(span);
        var flags = ss.GetSpanFlags(span);
        var font = (Font)type.GetProperty("Font").GetValue(span, null);
        ss.RemoveSpan(span);
        var newSpan = new CustomTypefaceSpan(Control, Element, font);
        ss.SetSpan(newSpan, start, end, flags);
    }
    Control.TextFormatted = ss;
} 
{% endhighlight %}

<b>Result</b><br>
<img src="/img/fonts-android-3.png" />

<h2 class="section-heading">iOS works ... NOT</h2>
I thought it would be nice to sum up my findings in a neat blog post. For a comparsion I wanted to show Android and iOS side by side to visualize the error. But unfortunately after I started the iOS app I saw the following:

<img src="/img/fonts-ios-1.png" />

As you can see, the standard iOS LabelRenderer works better than the Android one. But its not perfect, because:

- `FontAttibutes` are not used
- `Span` can't handle custom fonts if the `Size` or `FontAttributes` are set and the `FontFamily` has not been set explicitly 
- the background is drawn to the end of line, if you have line breaks in your text

You can fix the first issue, by setting `FontFamily` explicitly to `LobsterTwo-Bold` and `LobsterTwo-Italic`. But as mentioned in the Android part I think it would be very inconsistent. The same applies to the second issue. You can simply set the `FontFamily` explicitly on each span and you are done. The third issue can be solved, by moving the line break to non formatted spans. That's alot of manual work to do. Let's fix this in a custom renderer.

<h3>Renderer</h3>

iOS is using `AttributedString` to format text. The standard `LabelRenderer` is generating such an `AttributedString` out of the spans. We can use the same approach as in the Android renderer. After the standard renderer has done his work, we correct his work a little bit.

{% highlight c# %}
private void UpdateFormattedText()
{
    var text = Control?.AttributedText as NSMutableAttributedString;
    if(text == null)
        return;
        
    var fontFamily = Element.FontFamily;
    text.BeginEditing();
    if (Element.FormattedText == null)
    {
        FixFontAtLocation(0, text, fontFamily, Element.FontAttributes);
    }
    else
    {
        var location = 0;
        foreach (var span in Element.FormattedText.Spans)
        {
            var spanFamily = span.FontFamily ?? fontFamily;
            FixFontAtLocation(location, text, spanFamily, span.FontAttributes);
            location += span.Text.Length;
        }
    }
    text.EndEditing();
}
{% endhighlight %}

Our `UpdateFormattedText` method is casting the AttributedText property of our `UILabel` to `NSMutableAttributedString`. This is - as the name implies - the mutable version of `AttributedString` :) It is then calling `FixFontAtLocation` once, if `FormattedString` isn't used, or for each span, if `FormattedString` is used. 

{% highlight c# %}
private void FixFontAtLocation(int location, NSMutableAttributedString text, string fontFamily, FontAttributes fontAttributes)
{
    if(fontFamily == null)
        return;

    NSRange range;
    var font = (UIFont)text.GetAttribute(UIStringAttributeKey.Font, location, out range);
    var baseFontName = GetBaseFontName(fontFamily);

    if (font.Name.Contains("-") && font.Name.StartsWith(baseFontName))
        return;

    var newName = GetFontName(fontFamily, fontAttributes);
    font = UIFont.FromName(newName, font.PointSize);
    text.RemoveAttribute(UIStringAttributeKey.Font, range);
    text.AddAttribute(UIStringAttributeKey.Font, font, range);
}
{% endhighlight %}

`FixFontAtLocation` gets the font attribute at the given location and replaces it with a font attribute that has the correct font family.

<b>Result</b><br>
<img src="/img/fonts-ios-2.png" />

<h3>Fixing the Background</h3>
With the knowledge of the previous section, fixing the backgorund is easy. We just have to write a function that removes the background color at the positions where `\n` occur. 
{% highlight c# %}
private static void FixBackground(NSMutableAttributedString text)
{
    var str = text.Value;
    for (var i = 0; i < str.Length; i++)
    {
        if (str[i] == '\n')
        {
            text.RemoveAttribute(UIStringAttributeKey.BackgroundColor, new NSRange(i, 1));
        }
    }
}
{% endhighlight %}

<b>Result</b><br>
And what we get looks pretty similar to our Android version. The Android default font color is grey, because this is set in the used theme.
<img src="/img/fonts-ios-3.png" />

<h2 class="section-heading">Conclusion</h2>
<h3>Notes</h3>
The Renderers implement a behaviour that is similar to not using custom fonts, but aren't perfect. In productive projects you should

- cache the fonts
- add some fall back mechanisms if no font for Bold, Italic and BoldItalic are available

Some fonts have more than 2 Attributes. With the gathered knowledge it is now really easy to implement a `CustomLabel` with some sort of `FontAttributesEx` property where you can define Bold, Italic, Thin, etc.

<h3>Code Sample</h3>
The actual implementation of the Renderers and the sample projects are pushed in a github project (<i class="fa fa-github"></i>[xamarin-forms-formattedtext](https://github.com/smstuebe/xamarin-forms-formattedtext "xamarin-forms-formattedtext")). Feel free to use these if you want to play around or have a closer look to the formatted text issue.


<br>
<small>[Background Photo](https://flic.kr/p/fJNKB "Photo") by [Karen](https://www.flickr.com/photos/56832361@N00/ "Karen") / [CC BY](http://creativecommons.org/licenses/by/2.0/ "CC BY")</small>
<br>
<small>Found a typo? Send me a pull request!</small>