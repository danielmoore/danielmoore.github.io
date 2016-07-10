---
id: 14
title: Variable-length Integers
date: 2009-02-17T14:34:21+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=14
permalink: /2009/variable-length-integers/
categories:
  - Coding
  - Site News
tags:
  - 'C#'
  - Chord
  - Coursework
  - Hashing
  - Math
---
Wow. So it definitely took me long enough to get this blog going. A lot has happened since I got this site up and running, so hopefully there will be no shortage of thoughts to write about.

This semester I&#8217;ve finally qualified to take the CS Senior Design Project, which, this semester, is to design a [Chord](http://en.wikipedia.org/wiki/Chord_project) client. At the heart of the system is the _identifier_, which is essentially a glorified hash. We decided to go with SHA-256, partially because the instructor mentioned it was in the class of his favorite hashes. It didn&#8217;t really matter to me, since C# supports it just as easily as anything else.

Identifiers have two essential purposes, comparison and addition, from which every method can be derived. I wasn&#8217;t particularly intrigued by either of these topics, since almost without exception, you can rely on existing structures in the [FCL](http://en.wikipedia.org/wiki/Framework_Class_Library) to do whatever you need to do, and it&#8217;s always better to do so. Now, I said &#8220;almost without exception,&#8221; namely because I stumbled upon the fact that these identifiers unequivocally are exceptions. The thing about using a SHA-256 hash &#8211; or any hash for that matter &#8211; is that it&#8217;s an enormous value &#8211; 256 bits to be precise. There aren&#8217;t any real 32 or even 64 bit hashes available (I doubt they&#8217;d be useful), so relying on the good ol&#8217; FCL goes out the window.<!--more-->

Other students, stuck in the abyss of Java, were having difficulty getting their socket implementations going, so mucking around in massive integer arithmetic was right out. I believe the professor advised them to take the 32 [MSB](http://en.wikipedia.org/wiki/Most_significant_bit) and just rely on an unsigned integer. _(Side note: does Java even have unsigned integers?)_ I considered a few similar options: taking the 64 [LSB](http://en.wikipedia.org/wiki/Least_significant_bit) or separating the hash into several equal groups of 64-bits each and getting the xor result of the segments. Arrogantly I proceeded on, demanding that whatever I wrote work not only for SHA-256 but also SHA-512 or anything else using the full bit-length.

Comparison wasn&#8217;t difficult. Essentially, you start at the MSB in your array (which happens to be at `Length - 1`) and compare until you find a comparison `!= 0`:

<pre class="brush: csharp">public int CompareTo(Identifier other)
{
	for (int i = _hash.Length - 1; i &gt;= 0; i--)
	{
		int compare = _hash[i].CompareTo(other._hash[i]);

		if (compare != 0)
			return compare;
	}

	return 0;
}</pre>

Now comes the fun part. First, I wrote a little method called `Pad(byte[] list, int size)` which pads the MSB of an array with 0&#8217;s until it&#8217;s the proper length. After several iterations, I finally ended up with this:

<pre class="brush: csharp">public static byte[] Add(byte[] a, byte[] b)
{
	if (a.Length &gt; b.Length)
		b = Pad(b, a.Length);
	else if (a.Length &lt; b.Length)
		a = Pad(a, b.Length);

	byte[] result = new byte[a.Length];

	int carry = 0;
	for (int i = 0; i &lt; a.Length; i++)
	{
		int value = carry + a[i] + b[i];

		// get the bottom part of the bits
		result[i] = (byte)value;

		// knock off the bottom bits that we already got and
		// put the rest into carry.
		carry = (value - result[i]) / byte.MaxValue;
	}

	return result;
}</pre>

The data holder for carry has to be large enough for the worst case scenario `0xFF + 0xFF + carry 1`, which is why I chose an `int` rather than a `byte` (although a short could have worked just as well, I suppose).

Subtraction was more difficult. Consider the way you subtract, with carrying, etc. Carrying on its own is a recursive process, and I can&#8217;t imagine writing it out. It was at that time that I remembered how the [ALU](http://en.wikipedia.org/wiki/Arithmetic_logic_unit) does subtraction- by taking the two&#8217;s compliment of the second operand and doing addition. So I followed suit:

<pre class="brush: csharp">public static byte[] Subtract(byte[] a, byte[] b)
{
    if (a.Length &gt; b.Length)
        b = Pad(b, a.Length);
    else if (a.Length &lt; b.Length)
        a = Pad(a, b.Length);

	byte[] twoscomp = new byte[b.Length];

	// get the 1's compliment
	for (int i = 0; i &lt; b.Length; i++)
		twoscomp[i] = (byte)(byte.MaxValue - b[i]);

	// add 1
	twoscomp = Add(twoscomp, new byte[] { 1 });

	// NOTE: twoscomp is now the two's compliment of b.

	return Add(a, twoscomp);
}</pre>

I also needed multiplication, but I&#8217;m still working on the solution. I&#8217;ve got a working draft, but I&#8217;m not entirely sure I&#8217;m happy with it just yet. I&#8217;ll keep you notified.