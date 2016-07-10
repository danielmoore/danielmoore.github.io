---
id: 122
title: The Qualifications for Writing Textbooks
date: 2009-04-07T13:48:37+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=122
permalink: /2009/the-qualifications-for-writing-textbooks/
categories:
  - Coding
  - Education
tags:
  - Computer Science
  - Databases
  - Textbooks
---
It&#8217;s certainly true that there are many [unqualified authors](http://video.google.com/videoplay?docid=-4408250627226363306) that the self-taught programmer has to look out for, but what about the textbooks we read at universities? I won&#8217;t argue for a second that tenured professors are the least bit unqualified to discuss the intricacies of algorithms and abstract data structures, but who writes the book on using real world implementations? Who is qualified to write the book on Java or SQL Server? I&#8217;d want to see someone who has been using the technology for several years or a good portion of the technology&#8217;s life if it&#8217;s relatively new. Unfortunately, I find this is rarely the case.<!--more--> I think publishers could run a pitch like this:

> From the people who confused you with the most elementary algorithms comes a series of things you really need to know to do your job! Watch, mystified, as the book which you have a test on in eight hours _invents_ symbols to explain the details of single digit addition! You might have been top of your class in high school and showed promise through your core and basic classes, but just you wait! Our authors have learned and mastered techniques to simultaneously insult your intelligence and dumbfound you with a needless amount of abstraction and complexity! All this can be yours for the low-low price of $280 for our 120 page full color, hardcover book!

<span>I&#8217;ve always been upset with textbook publishers, ever since my days in Computer Science 1, sophomore year in high school. The authors of that poorly constructed book (the binding literally only lasted about six months before self-destructing) took some kind of sadistic enjoyment in coming up with unintelligible vocabulary, patterns, and practices in which to subject students. In those days our teacher was our savior; no matter how mind-gratingly stupid and dull the book was, Mr. Patterson invariably sorted things out and never hesitated to say the book was moronic. I have to thank him for saving my sanity or, at least, staving off the inevitable future.</span>

This semester, I draw particular complaint with [the book in my databases class](http://www.amazon.com/Fundamentals-Database-Systems-Ramez-Elmasri/dp/0321369572/ref=pd_bbs_sr_1?ie=UTF8&s=books&qid=1238532060&sr=8-1), inappropriately named &#8220;Fundamentals of Database Systems (5e),&#8221;  by <span>Ramez Elmasri and Shamkant B. Navathe. I&#8217;ve been writing SQL for about four years now, but it wasn&#8217;t untill my last job that I think I really learned the full potential of a database system. My old boss was an absolute <em>god </em>in writing SQL and I  made certain that the learning experience more than accounted for the pay. It was under his tutelage that I learned how well-designed database systems were made and how to retrieve amazing data no one thought possible. I wouldn&#8217;t consider myself a sage in designing databases, but I think that I&#8217;m definitely qualified to pass judgment on most designs, especially the ones that would be considered &#8220;fundamental.&#8221;</span>

<span>During the early stages of my class, my professor discussed the all-important topic of table keys. (For my non-programmer friends, you use database keys to look up data in a particular table. Since some tables can have millions of rows, you have to choose a key that is unique and is as small as possible for quick lookup.) It&#8217;s a well-held practice in modern databases that the database is really the best agent to tell you what the keys are, so in general, what you want to do is tell it what your data is, and, like a coat-check, it gives you a little token (a 32-bit signed integer) to get your data again. Because the database chooses the key, you can be assured that it is both unique and small enough to quickly look things up. This is known as an auto-increment integer primary key column, mainly because the integer that you&#8217;re given is 1 (or some increment) more than the key that was given out last. Even though it&#8217;s not a traditional mechanism in database research, every major database (MSSQL, Oracle, MySQL, Protegé) has some incarnation of it. Traditionally, the programmer chooses one or more fields that combined are unique. While this is effective, it takes significantly longer for the server to prepare the key to look up the data and the number of fields that must be duplicated on tables so the original data can be found becomes tremendous. My database professor still disagrees with me, but I have faith that he&#8217;ll have an epiphiny sometime and understand and regret the error of his ways.</span>

<span>So why is it that these authors, from whom my professor quotes, have got it all wrong? In lieu of real research, I did some investigative Googling to find the qualifications of these men.</span>

### <span>Ramez Elmarsi</span>

  * B.S., Electrical Engineering, Alexandria University (Egypt, 1972)
  * M.S., Ph.D., Computer Science, Stanford University (1980)

Elmarsi has spent the bulk of his life in research while consulting for Honeywell developing a distributed database testbed and associated tools from 1982 to 1987. He now teaches at the University of Texas at Arlington and consults for various law firms.

###### Source: <http://ranger.uta.edu/~elmasri/>

### Shamkant B. Navathe

  * Ph.D., Industrial and Operations Engineering, University of Michigan (1976)
  * M.S., Computer and Information Science, Ohio State University (1970)
  * B.E., Electrical Communications Engineering, Indian Institute of Science (1968)
  * B.Sc., Physics, Mathematics, University of Poona (India, 1965)

Navathe was a Systems Engineer for IBM from 1968-1969 before working for Electronic Data Systems in 1971. Starting in 1975, Navathe began teaching at NYU as an associate professor and then moved to the University of Florida for a few years until he ultimately settled at the Georgia Institute of Technology, where he teaches now.

###### Source: <http://www.cc.gatech.edu/computing/Database/faculty/sham/>

There&#8217;s no doubt that these men have the academic prowess demanded by their degrees and both have an extensive list of published works. But what really disturbs me is their professional résumé. Navathe stopped working professionally in 1971 and Elmarsi has never worked as a full-time employee at _any_ company. This is when I understood exactly why my book is heavy on unintuitive mathematical symbols wielded like a bat with barbed wire and light on intelligence or edifying information. This field changes dramatically every couple years and unless you can keep up to date on the very latest technology, you are willing yourself into obsolescence. But yet some <span style="text-decoration: line-through;">charlatans</span> authors have the audacity to publish texts advising the unsuspecting student to continue using the old technologies of yore, rather than using what is new and efficient. Perhaps even more inexcusable is the book selection committee at my university choosing textbooks no one can read, let alone understand to remind all of us undergrad students that we&#8217;re not Ph.D.&#8217;s and we haven&#8217;t joined their elite clique.

I think this problem is far more widespread than I have broached here. A year or so ago, I wrote a letter I never delivered to my dean, more or less as a venting exercise. I think I&#8217;m going to clean it up and discuss my points here, in the event someone who can do something stumbles upon my humble blog.