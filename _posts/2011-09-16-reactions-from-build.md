---
id: 476
title: Reactions from Build
date: 2011-09-16T17:51:02+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=476
permalink: /2011/reactions-from-build/
categories:
  - Coding
  - Lab49
tags:
  - Build Conf
  - Metro
  - Windows 8
  - WinRT
---
There’s been no shortage of news and non-news coming out of Build Conference this week, and I suspect there will be continue to be no shortage of the same for the next few months. We’ve learned that WPF isn’t quite dead, Silverlight isn’t quite dead, but both are in the process of being “reimagined,” whatever that means. Metro is clearly the first step of that process and it provokes a very diametric response from developers.<!--more-->

I sat down to breakfast this morning with a couple guys I hadn’t met. Since Build Conference was an incorporation of both ISVs and IHVs, I greeted them with, “Software or hardware?” The man across the table told me he was involved with software, while the guy to my right ignored me entirely. Without quite playing _20 Questions_, I found that the more vocal one was a client side developer in health care. He told me that he was very excited about the new Metro design and especially for the upcoming Windows 8 tablets. “Right now, everything is moving to Electronic Medical Records. So, when a doctor goes into see a patient, the first thing he does is turn his back on that patient and bring up the records on a desktop. A tablet is much more like the clipboards doctors used to use.” And, of course, Metro plays a huge role in the realization of that use case; a friendly, full-screen touch-accelerated app is intrinsic to the paradigm of a tablet. Not only that, but ease of access to peripherals, especially the webcam, make Windows Runtime (WinRT) a compelling API. As an added bonus, he pointed out that he could transition his current ASP.NET development stack and programmers much more easily to use either C# and XAML or even HTML5 and JavaScript to produce a Metro app than to use Coco and Objective-C to make an iPad app. Without a doubt, this is what Microsoft had envisioned.

At the opposite end of the spectrum, I met a couple developers that maintain an institutional client-facing application written in MFC. “There’s over three hundred screens in our application. No way can we fit that into Metro.” He went on further to explain that their application opens direct connections to a SQL Server instances located on both the client and server tiers. “I have to figure out how to go back and tell my boss that there’s just no way we can move over to Metro.”

I don’t know if he was aware or not of the seemingly obvious flaws in his app’s design, or that Microsoft was encouraging developers to forsake that all-too-familiar design of MFC apps that he showed me. “It’d be a complete rewrite;” that much we completely agreed upon.

There is a continuing consensus around the notion that WinRT is by far the most ambitious thing Microsoft has done in decades. Perhaps in a lot of ways, Windows XP is an apt analog of Win32 programming: it’s been around for so long, it’s in use in so many places, and it’s so well known, that it will take a very, very long time to unwind from it. Windows 8 and WinRT have enormous potential to revitalize the failing Windows desktop software market. All that remains to be seen is whether developers choose to capitalize upon it or cut their losses and move on.