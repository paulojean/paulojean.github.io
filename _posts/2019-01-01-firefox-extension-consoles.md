---
layout: post
title: Dealing with Web and Browser Console while developing Firefox Add-ons
description: "Learn some diffences between Firefox' Web Console and Browser Console"
modified: 2019-01-01
tags: [firefox, firefox-extensions, dev-tools]
image:
    feature:
    credit:
    creditlink:
feature:
credit:
creditlink:
---

tl;dr
There are two `Consoles` in Firefox: [Web](https://developer.mozilla.org/en-US/docs/Tools/Web_Console) and [Browser](https://developer.mozilla.org/en-US/docs/Tools/Browser_Console) Console, depending on what you are trying to do/debug, you need to check both.

## Firefox add-ons and consoles

When you start developing [Addon-ons](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons) for Firefox you may need to debug to understand why something is not working as intended.

### Loading local Add-on

The first thing to do is to [load the addon-on locally](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Temporary_Installation_in_Firefox)[1] (to avoid publishing new version).

### Console

From your local Add-on you can do things like `console.log(42)` and `42` will appear in the `Web` and `Browser` Console. But if yout try to log things that, for example, deals with the local storage, as in:

{% highlight bash %}
browser.storage.local.get()
  .then(console.log)
{% endhighlight %}

You will get different behaviours:
- `Web Console`: will log the contents that you've previously stored (using `browser.storage.set`
- `Browser Console`: will log `<unavailable>`

So, from what I did so far, most of the things that you want to log will be shown in the `Web Console`, but you  still need to keep an eye in the `Browser Console`, because things, such as errors (eg: syntax error), will be shown only in this console.

[1] One note in this process: if you want to load all of your code (to make it work as it was installed from Mozilla' store, you should load your `manifest.json` instead of your script's entry point)
