---
id: 482
title: 'Sharing in RX: Publish, Replay, and Multicast'
date: 2011-11-10T22:54:38+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=482
permalink: /2011/sharing-in-rx/
categories:
  - Coding
  - Lab49
tags:
  - 'C#'
  - RX
---
State is a tricky thing in RX, especially when we have more than one subscriber to a stream. Consider a fairly innocuous setup:

<pre class="brush:csharp">var interval = Observable.Interval(TimeSpan.FromMilliseconds(500));
interval.Subscribe(i => Console.WriteLine("First: {0}", i));
interval.Subscribe(i => Console.WriteLine("Second: {0}", i));
</pre>

At first glance it might look like I&#8217;m setting up two listeners to a single interval pulse, but what&#8217;s actually happening is that each time I call `Subscribe`, I&#8217;ve created a new timer to tick values. Imagine how bad this could be if instead of an interval I was, say, sending a message across the network and waiting for a response. <!--more-->

There are a few ways to solve this problem, but only one of them is actually correct. The first thing most people latch on to in Intellisense is `Publish()`. Now _that_ method looks useful. So I might try:

<pre class="brush:csharp">var interval = Observable.Interval(TimeSpan.FromMilliseconds(500)).Publish();
interval.Subscribe(i => Console.WriteLine("First: {0}", i));
interval.Subscribe(i => Console.WriteLine("Second: {0}", i));
</pre>

Now I get nothing at all. Great advertising there.

So what is actually happening here? Well, for one you might notice that `interval` is no longer `IObservable<long>` but is now an `IConnectableObservable<long>`, which extends `IObservable` with a single method: `IDisposable Connect()`.

As it turns out, `Publish` is simply a convenience method for `Multicast` that supplies the parameter for you. Specifically, calling `stream.Publish()` is exactly the same as calling `stream.Multicast(new Subject<T>())`. Wait, a subject?

What `Multicast` does is create a concrete implementation of `IConnectableObservable<T>` to wrap the subject we give it and forwards the `Subscribe` method of the `IConnectableObservable<T>` to `Subject<T>.Subscribe`, so it looks something like this:

<img src="http://northhorizon.net/wp-content/uploads/2011/11/before-connect.png" alt="" title="Before Connecting" width="375" height="106" class="aligncenter size-full wp-image-485" srcset="http://northhorizon.net/wp-content/uploads/2011/11/before-connect.png 375w, http://northhorizon.net/wp-content/uploads/2011/11/before-connect-300x84.png 300w" sizes="(max-width: 375px) 85vw, 375px" />

You might have noticed that the input doesn&#8217;t go anywhere. That&#8217;s exactly why our simple call to `Publish()` earlier didn&#8217;t produce any results at all &#8211; `IConnectableObservable<T>` hadn&#8217;t been fully wired up yet. To do that, we need to make a call to `Connect()`, which will subscribe our input into our subject.

<img src="http://northhorizon.net/wp-content/uploads/2011/11/connect.png" alt="" title="Connect" width="375" height="163" class="aligncenter size-full wp-image-486" srcset="http://northhorizon.net/wp-content/uploads/2011/11/connect.png 375w, http://northhorizon.net/wp-content/uploads/2011/11/connect-300x130.png 300w" sizes="(max-width: 375px) 85vw, 375px" />

`Connect()` returns to us an `IDisposable` which we can use to cut off the input again. Keep in mind the downstream observers _have no idea any of this is happening_. When we disconnect, `OnCompleted` will _not_ be fired.

<img src="http://northhorizon.net/wp-content/uploads/2011/11/disconnect.png" alt="" title="Disconnect" width="375" height="164" class="aligncenter size-full wp-image-487" srcset="http://northhorizon.net/wp-content/uploads/2011/11/disconnect.png 375w, http://northhorizon.net/wp-content/uploads/2011/11/disconnect-300x131.png 300w" sizes="(max-width: 375px) 85vw, 375px" />

Getting back to my example, the correct code looks like this:

<pre class="brush:csharp">var interval = Observable.Interval(TimeSpan.FromMilliseconds(500)).Publish();
interval.Subscribe(i => Console.WriteLine("First: {0}", i));
interval.Subscribe(i => Console.WriteLine("Second: {0}", i));

var connection = interval.Connect();

// Later
connection.Dispose();
</pre>

It is very important to make sure all of your subscribers are setup _before_ you call `Connect()`. You can think of `Publish` (or, really, `Multicast`) like a valve on a pipe. You want to be sure you have all your pipes together and sealed before you open it up, otherwise you&#8217;ll have a mess.

A problem that comes up fairly often is what do you do when you do not control the number or timing of subscriptions? For instance, if I have a stream of USD/EUR market rates, there&#8217;s no need for me to keep that stream open if nobody is listening, but if someone is, I&#8217;d like to share that connection, rather than create a new one.

This is where `RefCount()` comes in. `RefCount()` takes an `IConnectableObservable<T>` and returns an `IObservable<Tg>`, but with a twist. When `RefCount` gets its first subscriber, it automatically calls `Connect()` for you and keeps the connection open as long as anyone is listening; once the last subscriber disconnects, it will call `Dispose` on its connection token.

So now you might be wondering why I didn&#8217;t use `RefCount()` in my so-called &#8220;correct&#8221; implementation. I wouldn&#8217;t have had to call either `Connect()` or `Dispose`, and less is more, right? All that is true, but it omits the cost of safety. Once I dispose my connection, my source no longer has an object reference to my object, which allows the GC to do what it does best. Often, these streams start to make their way outside of my class, which can create a long dependency chain of object references. That&#8217;s fine, but if I dispose an object in the middle, I want to make sure that that object is now ready for collection, and if I `RefCount()`, I simply can&#8217;t make that assertion, because I&#8217;d have to ensure every downstream subscriber had also disposed.

Another scenario that comes up is how to keep a record of things you&#8217;ve already received. For instance, I might make a call to find all tweets with the hashtag &#8220;#RxNet&#8221; with live updates. If I subscribe second observer, I might expect that all the previously found data to be sent again without making a new request to the server. Fortunately, we have `Replay()` for this. It literally has 15 overloads, which cover just about every permutation of windowing by count and/or time, and supplying an optional scheduler and/or selector. The parameterless call, however, just remembers everything. `Replay` is just like `Publish` in the sense that it also forwards a call to `Multicast`, but this time with a `ReplaySubject<T>`.

Now the temptation is to combine `Replay()` and `RefCount` to make caches of things for subscribers when they are needed. Lets look at my Twitter example.

<pre class="brush:csharp;gutter:false">tweetService.FindByHashTag("RxNet").Replay().RefCount()</pre>

When the first observer subscribes, `FindByHashTag` will make a call (I assume this is deferred) to the network and start streaming data, and all is well. When the second observer subscribes, he gets all the previously found data and updates. Great! Now, let&#8217;s say both unsubscribe and a third observer then subscribes. He&#8217;s going to get all that previous data, and then the deferred call in `FindByHashTag` is going to be re-triggered and provide results that we might have already received from the replay cache! Instead, we should implement a caching solution that actually does what we expect and wrap it in an `Observable.Create`, or if we expect only fresh tweets, use `Publish().RefCount()` instead.