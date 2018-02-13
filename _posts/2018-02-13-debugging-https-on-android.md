---
layout: post
title: Debugging TLS traffic on android
description: "Learn how you can debug TLS traffic on android apps/browsing"
modified: 2018-02-13
tags: [burp, burpsuite, https, tls, android]
image:
    feature:
    credit:
    creditlink:
feature:
credit:
creditlink:
---

tl;dr
I'll show how you can use `burp`'s proxy to debug traffic of android apps that use https.


## Settup
The first step is do as described in `Configure the Burp Proxy listener` of the following article: [Configuring an Android Device to Work With Burp](https://support.portswigger.net/customer/portal/articles/1841101-configuring-an-android-device-to-work-with-burp).

As you can see, the second part of the previous article already show how you can use `burp` as a proxy, but with just this you wont be able to do any connection that uses https (eg: if you try to open chrome and access `https://google.com` you won't be able so see the traffic in `burp`s `Proxy` section and will be able to see some `ssl` errors on the `Alert` section).

## Generate a certificate
So, after you setup the proxy you need to generate a certificate on `burp`:

{% highlight %}
this is part of the following gist https://gist.github.com/PaulSec/dab5d25573d7f2d7da18

1. Export your Burp Certificate
Proxy > Options > CA Certificate > Export in DER format

2. Convert it to PEM
openssl x509 -inform der -in cacert.der -out burp.pem

4. Download it on the device
{% endhighlight %}

To download the file to the device you can run

{% highlight bash %}
adb push burp.pem /storage/self/primary/Download/
{% endhighlight %}

to store the certificate on the device's `Download` folder.

## Add the certificate to android's truststore
After that you just need to import the certificate, so the self signed certificates that `burp`s `Proxy` will use won't be rejected by android.
As described [here](https://stackoverflow.com/a/32887169/3939522) you just need `Settings -> Security -> Install from SD card` and find the certificate that you just pushed to the device.

Now you can go back to the `Configuring an Android Device to Work With Burp` article and do as described in `Configure your device to use the proxy` section. If your device is in the same network that your pc, just search for `find local ip` and, after you find out what is your ip you can put it in the `Device proxy` configuration.

## Finally
Just a few tips:
- `burp`s `Proxy` start with `Intercept` enabled by default (in `Proxy -> Intercept`), you probably want to turn it off, otherwise you will need to allow each request by hand.
- if you use `burp` free edition, youo may need to regenerate a new certificate after closing it.  It's easy: `Proxy -> Options -> Regenare CA certificate`, just remember to do the the steps to push it into the device and adding it to the device's truststore again.
- if you use a real device, you may see an warning from android, something in the line that you connection may be spoofed, this is due the new certificate in the truststore. After you finish your tests just remove `burp`s certificate (on android `Settings -> Security -> Trusted credentials -> User`, click on the cetificate e `Remove`, you may need to scroll a litle to see the `Remove` option) and the warning goes away.
