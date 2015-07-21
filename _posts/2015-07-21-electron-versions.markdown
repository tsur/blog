---
layout: post
title:  "Electron Version Numbers"
date:   2015-07-20 22:33:25
categories: electron
---

Electron is a framework for developing cross-platform desktop applications with the HTML/CSS/JS triada. A common example of electron applications is the Atom editor or Slack. Under the hook, Electron is based on io.js nad Chromium. Sometimes , when developing on Electron it becomes a bit tricky to know what versions of what thing are you currently using. So lets'a go one by one:

* Electron bundle version

This is straight forward, specially if you has installed it via npm. Simply by taking a look at your package.json file or issuing a "npm list --depth=1 --global | grep electron" command would make the trick.

* Chromium and io.js version

From a developer point of view, most of time Chromium is just Chrome as both of them use the same Blink layout/rendering engine and the same V8 Javascript engine, just that Chrome is a wrapper over Chromium providing extra features as Flash, PDF or MP3 support. Basically Chromium is the name of the project which becomes the base for future Chrome versions whereas Chrome itself is the name of the product. For that reason, Chromium docs encourages developers about not using never the word Chromium as constants or variable names. You can also check Chromium version for the last Electron version [here](https://github.com/atom/electron/blob/master/atom/common/chrome_version.h)

To check both the electron, the Chromium and the io.js version, open the dev tools and run "process.versions" on the console. To open the dev tools, just call BrowserWindow.openDevTools([options]).

{% highlight js %}
process.versions.chrome // prints the Chromium version
process.versions.electron // prints the Electron version
process.versions.node // prints the io.js version
{% endhighlight %}

To check what version of Chromium is your Electron based application using, you can also run "window.navigator.userAgent" in the console.