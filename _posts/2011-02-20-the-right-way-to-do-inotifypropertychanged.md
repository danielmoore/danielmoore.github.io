---
id: 397
title: The Right Way to do INotifyPropertyChanged
date: 2011-02-20T17:35:12+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=397
permalink: /2011/the-right-way-to-do-inotifypropertychanged/
categories:
  - Coding
  - Lab49
tags:
  - 'C#'
  - Extension Methods
  - INotifyPropertyChanged
  - Lambda
  - patterns
  - WPF
---
It&#8217;s sad how much controversy there is in doing something as simple as raising property change notification to our subscribers. It seems to me that we should have settled on something by now and moved on to bigger problems, yet, still, I see developers at every level of experience doing it differently.

I want to inform you all that [you&#8217;re doing it wrong](http://xkcd.com/386/).<!--more-->

## The Problem

Most projects choose to implement `INotifyPropertyChanged` instead of extending `DependencyObject` for the sole reason that dependency properties are an enormous pain to write. Having worked on a large project that used the latter approach, I&#8217;ve entirely sworn it off, and I don&#8217;t care what kind of tooling you say makes life better. It&#8217;s just too complicated. On top of that, it&#8217;s really difficult to subscribe to changes programmatically. Sure you can go through `DependencyPropertyDescriptor` and subscribe yourself, but that&#8217;s a hell of a lot harder than `+=`. So that leaves some people with properties that look like this:

<pre class="brush:csharp">private int _myValue;
public int MyValue
{
	get { return _myValue; }
	set { _myValue = value; OnPropertyChanged("MyValue"); }
}</pre>

Not bad, perhaps. It just doesn&#8217;t do much. What about `INotifyPropertyChanging`? How do I do things internally when that property changes other than subscribing to my own `PropertyChanged` event?

These things could almost be overlooked. The real kicker is that this isn&#8217;t actually a proper implementation of `INotifyPropertyChanged`. [According to MSDN](http://msdn.microsoft.com/en-us/library/system.componentmodel.inotifypropertychanged.propertychanged.aspx), the `PropertyChanged` event is described as, &#8220;Occurs when a property value changes,&#8221; and this code will raise an event even if the value doesn&#8217;t. Their implementation looks like this:

<pre class="brush: csharp">private string customerNameValue = String.Empty;
public string CustomerName
{
	get
	{
		return this.customerNameValue;
	}

	set
	{
		if (value != this.customerNameValue)
		{
			this.customerNameValue = value;
			NotifyPropertyChanged("CustomerName");
		}
	}
}</pre>

Aside from their code style (or lack thereof), allow me to point out a few things. First &#8211; and foremost &#8211; they&#8217;re comparing string equality with operators, no mere venial sin in my book. Secondly, this puts the onus of figuring out how to compare two types on the property writer ad-hoc, rather than using something more intelligent. Finally, does anyone else think that 17 lines is a few too many for a _dirt simple_ property? It&#8217;s crazy.

## The Trouble with Lambda

Some very smart programmers I know like to have their `NotifyPropertyChanged` method take an Expression<Func<TProp>> so you can do something like:

<pre class="brush: csharp">private int _myValue;
public int MyValue
{
	get { return _myValue; }
	set { _myValue = value; NotifyPropertyChanged(() =&gt; MyValue); }
}</pre>

They say it improves type checking and the ease of refactoring. But as it turns out, [it&#8217;s god-awful for performance](http://blog.quantumbitdesigns.com/2010/01/26/mvvm-lambda-vs-inotifypropertychanged-vs-dependencyobject/), which can run you into a lot of problems if you don&#8217;t keep track of even moderate update bursts.

I&#8217;d almost be willing to accept the performance trade-off if it weren&#8217;t for the fact that the only place that&#8217;s calling it is _inside the property setter_. When you&#8217;re refactoring, I don&#8217;t see it as too much effort to go ahead and fix the string in the underlying property setter while you&#8217;re there; you have to fix the backing member anyway.

Really what we need is lambdas for subscription. All those places out in your code that subscribe to `PropertyChanged` and then you check the property name with a string! That&#8217;s the real refactoring nightmare. What I want to do is `myViewModel.SubscribeToPropertyChanged(vm => vm.Foo, OnFooChanged);`

The implementation isn&#8217;t that complex at all:

<pre class="brush:csharp">public static IDisposable SubscribeToPropertyChanged&lt;TSource, TProp&gt;(
	this TSource source,
	Expression&lt;Func&lt;TSource, TProp&gt;&gt; propertySelector,
	Action onChanged)
	where TSource : INotifyPropertyChanged
{
	if (source == null) throw new ArgumentNullException("source");
	if (propertySelector == null) throw new ArgumentNullException("propertySelector");
	if (onChanged == null) throw new ArgumentNullException("onChanged");

	var subscribedPropertyName = GetPropertyName(propertySelector);

	PropertyChangedEventHandler handler = (s, e) =&gt;
	{
		if (string.Equals(e.PropertyName, subscribedPropertyName, StringComparison.InvariantCulture))
			onChanged();
	};

	source.PropertyChanged += handler;

	return Disposable.Create(() =&gt; source.PropertyChanged -= handler);
}

public static IDisposable SubscribeToPropertyChanging&lt;TSource, TProp&gt;(
	this TSource source, Expression&lt;Func&lt;TSource,
	TProp&gt;&gt; propertySelector,
	Action onChanging)
	where TSource : INotifyPropertyChanging
{
	if (source == null) throw new ArgumentNullException("source");
	if (propertySelector == null) throw new ArgumentNullException("propertySelector");
	if (onChanging == null) throw new ArgumentNullException("onChanged");

	var subscribedPropertyName = GetPropertyName(propertySelector);

	PropertyChangingEventHandler handler = (s, e) =&gt;
	{
		if (string.Equals(e.PropertyName, subscribedPropertyName, StringComparison.InvariantCulture))
			onChanging();
	};

	source.PropertyChanging += handler;

	return Disposable.Create(() =&gt; source.PropertyChanging -= handler);
}

private static string GetPropertyName&lt;TSource, TProp&gt;(Expression&lt;Func&lt;TSource, TProp&gt;&gt; propertySelector)
{
	var memberExpr = propertySelector.Body as MemberExpression;

	if (memberExpr == null) throw new ArgumentException("must be a member accessor", "propertySelector");

	var propertyInfo = memberExpr.Member as PropertyInfo;

	if (propertyInfo == null || propertyInfo.DeclaringType != typeof(TSource))
		throw new ArgumentException("must yield a single property on the given object", "propertySelector");

	return propertyInfo.Name;
}</pre>

## The Best Solution

This leaves me with a pretty explicit spec for implementing `INotifyPropertyChanged`:

  1. It will take no more lines than a basic backing field setter.
  2. It will raise INotifyPropertyChanged and INotifyPropertyChanging.
  3. It will only raise the properties if the value has changed.
  4. it will provide an easy way to have internal property changed subscriptions.
  5. It will not use Lambdas to identify the property.

My solution allows you to do this:

<pre class="brush: csharp">private int _myValue;
public int MyValue
{
	get { return _myValue; }
	set { SetProperty(ref _myValue, value, "MyValue", OnMyValueChanged, OnMyValueChanging); }
}

private void OnMyValueChanged() { }
private void OnMyValueChanging(int newValue) { }</pre>

Isn&#8217;t that nice? SetProperty looks like this:

<pre class="brush:csharp">protected void SetProperty(
	ref T backingStore, T value,
	string propertyName,
	Action onChanged = null,
	Action onChanging = null)
{
	VerifyCallerIsProperty(propertyName);

	if (EqualityComparer&lt;T&gt;.Default.Equals(backingStore, value)) return;

	if (onChanging != null) onChanging(value);

	OnPropertyChanging(propertyName);

	backingStore = value;

	if (onChanged != null) onChanged();

	OnPropertyChanged(propertyName);
}

[Conditional("DEBUG")]
private void VerifyCallerIsProperty(string propertyName)
{
	var stackTrace = new StackTrace();
	var frame = stackTrace.GetFrames()[2];
	var caller = frame.GetMethod();

	if (!caller.Name.Equals("set_" + propertyName, StringComparison.InvariantCulture))
		throw new InvalidOperationException(
			string.Format("Called SetProperty for {0} from {1}", propertyName, caller.Name))
}</pre>

The default parameters on `SetProperty` allow you to specify the changed callback or the changing callbacks in any permutation with the usual syntax and the stack trace analysis happens at debug time to make sure that the string you provide actually represents the property it&#8217;s being called from.

So how do we update dependent properties? The one thing you don&#8217;t want to do is **ever** have external property changing logic directly in your set method. It&#8217;s better to call out to a `OnMyPropertyChanging` event and have that method update your dependent method. The syntax is a little verbose, but it&#8217;s the most readable and extensible way to do it:

<pre class="brush: csharp">private int _foo;
public int Foo
{
	get { return _foo; }
	set { SetProperty(ref _foo, value, "Foo", OnFooChanged); }
}

private int _bar;
public int Bar
{
	get { return _bar; }
	set { SetProperty(ref _bar, value, "Bar", OnBarChanged); }
}

private int _foobar;
public int Foobar
{
	get { return _foobar; }
	private set { SetProperty(ref _foobar, value, "Foobar"); }
}

private void OnFooChanged() { UpdateFoobar(); }

private void OnBarChanged() { UpdateFoobar(); }

private void UpdateFoobar() { Foobar = _foo + _bar; }</pre>

In general you should **never** call `OnProeprtyChanged`; that decision is best left to the property and its underlying helper.

I hope this clears the water  for some people new to WPF and provides reasons for the veterans to change their tune. If you think I&#8217;m wrong, feel free to try and prove it to me in the comments section. After all, duty calls.

The source and XML documentation for all this and a few tests is on [github](https://github.com/danielmoore/InpcTemplate).