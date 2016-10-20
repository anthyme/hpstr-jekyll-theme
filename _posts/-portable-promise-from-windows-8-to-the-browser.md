---
layout: post
title:  ""
excerpt: ""
date:   2012-03-01 22:18:44
categories: post
tags : []
---



```csharp

```

This article presents a way to share asynchronous Javascript code between Windows 8 and the browser.

<!--more-->

<h2>Introduction</h2>

I am currently developing a Windows 8 application with the html 5 technologies.
After some time I finaly ask myself "how can I share (async) code between Windows 8 and the browser?".

A solution is to recreate the Promise API in the browser world and I make the choice to use the JQuery defered object to help me (a lot).
In fact jquery defered create "promise" that have an API similar to the Windows 8.

Now let see how we can create this:

[sourcecode language="js"]
(function (global) {
    var Portable = global.Portable = global.Portable || {};

    if (global.WinJS && global.WinJS.Promise) {
        Portable.Promise = WinJS.Promise;
    }
    else {
        Portable.Promise = function (callback) {
            var deferred = Q.defer();
            var complete = function (r) {
                deferred.resolve(r);
            };
            var error = function (r) {
                deferred.reject(r);
            };
            var progress = function (r) {
                deferred.notify(r);
            };

            try {
                callback(complete, error, progress);
            }
            catch (er) {
                deferred.reject(er);
            }
            return deferred.promise;
        };
    }
})(this)
[/sourcecode]

Now you have same API for create promise and continue code with "then" or "done" methods: You can share simple Javascript promises between Windows 8 and browser.
I will share a Repository code for indexedDB in a future article using promise.

Now what is missing ? For example static methods of the WinJS Promise like "as", "any", "join", etc can be really usefull. If you create them (before me) don't hesitate to post them in comments ;)

<h2>Final words</h2>
It is a choice you have to do before starting your project you can imagine these scenarios:

1)

2)

3)

You can find the source code associated to this article <a href="http://sdrv.ms/LfIxhF" target="_blank">here</a>

You can find the source code associated to this article [here][downloadlink]

If you try and you have any observations don't hesitate to comment. ;-)

[downloadlink]: https://skydrive.live.com/re


