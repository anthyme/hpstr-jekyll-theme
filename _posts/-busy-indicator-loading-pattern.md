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

This article presents a way to use busy indicator efficiently in your application.

<!--more-->
<h2>Introduction</h2>

Today Developpers are not just coding, they are developping rich GUI for users that are more and more demanding.

For exemple, in a lot of application you have to load data and block the user during the loading. 

The simpliest approach is to write a Load method and cal it in your View (ViewModel/Model/etc).
In this case you have an application frozen in each load! Your user will say that the application crashs. "Not a crash, it is just a loading..." :(

Fortunatly , nice guys create controls like "BusyIndicator" (WPF toolkit). You put the BusyIndicator in your app and when you have to load data you show it and hide it in the end of loading.

Well ... In fact this is sometime an anti pattern! Why? Because for short loading time your BusyIndicator will blink! And it's horrible for the user and he will think that the application is artificially slow (showing loading at every new screen/interaction)...

The rule (mine) is : don't show a BusyIndicator if the duration is less than 500 ms ans show the BusyIndicator at least 1 seconde.
Easy! The problem is that sometime you can predict the duration (and directly show the BusyIndicator for long running loading) and sometimes you can't.

The solution for a generic loading strategy is to wait 500 ms (or a little less) before showing the busy indicator and them hide it after 1 s or the end af the work.

<h2>WPF</h2>

Here is an implementation of this concept with WPF.

first the View with 3 BusyIndicator for a loadtime of 100ms/600ms/3s

[sourcecode language="xml"]
<xctk:BusyIndicator IsBusy="{Binding IsBusy1}" Grid.Row="0">
    <TextBlock Text="Short task (100ms) load quickly without showing the loading"/>
</xctk:BusyIndicator>
<xctk:BusyIndicator IsBusy="{Binding IsBusy2}" Grid.Row="1">
    <TextBlock Text="Medium task (600ms) display loading after 500ms for 1s"/>
</xctk:BusyIndicator>
<xctk:BusyIndicator IsBusy="{Binding IsBusy3}" Grid.Row="2">
    <TextBlock Text="Big task (3s) display loading after 500ms for 2.5s"/>
</xctk:BusyIndicator>
[/sourcecode]

And the C# code associated.

[sourcecode language="csharp"]
private void Load()
{
    var workPromise1 = Delay(100);
    ManageProgress(workPromise1, v => IsBusy1 = v);

    var workPromise2 = Delay(600);
    ManageProgress(workPromise2, v => IsBusy2 = v);

    var workPromise3 = Delay(3000);
    ManageProgress(workPromise3, v => IsBusy3 = v);
}

public Task ManageProgress(Task task, Action<bool> setter)
{
    var scheduler = TaskScheduler.Current;

    return Delay(500).ContinueWith(
        t =>
        {
            if (task.IsCompleted == false)
            {
                setter(true);
                Delay(1000).ContinueWith(t2 => task.ContinueWith(
                    t3 => setter(false), scheduler));
            }
        }, scheduler);
}

private Task Delay(int ms)
{
    return Task.Factory.StartNew(() => Thread.Sleep(ms));
}
[/sourcecode]


<h2>Web</h2>

The same is possible in the web with promises (here from q.js):

[sourcecode language="html"]
<span>Short task (100ms) load quickly without showing the loading :</span>
<br />
<span id="loading1" style="display:none;">loading</span>
<br /><br />
<span>Medium task (600ms) display loading after 500ms for 1s:</span>
<br />
<span id="loading2" style="display:none;">loading</span>
<br /><br />
<span>Big task (3s) display loading after 500ms for 2.5s:</span>
<br />
<span id="loading3" style="display:none;">loading</span>
[/sourcecode]

[sourcecode language="js"]
$(function () {
    var workPromise1 = delay(100);
    manageProgress(workPromise1, "loading1");

    var workPromise2 = delay(600);
    manageProgress(workPromise2, "loading2");

    var workPromise3 = delay(3000);
    manageProgress(workPromise3, "loading3");
});

function manageProgress(workPromise, loadingId) {
    var loading = true;
    workPromise.then(function() { loading = false; });

    delay(500).then(function() {
        if (loading) {
            showProgress(loadingId);
            delay(1000).then(function() {
                workPromise.then(function () { hideProgress(loadingId); });
            });
        }
    });
}

function delay(ms) {
    var deferred = Q.defer();
    setTimeout(deferred.resolve, ms);
    return deferred.promise;
}
        
function showProgress(id) {
    $("#" + id).show();
}
        
function hideProgress(id) {
    $("#" + id).hide();
}
[/sourcecode]

<h2>Final words</h2>
A little thing for a big win. Focus on ergonomy! Don't let your users with a poor experience on loading data. 

Put yourself in the user place! ;-)

You can find the source code associated to this article [here][downloadlink]

If you try and you have any observations don't hesitate to comment. ;-)

[downloadlink]: https://skydrive.live.com/re


