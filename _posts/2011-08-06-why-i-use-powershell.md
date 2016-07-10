---
id: 457
title: Why I Use Powershell
date: 2011-08-06T23:36:11+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=457
permalink: /2011/why-i-use-powershell/
categories:
  - Coding
  - Lab49
tags:
  - 'C#'
  - powershell
---
I&#8217;ve been using Powershell for just over a year now, and its effect on my development workflow has been steadily increasing. Looking back, I have no doubt that it is the most important tool in my belt &#8211; to be perfectly honest, I&#8217;d rather have Powershell than Visual Studio now. Of course, that&#8217;s not to say Visual Studio isn&#8217;t useful &#8211; it is &#8211; but rather more that Poweshell fills an important role in the development process that isn&#8217;t even approached by other tools on the platform. Visual Studio may be the best IDE on the market, but at the end of the day, there are other tools that can replace it, albeit imperfectly.

To take some shavings off of the top top of the iceberg that is why I use Powershell, I&#8217;d like to share a recent experience of Powershell delivering for me. <!--more-->

## Setup for Failure

As a co-orgranizer for the [New York .NET Meetup](http://www.meetup.com/NY-Dotnet/) I was tasked with getting the list of attendees off of meetup.com and over to the lovely people at Microsoft who kindly give us space to meet. Now, you might think there&#8217;d be some nifty &#8220;export attendee list to CSV&#8221; function the meetup.com website, but like a lot of software, it doesn&#8217;t do half the things you think it should and this is one of those things. Usually my colleague David assembles the list, but this particular month he was out on vacation. He did, however, point me over to a GitHub repository of a tool that would do the extraction.

Following his advice, I grabbed the repository and brought up the C#/WinForms solution in Visual Studio. Looking at the project structure, I was a bit stunned at the scale of it all. The author had divided his concerns very well into UI, data access, the core infrastructure, and, of course, unit testing. I thought that was pretty peculiar considering all I wanted to do was get a CSV of names and such off of a website. Far be it for me to criticize another developer&#8217;s fastidiousness. Maybe it also launched space shuttles; you never know.

In what I can only now describe as naivety, I hit F5 and expected something to happen. 

I was rewarded with 76 errors.

Right off the bat, I realized that the author had botched a commit and forgot a bunch of libraries. I was able to find NHibernate fairly easily with Nuget, but had no luck with &#8220;Rhino.Commons.NHibernate&#8221;. I tried to remove the dependency problem, but didn&#8217;t have much luck. And the whole time I was wondering why the hell you needed all these libraries **to extract a damn CSV from the internet**.

## The Problem

Rather than throw more time after the problem, I decided to forge out on my own. Really, how hard could it be to

  1. Get an XML doc from the internet
  2. Extract the useful data
  3. Perform some heuristics on the names
  4. Dump a CSV file

Being a long-time C# programmer, my knee-jerk reaction was to build a solution in that technology. Forgoing a GUI to spend as little time as possible in building this, I&#8217;d probably build a single file design that ostensibly could consist of a single method. And if that were the case, why not script it?

So if I was going to write a script, what to use? I could write a JavaScript and run it on node.js, but it&#8217;s lacking proper CSV utilities and I&#8217;d have to run it on something other than my main Windows box. Not to mention I don&#8217;t particularly writing in JavaScript, so I&#8217;d probably write it in CoffeeScript and have to compile it, etc, etc. 

I briefly considered writing an F# script, but I suspect only about ten people would know what on earth it was, and, at the end of the day, I would like to share my script to others.

## The Solution

In the end, I concluded what I had known already: Powershell was the tool to use. It had excellent support for dealing with XML (via accelerators) and, as a real scripting language, had no pomp and circumstance.

Here&#8217;s the script I ended up writing:

<pre class="brush:powershell">function Get-MeetupRsvps([string]$eventId, [string]$apiKey) {
    $nameWord = "[\w-']{2,}"
    $regex = "^(?'first'$nameWord) ((\w\.?|($nameWord )+) )?(?'last'$nameWord)|(?'last'$nameWord), ?(?'first'$nameWord)( \w\.|( $nameWord)+)?$"

    function Get-AttendeeInfo {
        process {
            $matches = $null
            $answer = $_.answers.answers_item
        
            if(-not ($_.name -match $regex)) { $answer -match $regex | Out-Null }
            
            return New-Object PSObject -Property @{
                'FirstName' = $matches.first
                'LastName' = $matches.last
                'RSVPName' = $_.name
                'RSVPAnswer' = $answer
                'RSVPGuests' = $_.guests
            }
        }
    }

    $xml = [Xml](New-Object Net.WebClient).DownloadString("https://api.meetup.com/rsvps.xml?event_id=$eventId`&key=$apiKey")
    $xml.SelectNodes('/results/items/item[response="yes"]') `
    | Get-AttendeeInfo `
    | select FirstName, LastName, RSVPName, RSVPAnswer, RSVPGuests
}
</pre>

To dump this to a CSV file is then really easy:

<pre class="brush: powershell; gutter: false">Get-MeetupRsvps -EventId 1234 -ApiKey 'MyApiKey' | Export-Csv -Path rsvps.csv -NoTypeInformation
</pre>

And because of this design, it&#8217;s really extensible. Potentially, instead of exporting to a CSV, you could pipe the information into another processor that would remove anyone named &#8220;Thorsten.&#8221; Actually, that would look like this:

<pre class="brush: powershell; gutter: false">Get-MeetupRsvps -EventId 1234 -ApiKey 'MyApiKey' `
| ? { %_.FirstName -ne 'Thorsten' } `
| Export-Csv -Path rsvps.csv -NoTypeInformation
</pre>

It&#8217;d be pretty difficult to do that if I&#8217;d written a C# executable &#8211; you&#8217;d have to go into Excel to do that. Or write a Powershell script. Just saying.

Here&#8217;s the real kicker: I spent all of five minutes writing my Powershell script and then spent minutes tweaking my regex to identify as many names as possible. I didn&#8217;t need to recompile, just F5 again in Poweshell ISE, which you have installed already if you&#8217;re on Windows 7. Since I left the `Export-Csv` part off during debugging, I could just read the console output and see what I got.

When I was happy with my output, it was dead simple to distribute: throw it in [a GitHub Gist](https://gist.github.com/1091011) and move on with my life. If you decide to use it, all you need is Powershell installed (again, are you on Windows 7?) and the ability to copy and paste. No libraries. No worries. If you don&#8217;t like my regex, it couldn&#8217;t be easier to figure out how to replace it. If you want more fields, it&#8217;s easy to see where they should be added.

It&#8217;s really just that easy.