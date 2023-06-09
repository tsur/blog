---
layout: post
title:  "Electron Version Numbers"
date:   2015-07-21 19:56:25
categories: electron
published: false
---

Electron is a framework for developing cross-platform desktop applications with the HTML/CSS/JS triada. A common example of electron applications is the Atom editor or Slack. Under the hook, Electron is based on io.js and Chromium projects. When developing on Electron it becomes sometimes necessary to know what versions of what thing are you currently using.

From a developer point of view, most of time Chromium is just Chrome as both of them use the same Blink layout/rendering engine and the same V8 Javascript engine, just that Chrome is a wrapper over Chromium providing extra [features](https://code.google.com/p/chromium/wiki/ChromiumBrowserVsGoogleChrome) as Flash, PDF or MP3 support. Basically Chromium is the name of the project which becomes the base for future Chrome versions whereas Chrome itself is the name of the product. For that reason, Chromium docs encourage developers about not using never the word Chromium as constants or variable names. 

To check both the Electron application, the Chromium and the io.js version, open the dev tools and run "process.versions" on the console. To open the dev tools, just call openDevTools([options]) method in your BrowserWindow application instance.

{% highlight javascript %}
process.versions.chrome // prints the Chromium version
process.versions.electron // prints the Electron version
process.versions.node // prints the io.js version
{% endhighlight %}

To check what version of Chromium is your Electron based application using, you can also run "window.navigator.userAgent" in the console.

You may also check the Electron version by simply taking a look at your package.json file or issuing the command below if installed globally.

{% highlight bash %}
npm list --depth=1 --global | grep electron
{% endhighlight %}

You can also check Chromium version for the last Electron version right in the [sources](https://github.com/atom/electron/blob/master/atom/common/chrome_version.h)