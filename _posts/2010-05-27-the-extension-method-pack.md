---
id: 220
title: The Extension Method Pack
date: 2010-05-27T06:19:26+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=220
permalink: /2010/the-extension-method-pack/
categories:
  - Coding
  - Lab49
tags:
  - 'C#'
  - Dependency Object
  - Extension Methods
  - Hash Table
  - Lambda
  - WPF
  - XMP
---
Since .NET 3.0 came out, I&#8217;ve been enjoying taking advantage of extension methods and the ability to create my own. The thing I&#8217;ve noticed is that a handful of them are useful to almost any application, above and beyond what Microsoft provides in `System.Linq`. So over the last few days I took the time to gather these methods together, unit test them, and run them through FXCop to make a high-quality package ready to go in any application with a little re-namespacing.

I&#8217;ve broken each code sample into independent blocks wherein all necessary dependencies are contained, so you can take any extension method _a la carte_ or you can get everything from the [attached zip file](http://northhorizon.net/wp-content/uploads/2010/05/Extension-Method-Pack.zip). My solution was built in .NET 4.0 in Visual Studio 2010, but everything should work just fine in .NET 3.5 with Visual Studio 2008.

Also included in the zip file are my unit tests, which may help you understand usage of some of the more esoteric extensions, such as ChainGet, and XML comments for your IntelliSense and XML documentation generator.

**Edit:** The whole solution is now available on [github](https://github.com/danielmoore/Extension-Method-Pack)!

<!--more-->

Here&#8217;s the table of contents, so you can jump around more easily:
  
<a name="toc"></a>

  * IEnumerable 
      * [ForEach](#IEnumerable.Foreach)
      * [Append](#IEnumerable.Append)
      * [Prepend](#IEnumerable.Prepend)
      * [AsObservable](#IEnumerable.AsObservable)
      * [AsHashSet](#IEnumerable.AsHashSet)
      * [ArgMax](#IEnumerable.ArgMax)
      * [ArgMin](#IEnumerable.ArgMin)
  * ICollection 
      * [AddAll](#ICollection.AddAll)
      * [RemoveAll](#ICollection.RemoveAll)
  * IDictionary 
      * [Add](#IDictionary.Add)
      * [AddAll](#IDictionary.AddAll)
      * [Remove](#IDictionary.Remove)
      * [RemoveAll](#IDictionary.RemoveAll)
      * [RemoveAndClean](#IDictionary.RemoveAndClean)
      * [RemoveAllAndClean](#IDictionary.RemoveAllAndClean)
      * [Clean](#IDictionary.Clean)
  * Object 
      * [As](#Object.As)
      * [AsValueType](#Object.AsValueType)
      * [ChainGet](#Object.ChainGet)
  * DependencyObject 
      * [SafeGetValue](#DependencyObject.SafeGetValue)
      * [SafeSetValue](#DependencyObject.SafeSetValue)

## IEnumerable

<a name="IEnumerable.ForEach"></a>`ForEach` is pretty straightforward. It mimics `List<T>.ForEach`, but for all `IEnumerable`, both generic and weakly typed.

<pre class="brush: csharp">public static void ForEach&lt;T&gt;(
	this IEnumerable&lt;T&gt; collection,
	Action&lt;T&gt; action)
{
	if (collection == null)
		throw new ArgumentNullException("collection");

	if (action == null)
		throw new ArgumentNullException("action");

	foreach (var item in collection)
		action(item);
}</pre>

<pre class="brush: csharp">public static void ForEach(
	this IEnumerable collection,
	Action&lt;object&gt; action)
{
	if (collection == null)
		throw new ArgumentNullException("collection");

	if (action == null)
		throw new ArgumentNullException("action");

	foreach (var item in collection)
		action(item);
}</pre>

[Table of Contents](#toc){.more-link.alignright}

<a name="IEnumerable.Append"></a><a name="IEnumerable.Prepend"></a>`Append` and `Prepend` simply take an item and return a a new `IEnumerable<T>` with that item on the end or beginning, respectively. `Prepend` is the equivalent of the `[[cons]]` operation to a list.

<pre class="brush: csharp">public static IEnumerable&lt;T&gt; Append&lt;T&gt;(
	this IEnumerable&lt;T&gt; collection,
	T item)
{
	if (collection == null)
		throw new ArgumentNullException("collection");

	if (item == null)
		throw new ArgumentNullException("item");

	foreach (var colItem in collection)
		yield return colItem;

	yield return item;
}</pre>

<pre class="brush: csharp">public static IEnumerable&lt;T&gt; Prepend&lt;T&gt;(
	this IEnumerable&lt;T&gt; collection,
	T item)
{
	if (collection == null)
		throw new ArgumentNullException("collection");

	if (item == null)
		throw new ArgumentNullException("item");

	yield return item;

	foreach (var colItem in collection)
		yield return colItem;
}</pre>

[Table of Contents](#toc){.more-link.alignright}

<a name="IEnumerable.AsObservable"></a><a name="IEnumerable.AsHashSet"></a>`AsObservable` and `AsHashSet` yield their respective data structures, but check to see if they are already what you want, saving valuable time when dealing with interfaces.

<pre class="brush: csharp">public static ObservableCollection&lt;T&gt; AsObservable&lt;T&gt;(
	this IEnumerable&lt;T&gt; collection)
{
	if (collection == null)
		throw new ArgumentNullException("collection");

	return collection as ObservableCollection&lt;T&gt; ??
		new ObservableCollection&lt;T&gt;(collection);
}</pre>

<pre class="brush: csharp">public static HashSet&lt;T&gt; AsHashSet&lt;T&gt;(this IEnumerable&lt;T&gt; collection)
{
	if (collection == null)
		throw new ArgumentNullException("collection");

	return collection as HashSet&lt;T&gt; ?? new HashSet&lt;T&gt;(collection);
}</pre>

[Table of Contents](#toc){.more-link.alignright}

<a name="IEnumerable.ArgMax"></a><a name="IEnumerable.ArgMin"></a>`[[Arg max|ArgMax]]` and `ArgMin` are corollaries to `Max` and `Min` in the `System.Linq` namespace, but return the item in the list that produced the highest value from `Max` or least value from `Min`, respectively.

<pre class="brush: csharp">public static T ArgMax&lt;T, TValue&gt;(
	this IEnumerable&lt;T&gt; collection,
	Func&lt;T, TValue&gt; function)
	where TValue : IComparable&lt;TValue&gt;
{
	return ArgComp(collection, function, GreaterThan);
}

private static bool GreaterThan&lt;T&gt;(T first, T second)
	where T : IComparable&lt;T&gt;
{
	return first.CompareTo(second) &gt; 0;
}

public static T ArgMin&lt;T, TValue&gt;(
	this IEnumerable&lt;T&gt; collection,
	Func&lt;T, TValue&gt; function)
	where TValue : IComparable&lt;TValue&gt;
{
	return ArgComp(collection, function, LessThan);
}

private static bool LessThan&lt;T&gt;(T first, T second) where T : IComparable&lt;T&gt;
{
	return first.CompareTo(second) &lt; 0;
}

private static T ArgComp&lt;T, TValue&gt;(
	IEnumerable&lt;T&gt; collection, Func&lt;T, TValue&gt; function,
	Func&lt;TValue, TValue, bool&gt; accept)
	where TValue : IComparable&lt;TValue&gt;
{
	if (collection == null)
		throw new ArgumentNullException("collection");

	if (function == null)
		throw new ArgumentNullException("function");

	var isSet = false;
	var maxArg = default(T);
	var maxValue = default(TValue);

	foreach (var item in collection)
	{
		var value = function(item);
		if (!isSet || accept(value, maxValue))
		{
			maxArg = item;
			maxValue = value;
			isSet = true;
		}
	}

	return maxArg;
}</pre>

[Table of Contents](#toc){.more-link.alignright}

## ICollection

<a name="ICollection.AddAll"></a>`AddAll` imitates `List<T>.AddRange`.

<pre class="brush: csharp">public static void AddAll&lt;T&gt;(
	this ICollection&lt;T&gt; collection,
	IEnumerable&lt;T&gt; additions)
{
	if (collection == null)
		throw new ArgumentNullException("collection");

	if (additions == null)
		throw new ArgumentNullException("additions");

	if (collection.IsReadOnly)
		throw new InvalidOperationException("collection is read only");

	foreach (var item in additions)
		collection.Add(item);
}</pre>

[Table of Contents](#toc){.more-link.alignright}

<a name="ICollection.RemoveAll"></a>`RemoveAll` imitates `List<T>.RemoveAll`. A second overload allows you to specify the removals if you already have them.

<pre class="brush: csharp">public static IEnumerable&lt;T&gt; RemoveAll&lt;T&gt;(
	this ICollection&lt;T&gt; collection,
	Predicate&lt;T&gt; predicate)
{
	if (collection == null)
		throw new ArgumentNullException("collection");

	if (predicate == null)
		throw new ArgumentNullException("predicate");

	if (collection.IsReadOnly)
		throw new InvalidOperationException("collection is read only");

	// we can't possibly remove more than the entire list.
	var removals = new List&lt;T&gt;(collection.Count);

	// this is an O(n + m * k) operation where n is collection.Count,
	// m is removals.Count, and K is the removal operation time. Because
	// we know n &gt;= m, this is an O(n + n * k) operation or just O(n * k).

	foreach (var item in collection)
		if (predicate(item))
			removals.Add(item);

	foreach (var item in removals)
		collection.Remove(item);

	return removals;
}</pre>

<pre class="brush: csharp">public static void RemoveAll&lt;T&gt;(
	this ICollection&lt;T&gt; collection,
	IEnumerable&lt;T&gt; removals)
{
	if (collection == null)
		throw new ArgumentNullException("collection");

	if (removals == null)
		throw new ArgumentNullException("removals");

	if (collection.IsReadOnly)
		throw new InvalidOperationException("collection is read only");

	foreach (var item in removals)
		collection.Remove(item);
}</pre>

[Table of Contents](#toc){.more-link.alignright}

## IDictionary

All of these methods have to do with [[Hash table|hash tables]]. I use them pretty frequently and these methods are able to make life a lot easier.

<a name="IDictionary.Add"></a><a name="IDictionary.AddAll"></a>`Add` and `AddAll` insert the key and a new collection into the dictionary if the key doesn&#8217;t already exist and then adds the item or items to the collection.

<pre class="brush: csharp">public static void Add&lt;TKey, TCol, TItem&gt;(
	this IDictionary&lt;TKey, TCol&gt; dictionary,
	TKey key,
	TItem item)
	where TCol : ICollection&lt;TItem&gt;, new()
{
	if (dictionary == null)
		throw new ArgumentNullException("dictionary");

	if (key == null)
		throw new ArgumentNullException("key");

	TCol col;
	if (dictionary.TryGetValue(key, out col))
	{
		if (col.IsReadOnly)
			throw new InvalidOperationException("bucket is read only");
	}
	else
		dictionary.Add(key, col = new TCol());

	col.Add(item);
}</pre>

<pre class="brush: csharp">public static void AddAll&lt;TKey, TCol, TItem&gt;(
	this IDictionary&lt;TKey, TCol&gt; dictionary,
	TKey key,
	IEnumerable&lt;TItem&gt; additions)
	where TCol : ICollection&lt;TItem&gt;, new()
{
	if (dictionary == null)
		throw new ArgumentNullException("dictionary");

	if (key == null)
		throw new ArgumentNullException("key");

	if (additions == null)
		throw new ArgumentNullException("additions");

	TCol col;
	if (!dictionary.TryGetValue(key, out col))
		dictionary.Add(key, col = new TCol());

	foreach (var item in additions)
		col.Add(item);
}</pre>

[Table of Contents](#toc){.more-link.alignright}

<a name="IDictionary.Remove"></a><a name="IDictionary.RemoveAll"></a>`Remove` and `RemoveAll` simply remove items from the collection associated with the specified key, if there is one. For the predicate overloads, you need to explicitly construct your `Predicate<T>` delegate because the C# compiler has trouble doing the type inference.

<pre class="brush: csharp">public static void Remove&lt;TKey, TCol, TItem&gt;(
	this IDictionary&lt;TKey, TCol&gt; dictionary,
	TKey key,
	TItem item)
	where TCol : ICollection&lt;TItem&gt;
{
	if (dictionary == null)
		throw new ArgumentNullException("dictionary");

	if (key == null)
		throw new ArgumentNullException("key");

	TCol col;
	if (dictionary.TryGetValue(key, out col))
		col.Remove(item);
}</pre>

<pre class="brush: csharp">public static void RemoveAll&lt;TKey, TCol, TItem&gt;(
	this IDictionary&lt;TKey, TCol&gt; dictionary,
	TKey key,
	IEnumerable&lt;TItem&gt; removals)
	where TCol : ICollection&lt;TItem&gt;
{
	if (dictionary == null)
		throw new ArgumentNullException("dictionary");

	if (key == null)
		throw new ArgumentNullException("key");

	if (removals == null)
		throw new ArgumentNullException("removals");

	TCol col;
	if (dictionary.TryGetValue(key, out col))
		foreach (var item in removals)
			col.Remove(item);
}</pre>

<pre class="brush: csharp">public static IEnumerable&lt;TItem&gt; RemoveAll&lt;TKey, TCol, TItem&gt;(
	this IDictionary&lt;TKey, TCol&gt; dictionary,
	TKey key,
	Predicate&lt;TItem&gt; predicate)
	where TCol : ICollection&lt;TItem&gt;
{
	if (dictionary == null)
		throw new ArgumentNullException("dictionary");

	if (key == null)
		throw new ArgumentNullException("key");

	if (predicate == null)
		throw new ArgumentNullException("predicate");

	var removals = new List&lt;TItem&gt;();

	TCol col;
	if (dictionary.TryGetValue(key, out col))
	{
		foreach (var item in col)
			if (predicate(item))
				removals.Add(item);

		foreach (var item in removals)
			col.Remove(item);
	}

	return removals;
}

// Usage:
Dictionary&lt;int, List&lt;int&gt;&gt; myDictionary;
myDictionary.RemoveAll(4, new Predicate&lt;int&gt;(i =&gt; i &lt; 42));</pre>

[Table of Contents](#toc){.more-link.alignright}

<a name="IDictionary.RemoveAndClean"></a><a name="IDictionary.RemoveAllAndClean"></a>`RemoveAndClean` and `RemoveAllAndClean` both remove items from the collection associated with the specified key and if the resulting collection is empty, they remove the key from the dictionary as well. These come in both predicate and list forms.

<pre class="brush: csharp">public static void RemoveAndClean&lt;TKey, TCol, TItem&gt;(
	this IDictionary&lt;TKey, TCol&gt; dictionary,
	TKey key,
	TItem item)
	where TCol : ICollection&lt;TItem&gt;
{
	if (dictionary == null)
		throw new ArgumentNullException("dictionary");

	if (key == null)
		throw new ArgumentNullException("key");

	TCol col;
	if (dictionary.TryGetValue(key, out col))
	{
		col.Remove(item);

		if (col.Count == 0)
			dictionary.Remove(key);
	}
}</pre>

<pre class="brush: csharp">public static void RemoveAllAndClean&lt;TKey, TCol, TItem&gt;(
	this IDictionary&lt;TKey, TCol&gt; dictionary,
	TKey key,
	IEnumerable&lt;TItem&gt; removals)
	where TCol : ICollection&lt;TItem&gt;
{
	if (dictionary == null)
		throw new ArgumentNullException("dictionary");

	if (key == null)
		throw new ArgumentNullException("key");

	if (removals == null)
		throw new ArgumentNullException("removals");

	TCol col;
	if (dictionary.TryGetValue(key, out col))
	{
		foreach (var item in removals)
			col.Remove(item);

		if (col.Count == 0)
			dictionary.Remove(key);
	}
}</pre>

<pre class="brush: csharp">public static IEnumerable&lt;TItem&gt; RemoveAllAndClean&lt;TKey, TCol, TItem&gt;(
	this IDictionary&lt;TKey, TCol&gt; dictionary,
	TKey key,
	Predicate&lt;TItem&gt; predicate)
	where TCol : ICollection&lt;TItem&gt;
{
	if (dictionary == null)
		throw new ArgumentNullException("dictionary");

	if (key == null)
		throw new ArgumentNullException("key");

	if (predicate == null)
		throw new ArgumentNullException("predicate");

	var removals = new List&lt;TItem&gt;();

	TCol col;
	if (dictionary.TryGetValue(key, out col))
	{
		foreach (var item in col)
			if (predicate(item))
				removals.Add(item);

		foreach (var item in removals)
			col.Remove(item);

		if (col.Count == 0)
			dictionary.Remove(key);
	}

	return removals;
}

// Usage:
Dictionary&lt;int, List&lt;int&gt;&gt; myDictionary;
myDictionary.RemoveAll(4, new Predicate&lt;int&gt;(i =&gt; i &lt; 42));</pre>

[Table of Contents](#toc){.more-link.alignright}

<a name="IDictionary.Clean"></a>`Clean` simply goes through all the keys and removes entries with empty collections.

<pre class="brush: csharp">public static void Clean&lt;TKey, TCol&gt;(this IDictionary&lt;TKey, TCol&gt; dictionary)
	where TCol : ICollection
{
	if (dictionary == null)
		throw new ArgumentNullException("dictionary");

	var keys = dictionary.Keys.ToList();

	foreach (var key in keys)
		if (dictionary[key].Count == 0)
			dictionary.Remove(key);
}</pre>

[Table of Contents](#toc){.more-link.alignright}

## Object

<a name="Object.As"></a>As casts an object to a specified type, executes an action with it, and returns whether or not the cast was successful.

<pre class="brush: csharp">public static bool As&lt;T&gt;(this object obj, Action&lt;T&gt; action)
	where T : class
{
	if (obj == null)
		throw new ArgumentNullException("obj");

	if (action == null)
		throw new ArgumentNullException("action");

	var target = obj as T;
	if (target == null)
		return false;

	action(target);
	return true;
}</pre>

[Table of Contents](#toc){.more-link.alignright}

<a name="Object.AsValueType"></a>`AsValueType` does the same thing as `As`, but for value types.

<pre class="brush: csharp">public static bool AsValueType&lt;T&gt;(this object obj, Action&lt;T&gt; action)
	where T : struct
{
	if (obj == null)
		throw new ArgumentNullException("obj");

	if (action == null)
		throw new ArgumentNullException("action");

	if (obj is T)
	{
		action((T)obj);
		return true;
	}

	return false;
}</pre>

[Table of Contents](#toc){.more-link.alignright}

<a name="Object.ChainGet"></a>`ChainGet` attempts to resolve a chain of member accesses and returns the result or `default(TValue)`. Since `TValue` could be a value type, there is also an overload that has an out parameter indicating whether the value was obtained.

Be careful with this extension. Since it uses reflection, it&#8217;s a bit slow. For discrete usage, each call is under 1 ms, but if you use it in a loop with many items, the performance hit will become more tangible.

<pre class="brush: csharp">public static TValue ChainGet&lt;TRoot, TValue&gt;(
	this TRoot root,
	Expression&lt;Func&lt;TRoot, TValue&gt;&gt; getExpression)
{
	bool success;
	return ChainGet(root, getExpression, out success);
}

public static TValue ChainGet&lt;TRoot, TValue&gt;(
	this TRoot root,
	Expression&lt;Func&lt;TRoot, TValue&gt;&gt; getExpression,
	out bool success)
{
	// it's ok if root is null!

	if (getExpression == null)
		throw new ArgumentNullException("getExpression");

	var members = new Stack&lt;MemberAccessInfo&gt;();

	Expression expr = getExpression.Body;
	while (expr != null)
	{
		if (expr.NodeType == ExpressionType.Parameter)
			break;

		var memberExpr = expr as MemberExpression;
		if (memberExpr == null)
			throw new ArgumentException(
				"Given expression is not a member access chain.",
				"getExpression");

		members.Push(new MemberAccessInfo(memberExpr.Member));

		expr = memberExpr.Expression;
	}

	object node = root;
	foreach (var member in members)
	{
		if (node == null)
		{
			success = false;
			return default(TValue);
		}

		node = member.GetValue(node);
	}

	success = true;
	return (TValue)node;
}

private class MemberAccessInfo
{
	private PropertyInfo _propertyInfo;
	private FieldInfo _fieldInfo;

	public MemberAccessInfo(MemberInfo info)
	{
		_propertyInfo = info as PropertyInfo;
		_fieldInfo = info as FieldInfo;
	}

	public object GetValue(object target)
	{
		if (_propertyInfo != null)
			return _propertyInfo.GetValue(target, null);
		else if (_fieldInfo != null)
			return _fieldInfo.GetValue(target);
		else
			throw new InvalidOperationException();
	}
}

// Usage:
var myValue = obj.ChainGet(o =&gt; o.MyProperty.MySubProperty.MySubSubProperty.MyValue);</pre>

[Table of Contents](#toc){.more-link.alignright}

## DependencyObject

<a name="DependencyObject.SafeGetValue"></a><a name="DependencyObject.SafeSetValue"></a>These extensions work just like their non-safe counterparts on `DependencyObject`, but will call `Dispatcher.Invoke` to do operations if the current thread isn&#8217;t a UI thread.

<pre class="brush: csharp">public static object SafeGetValue(
	this DependencyObject obj,
	DependencyProperty dp)
{
	if (obj == null)
		throw new ArgumentNullException("obj");

	if (dp == null)
		throw new ArgumentNullException("dp");

	if (obj.CheckAccess())
		return obj.GetValue(dp);

	var self = new Func
		&lt;DependencyObject, DependencyProperty, object&gt;
		(SafeGetValue);

	return Dispatcher.Invoke(self, obj, dp);
}</pre>

<pre class="brush: csharp">public static void SafeSetValue(
	this DependencyObject obj,
	DependencyProperty dp,
	object value)
{
	if (obj == null)
		throw new ArgumentNullException("obj");

	if (dp == null)
		throw new ArgumentNullException("dp");

	if (obj.CheckAccess())
		obj.SetValue(dp, value);
	else
	{
		var self = new Action
			&lt;DependencyObject, DependencyProperty, object&gt;
			(SafeSetValue);
		Dispatcher.Invoke(self, obj, dp, value);
	}
}</pre>

<pre class="brush: csharp">public static void SafeSetValue(
	this DependencyObject obj,
	DependencyPropertyKey key,
	object value)
{
	if (obj == null)
		throw new ArgumentNullException("obj");

	if (key == null)
		throw new ArgumentNullException("key");

	if (obj.CheckAccess())
		obj.SetValue(key, value);
	else
	{
		var self = new Action
			&lt;DependencyObject, DependencyPropertyKey, object&gt;
			(SafeSetValue);
		Dispatcher.Invoke(self, obj, key, value);
	}
}</pre>

[Table of Contents](#toc){.more-link.alignright}