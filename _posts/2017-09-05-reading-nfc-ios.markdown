---
layout:     post
title:      "Reading NFC tags with <nobr>iOS 11</nobr> and CoreNFC"
subtitle:   "CoreNFC is a new iOS 11 API that allows you to read NFC tags."
date:       2017-09-05 12:00:00
author:     "Sven-Michael Stübe"
card-img:   "img/nfc-ios-icon.png"
tags: [Xamarin, iOS, iOS 11, NFC, CoreNFC, NDEF]
---

Besides introducing the famous ARKit, Apple made a former private API usable in every App with iOS 11: <a href="https://developer.apple.com/documentation/corenfc" target="_blank" onclick="return tol(this);">CoreNFC</a>. CoreNFC allows reading NFC tags with your iPhone 7/7+ and maybe soon with your iPhone 8 ;) And of course the Xamarin team has worked hard to make it usable with C#. In this blog post I'll show you how to quickly read a message from a NFC tag formatted in NDEF.

<h2 class="section-heading">Setup</h2>

<h3>Development environment</h3>
Because iOS 11 is still beta, the setup up effort is a bit higher. You can skip this if Apple and Xamarin have released the stable verions.

- Download and install XCode 9 beta 6 
- Download and install the Xamarin release for XCode 9 support <a href="https://releases.xamarin.com/preview-xcode-9-beta-6-ios-11-macos-10-13-support-preview-6/" target="_blank" onclick="return tol(this);">here</a> and read **carefully**!

<h3>App Identifier</h3>
Unfortunately, you can't just use the API with any App. You have to enable the `NFC Tag Reading` feature in your developer account.

- Go to <a href="https://developer.apple.com" target="_blank" onclick="return tol(this);">developer.apple.com</a> and sign into your developer account
- Open App IDs
- Create a new **Explicit** App ID e.g. `com.smstuebe.test.nfc` (Wildcard App IDs don't allow enabling this feature) 
- check `NFC Tag Reading` and continue
- make sure the App ID is included in a provisioning profile

<img src="/img/nfc-ios-app-id.png" style="margin:0 auto; cursor: pointer;"/>

<h3>Project</h3>
In your iOS project you have to add some more stuff.

<b>Info.plist</b>

In your Info.plist file you have to set the `CFBundleIdentifier` and add the `NFCReaderUsageDescription`.

{% highlight xml %}
<key>CFBundleIdentifier</key>
<string>com.smstuebe.test.nfc</string>
<key>NFCReaderUsageDescription</key>
<string>This is very important to my App.</string>
{% endhighlight %}

<b>Entitlements.plist</b>

Add the following lines to the Entitlements.plist file. Because we want to read Tags using the NDEF format, we add `NDEF` to the `com.apple.developer.nfc.readersession.formats`.

{% highlight xml %}
<dict>
    <key>com.apple.developer.nfc.readersession.formats</key>
    <array>
        <string>NDEF</string>
    </array>
</dict>
{% endhighlight %}

<h2 class="section-heading">Code</h2>
Finally, the setup is done. Coding will be much easier :) You can easily asyncify the API by using a `TaskCompletionSource`.

{% highlight c# %}
public class AsyncNfcReader : NSObject, INFCNdefReaderSessionDelegate
{
    private NFCNdefReaderSession _session;
    private TaskCompletionSource<string> _tcs;

    public Task<string> ScanAsync()
    {
        if (NFCNdefReaderSession.ReadingAvailable)
        {
            throw new InvalidOperationException("Reading NDEF is not available");
        }

        _tcs = new TaskCompletionSource<string>();
        _session = new NFCNdefReaderSession(this, DispatchQueue.CurrentQueue, true);
        _session.BeginSession();
        
        return _tcs.Task;
    }

    public void DidInvalidate(NFCNdefReaderSession session, NSError error)
    {
        _tcs.TrySetException(new Exception(error?.LocalizedFailureReason));
    }

    public void DidDetect(NFCNdefReaderSession session, NFCNdefMessage[] messages)
    {
        var bytes = messages[0].Records[0].Payload.Skip(3).ToArray();
        var message = Encoding.UTF8.GetString(bytes);
        _tcs.SetResult(message);
    }
}
{% endhighlight %}

`NFCNdefReaderSession` is the class that handles the reading of NDEF tags. It creates a system dialog (see Test section) when calling `BeginSession()`. This dialog isn't customizable, yet. Because of my experiences with the fingerprint API, I think Apple will add some possibilities for customization (mainly localizing the shown texts) in later releases.

You can read a single tag or multiple tags in one session. The 3rd constructor parameter `invalidateAfterFirstRead` is set to true, because we want to read just one tag.

The `AsyncNfcReader` implements the `INFCNdefReaderSessionDelegate` and thats why it inherits from `NSObject`. In the `DidDetect` we read the message from the first record. In my case the Record is of type `T`. This means the first 3 bytes contain the Encoding and the language. In the interests of simplification I just ignore the first 3 bytes with `Skip(3)` and decode the remaining bytes as UTF8 string. The format is specified in the <a href="https://learn.adafruit.com/adafruit-pn532-rfid-nfc/ndef" target="_blank" onclick="return tol(this);">NDEF specification</a>. There are libraries like <a href="https://github.com/andijakl/ndef-nfc" target="_blank" onclick="return tol(this);">NDEF NFC</a> that allow decoding the payload correctly.

You can use the `AsyncNfcReader` like this in your code:

{% highlight c# %}
var reader = new AsyncNfcReader();
var message = await reader.ScanAsync();
{% endhighlight %}

<h2 class="section-heading">Test</h2>
For testing I bought some <a href="http://amzn.to/2wDpWsY" target="_blank" onclick="return tol(this);">NFC tags with 4kB storage</a> and wrote some data to it with an Android App.
Simply start the App and scan the tag. The antenna is located at the upper egde of your iPhone.

<img src="/img/nfc-ios-app-test.png" style="margin:0 auto; cursor: pointer;"/>

<h2 class="section-heading">Discussion</h2>
- Error handling and decoding is very sloppy. This has to be hardened in production apps.
- The documentation from Apple isn't complete. The API supports other formats than NDEF (e.g. ISO15693).
- The API doesn't seem to support writing tags (06.09.2017)
- I started implementing a <a href="https://github.com/smstuebe/xamarin-nfc" target="_blank" onclick="return tol(this);">Plugin for reading NFC tags</a>. I'll publish it in the next months after the features are available in the stable releases.

<br>
<small>Found a typo? Send me a pull request!</small>