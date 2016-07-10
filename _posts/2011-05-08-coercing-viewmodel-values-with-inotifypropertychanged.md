---
id: 429
title: Coercing ViewModel Values with INotifyPropertyChanged
date: 2011-05-08T17:21:14+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=429
permalink: /2011/coercing-viewmodel-values-with-inotifypropertychanged/
categories:
  - Coding
  - Lab49
tags:
  - 'C#'
  - INotifyPropertyChanged
  - WPF
---
Perhaps one of the most ambivalent things about putting code on GitHub is that it&#8217;s more or less an open project. It&#8217;s great that people (including yourself) can continue to work on it, but it seems to lack closure so that you can move on with your life.

So one of the things that I&#8217;ve been missing in my `BindableBase` class is property coercion, a la dependency properties. It&#8217;s a pretty smart idea; you can keep values in a valid state without pushing change notification. Unfortunately, there are some problems that crop up pretty quickly in `INotifyPropertyChanged` based view models.<!--more-->

Consider a Foo property that refuses to set a value less than zero.

<pre class="brush:csharp">private int _foo;
public int Foo
{
  get { return _foo; }
  set
  {
    if(_foo &gt;= 0)
      SetProperty(ref _foo, value, "Foo");
  }
}</pre>

That looks like pretty good coercion, but the problem is that your binding and your property are now out of sync. That is, the binding told your object to set a value and assumed it did; you gave it no notification to the contrary. So while your object retains the old value, the bound `TextBox` will blissfully report -23.

A more brute force option would raise `PropertyChanged` in the event of this kind of coercion, but that breaks the code contract for `INotifyPropertyChanged` in that _nothing changed_. So how do we get around this problem?

If you take a look at the invocation list of your `PropertyChanged` event with some bound variables, you&#8217;ll notice that there&#8217;s a `PropertyChangedEventManager` hooked up.

[<img class="aligncenter size-full wp-image-432" title="PropertyChangedEventManager in the Watch" src="http://northhorizon.net/wp-content/uploads/2011/05/propertychangedeventmanager-watch.png" alt="" width="707" height="330" srcset="http://northhorizon.net/wp-content/uploads/2011/05/propertychangedeventmanager-watch.png 707w, http://northhorizon.net/wp-content/uploads/2011/05/propertychangedeventmanager-watch-300x140.png 300w" sizes="(max-width: 709px) 85vw, (max-width: 909px) 67vw, (max-width: 984px) 61vw, (max-width: 1362px) 45vw, 600px" />](http://northhorizon.net/wp-content/uploads/2011/05/propertychangedeventmanager-watch.png)

Considering that the other item in the list is my default delegate, this object must be responsible for communicating my events to the binding system

Of course, the next thing to do is fire up Reflector and take a look at what&#8217;s there.

<pre class="brush:csharp">public class PropertyChangedEventManager : WeakEventManager
{
    // Fields
    private WeakEventManager.ListenerList _proposedAllListenersList;
    private static readonly string AllListenersKey;

    // Methods
    static PropertyChangedEventManager();
    private PropertyChangedEventManager();
    public static void AddListener(INotifyPropertyChanged source, IWeakEventListener listener, string propertyName);
    private void OnPropertyChanged(object sender, PropertyChangedEventArgs args);
    private void PrivateAddListener(INotifyPropertyChanged source, IWeakEventListener listener, string propertyName);
    private void PrivateRemoveListener(INotifyPropertyChanged source, IWeakEventListener listener, string propertyName);
    protected override bool Purge(object source, object data, bool purgeAll);
    public static void RemoveListener(INotifyPropertyChanged source, IWeakEventListener listener, string propertyName);
    protected override void StartListening(object source);
    protected override void StopListening(object source);

    // Properties
    private static PropertyChangedEventManager CurrentManager { get; }
}</pre>

Basically, `AddListener` access the private `CurrentManager` singleton and subscribes the given listener to `OnPropertyChanged`. Fortunately for us, there&#8217;s a flaw in th the implementation of `OnPropertyChanged`. What it does is it gets the list of listeners based on the sender and raises their event with the given sender and args. The problem here is that it doesn&#8217;t verify that the object raising the event is actually the sender! That is to say, we should be able to send a fake PropertyChanged event through another object acting as a surrogate. All we need to do is add that object to the PropertyChangedEvenManager&#8217;s list and start impersonating.

To that end, I added this private class to `BindableBase` to lazily add the `INotifyPropertyChanged` proxy object to the `PropertyChangedEventManager`. Since we&#8217;re not actually interested in the events, I made a stub implementation of `IWeakEventListener` that returns false constantly to indicate it&#8217;s not handling the event. Finally, I hold onto both of these references to keep them from being garbage collected.

<pre class="brush:csharp">private class PropertyChangedEventManagerProxy
{
    // We need to hold on to these refs to keep it from getting GC'd
    private readonly NotifyPropertyChangedProxy _notifyPropertyChangedProxy;
    private readonly IWeakEventListener _weakEventListener;

    private PropertyChangedEventManagerProxy()
    {
        _notifyPropertyChangedProxy = new NotifyPropertyChangedProxy();
        _weakEventListener = new WeakListenerStub();

        PropertyChangedEventManager.AddListener(_notifyPropertyChangedProxy, _weakEventListener, string.Empty);
    }

    public void RaisePropertyChanged(object sender, string propertyName)
    {
        _notifyPropertyChangedProxy.Raise(sender, new PropertyChangedEventArgs(propertyName));
    }

    private static PropertyChangedEventManagerProxy _instance;
    public static PropertyChangedEventManagerProxy Instance { get { return _instance ?? (_instance = new PropertyChangedEventManagerProxy()); } }

    private class NotifyPropertyChangedProxy : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged = delegate { };

        public void Raise(object sender, PropertyChangedEventArgs e)
        {
            PropertyChanged(sender, e);
        }
    }

    private class WeakListenerStub : IWeakEventListener
    {
        public bool ReceiveWeakEvent(Type managerType, object sender, EventArgs e) { return false; }
    }
}</pre>

The only thing left to do is add the coerce value function as an optional parameter on `SetProperty` and hook it up:

<pre class="brush:csharp">protected void SetProperty&lt;T&gt;(
    ref T backingStore,
    T value,
    string propertyName,
    Action onChanged = null,
    Action&lt;T&gt; onChanging = null,
    Func&lt;T, T&gt; coerceValue = null)
{
    VerifyCallerIsProperty(propertyName);

    var effectiveValue = coerceValue != null ? coerceValue(value) : value;

    if (EqualityComparer&lt;T&gt;.Default.Equals(backingStore, effectiveValue))
    {
        // If we coerced this value and the coerced value is not equal to the original, we need to
        // send a fake PropertyChanged event to notify WPF that this value isn't what it thinks it is.
        if (coerceValue != null && !EqualityComparer&lt;T&gt;.Default.Equals(value, effectiveValue))
            PropertyChangedEventManagerProxy.Instance.RaisePropertyChanged(this, propertyName);

        return;
    }

    if (onChanging != null) onChanging(effectiveValue);

    OnPropertyChanging(propertyName);

    backingStore = effectiveValue;

    if (onChanged != null) onChanged();

    OnPropertyChanged(propertyName);
}</pre>

And there you have it. All the benefits of dependency property coercion, and the same short, sweet `SetProperty` syntax.

As before, everything is on [GitHub](https://github.com/danielmoore/InpcTemplate) so you can take a look at the whole thing and fork it if you see something to be improved.