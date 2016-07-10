---
id: 98
title: W3C Validation and the Web
date: 2009-03-10T08:00:57+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=98
permalink: /2009/w3c-validation-and-the-web/
categories:
  - Coding
tags:
  - Browser
  - Coding Horror
  - CSS
  - HTML
  - W3C
  - W3C Validation
  - Web Standards
  - XHTML
---
I try to keep up on several blogs, one of which is Jeff Atwood&#8217;s [Coding Horror](http://codinghorror.com). Recently, he chose the topic of W3 Validation and its necessity, or lack thereof. I also seem to have made a [few statements](http://northhorizon.net/2009/browser-compatibility-and-the-sanity-of-web-developers/) on a similar topic, so perhaps my view is nothing short of expected. What _is_ strange, however, is Jeff&#8217;s point of view, considering he and I are kindred spirits in the world of .NET and C#.<!--more-->

Jeff&#8217;s argument is pretty simple:

  * The web is a forgiving place.
  * Many big-name websites don&#8217;t pass validation.
  * James Bennett doesn&#8217;t like XHTML.
  * CSS isn&#8217;t as intuitive as HTML-embedded properties.
  * Validity is relative to your standards.

Since Jeff &#8220;vehemently agrees&#8221; with Mr. Bennett, I decided to investigate [his article](http://www.b-list.org/weblog/2008/jun/18/html/) as well, to try and understand where Atwood was coming from. Benett&#8217;s argument is even more basic:

  * No real advantages.
  * Markup errors are fatal.
  * There is some CSS/DOM incompatibility between HTML 4.01 and XHTML.

The majority of these points, are, in my opinion, rooted in ignorance and xenophobia, both of which are disastrously fatal in our industry, _especially_ on the Web. It is true the Web is very forgiving, but only because it must be so. HTML is a novel language, in the sense that you don&#8217;t move from line to line, from procedure to procedure, but rather declare the structure of how something looks and let the renderer (IE, Firefox, Safari) choose the best way to view it. A good example is when Firefox switched from an iterative to a recursive implementation in its third version and also reimplemented its graphics engine. Because of the declarative nature of HTML, websites weren&#8217;t adversely impacted. However, like all new things, HTML has its quirks. Namely, it&#8217;s not very easy for computers to understand. In fact, sometimes it&#8217;s not all that easy for even humans to understand &#8211; maybe a sort of inverted Turing test.

Like all the triumphs in Computer Science, someone identified the problems that exist in HTML and created a far more flexible and extensible language called XML. XML is now one of the best containers for information because of its well-designed and understandable structure and its infinite flexibility. While that&#8217;s great for data storage, what about the Web? Well, that&#8217;s where XHTML 1.0 came in, as a statically defined language like HTML, but a machine-understandable and consistent language, like XML. XHTML, is, at its very essence, the best of both worlds.

I think Atwood and Bennett had a similar reaction to XHTML [as I did](http://northhorizon.net/2009/browser-compatibility-and-the-sanity-of-web-developers/):

> Back when XHTML went 1.0, I was somewhat surprised to see that they had actually removed elements and attributes from the specification. Usually, moving to a new product means that you get more, not less.

The difference here is that they refused, either consciously or unconsciously to adapt to and understand where the Web was &#8211; and still is &#8211; going. The W3C is going for a very specific separation of concerns for websites. XHTML is designed to be the static structure of a website. It contains data (text and such), and some hints on how to present it (whether it be in paragraphs, tabular, or a list), but nothing else. This is where CSS comes in. CSS decorates the XHTML, quantifying distances, sizes, backgrounds, and organization. Finally, JavaScript gives the client a smooth feel, dynamically altering the web page to best suit the client, and, as of late, retrieve data on the fly through AJAX and JSON. This is why HTML attributes are almost all deprecated in XHTML &#8211; the static structure doesn&#8217;t care how wide things are or how tall they are. One of the best examples that I can think of to demonstrate the division of power between XHTML and CSS is [CSS Zen Garden](http://www.csszengarden.com/). The opening page is modest enough, but choose one of the designs on the right sidebar and the entire layout changes. The only difference between the default and new design is a new CSS style sheet. You&#8217;ll notice it has all the same text, all the same navigation, but a completely different look. Why is this useful or important? Well, as a form of broadcast media, it is salient to shape the way your website looks based on the viewer. For instance, if you go to a website on an web-enabled cellphone, I&#8217;d want to strip out as many ancillary graphics as possible and conserve as much width as possible. If you&#8217;re vision impaired, I want to be able to deliver to you a web page that a TTY device can easily understand. Or, perhaps, you&#8217;re a search engine and I want to get straight to the point so you can index better. This should all be easily accomplished by specifying the use and intention of each style sheet and allowing the user to choose the one that best suits their purpose.

Of course, someone always brings up the fact that many name-brand websites aren&#8217;t valid at all. I&#8217;d also like to point out that many big-name trading firms are going bankrupt. Just because they don&#8217;t care doesn&#8217;t mean you shouldn&#8217;t. The argument itself is infantile. I&#8217;m not trying to inflate my ego any (trust me, that&#8217;d be a bad idea), but I find writing XHTML 1.1 (which is by far the strictest) to be very easy. When Marlene and I drew up the static format for this website (that is, before we plugged in all the WordPress stuff), I had about three validation errors, all of which were quelled very easily. Personally, I was a little disappointed I had _any_, but I suppose perfection was never a trait I had.

Jeff also wonders whether CSS validation is possible. I&#8217;m not entirely sure why he didn&#8217;t bother to [check Google](http://www.google.com/search?hl=en&client=firefox-a&rls=org.mozilla%3Aen-US%3Aofficial&hs=zyV&newwindow=1&q=css+validation&btnG=Search). Of course CSS validation exists, and I&#8217;m proud to say this website&#8217;s CSS is valid as well.

It&#8217;s unfortunate that XHTML markup errors are indeed fatal (and the errors aren&#8217;t always very helpful), but as a C# developer, I&#8217;m used to having compiled code. If I messed up and miscommunicated with the machine (again, perfection is not one of my virtues), I want the machine to tell me that it doesn&#8217;t understand, rather than acting like my girlfriend, misinterpreting what I said, and summarily stating that I don&#8217;t love her anymore. Obviously it&#8217;s simply untrue; I will always love my computer. Girlfriends are a bit different.

So, perhaps the most important question is, why do we need standards? The need for validation clearly hinges upon the need for standards; without validation, how can one tell if standards are being upheld? And without standards, the Internet would be far more segmented than it is already. Jeff is right that most people don&#8217;t understand the W3 Valid &#8220;stickers&#8221; some webmasters (like myself) have on their websites in various locations. This doesn&#8217;t really bother me, though. Most people don&#8217;t notice half the validations on many of the products they use everyday. On the inside of the battery cover of your cellphone, there exist a multitude of symbols, each representing a different standards body that approved this device to work with the network. Yet millions of people blithely continue on, blissfully ignorant of the years of effort that go into making such a device. Such is the thankless life of an engineer. No one notices when you make amazing accomplishments (and a W3C valid website is hardly a major feat), but everyone complains when the slightest flaw presents itself.

I suppose we just learn to deal with it.