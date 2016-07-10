---
id: 47
title: Browser Compatibility and the Sanity of Web Developers
date: 2009-02-19T10:47:31+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=47
permalink: /2009/browser-compatibility-and-the-sanity-of-web-developers/
categories:
  - Coding
tags:
  - Architecture
  - Browser
  - Chrome
  - CSS
  - CSS floats
  - Firefox
  - IE
  - Opera
  - Safari
---
Since I&#8217;ve decided to get this blog rolling, one of my primary objectives was to ensure a consistent, high quality experience for any platform visitors come from. Obviously, it&#8217;s impossible to test the myriad of browsers out there, but I think it&#8217;s useful to at least test on the top three engines, namely Trident (Internet Explorer), Gecko (Firefox), and WebKit (Safari / Chrome). I also test on Opera, even though it has [little market share](http://arstechnica.com/microsoft/news/2009/02/january-2009.ars). Hopefully I&#8217;ll get my traffic statistics going again soon, so  I can prove more definitively that sixty something percent of the ten people who come here use Firefox anyway.

So that leave three or so of you guys using IE. Well, I hate to break it to you, but your browser sucks. It really does. Web developers everywhere have been rejecting Microsoft&#8217;s browser for some time now, which has given rise to the healthy market share Firefox has had.

How does this affect you and me, you may be wondering. Well, let&#8217;s do a case study on the site you&#8217;re looking at right now, since I&#8217;m fixing it up anyway.<!--more--> For those of you not using a W3C compliant browser (read: IE), the website is supposed to have some blocks of color of varying height below each of the navigation links, which highlight to orange and a main body text that wraps around the sidebar on the right.<figure id="attachment_59" style="width: 150px" class="wp-caption aligncenter">

[<img class="size-thumbnail wp-image-59" title="northhorizon-ff" src="http://northhorizon.net/wp-content/uploads/2009/02/northhorizon-ff-150x150.png" alt="northhorizon-ff" width="150" height="150" />](http://northhorizon.net/wp-content/uploads/2009/02/northhorizon-ff.png)<figcaption class="wp-caption-text">My post about hashing as seen in Firefox.</figcaption></figure> 

Beautiful. Everything looks just like it should in Firefox, Safari, Chrome, and Opera. But what happens when we look at it in IE?<figure id="attachment_61" style="width: 150px" class="wp-caption aligncenter">

[<img class="size-thumbnail wp-image-61" title="northhorizon-ie7" src="http://northhorizon.net/wp-content/uploads/2009/02/northhorizon-ie7-150x150.png" alt="northhorizon-ie7" width="150" height="150" />](http://northhorizon.net/wp-content/uploads/2009/02/northhorizon-ie7.png)<figcaption class="wp-caption-text">The same post as seen by Internet Explorer 7</figcaption></figure> 

Oh wow. Not what I was going for at all. This is quite common for web developers to experience: write a website and make it perfect for the W3C compliant browsers and then try and get it to work in IE, because in all honesty, IE makes very little sense in terms of expected results. On another project, I instructed the browser to move an object 6 pixels left and 10 pixels down. Firefox was happy to oblige, but IE only moved it 3 pixels left and 8 pixels down.

So what&#8217;s going on here at the root, so we can fix it? Well, if you inspect my CSS, you&#8217;ll find this tidbit:

<pre lang="css">#sidebar {
	float: right;
	width: 250px;
	list-style: none outside;
	background: white;
	margin: 0 0 0 10px;
	padding: 5px;
	padding-left: 20px;
}</pre>

Most notably, I&#8217;m using CSS floats to put the Sidebar on the right. By doing so, I&#8217;m instructing the browser to make some space for it and to wrap text around it. IE obviously doesn&#8217;t care about my floated `div`, so it just disregards the instruction. That leaves me in a precarious position. Should I abandon my floats, which lead to nicer, cleaner XHTML in favor of tables because of one non-compliant browser? Of course not! Instead I have a little function that tells me what browser you&#8217;re using called [`get_browser()`](http://us3.php.net/manual/en/function.get-browser.php). The code looks like this:

<pre lang="php">$browscap = get_browser();
define('IE', ($browscap-&gt;browser == 'IE'));</pre><figure id="attachment_64" style="width: 150px" class="wp-caption alignright">

[<img class="size-thumbnail wp-image-64" title="northhorizon-ie7-fixed" src="http://northhorizon.net/wp-content/uploads/2009/02/northhorizon-ie7-fixed-150x150.png" alt="northhorizon-ie7-fixed" width="150" height="150" />](http://northhorizon.net/wp-content/uploads/2009/02/northhorizon-ie7-fixed.png)<figcaption class="wp-caption-text">The same post in Internet Explorer 7 with my fixes.</figcaption></figure> 

This gives me a global constant, `IE` that allows me to change how the website looks if you&#8217;re running IE. With this in mind, I actually insert in a table in the middle of the page to force the sidebar to stay on the right. Problem solved.

As I was doing all this, I wondered if IE8 would give me any relief. I was somewhat surprised to find that it does, in fact, respect floating `div`s, and even respects the :after pseudo classes that allow me to add those blocks under the navigation links. Just not the hovering. _Sigh_. Anyway, all I needed to do was modify my IE code to check the major version number of the browser to say that IE8+ was <span style="text-decoration: line-through;">really</span> mostly W3C compliant.

<pre lang="php">$browscap = get_browser();
define('IE', ($browscap-&gt;browser == 'IE' && $browscap-&gt;majorver &lt; 8));</pre>

So why is IE this way fundamentally? I don&#8217;t have any real code or facts to base these assumptions on, but my speculation is based on working with both XHTML and CSS as well as C# / WinForms / WPF. I assert to you that Microsoft has misunderstood the Web since its inception. With things like [ActiveX](http://en.wikipedia.org/wiki/ActiveX) (now, thankfully, deprecated) it seems like Microsoft has always wanted to bring Windows to the web &#8211; literally. In Windows Forms (the battleship gray programs we all used to know), each visual element is a very unique item. Buttons don&#8217;t behave very much like Panels which don&#8217;t behave very much like CheckBoxes. It follows that you can&#8217;t really apply properties that relate to a Button to a Panel. Sensible enough. Unlees you&#8217;re dealing with XHTML and CSS. Back when XHTML went 1.0, I was somewhat surprised to see that they had actually _removed_ elements and attributes from the specification. Usually, moving to a new product means that you get _more_, not less. It took some time, but I eventually realized that the Web isn&#8217;t like WinForms at all. Each element on the web (with the exception of form controls like ComboBoxes and CheckBoxes) is really the same. Almost everything can be broken down into a div or a span, and really if you change the display attribute on them, you end up with one amorphous panel.

This is the fundamental difference between IE and the W3C triad. Microsoft insists that floats and hovers really shouldn&#8217;t apply to things like divs, because they come from two seperate branches in the object tree. Really, until Microsoft realizes this, throws the existing codebase out, and tries again, IE will never be able to keep up with the rest. It&#8217;s a matter of understanding the nature of the Web and using an appropriate architecture to interpret it.