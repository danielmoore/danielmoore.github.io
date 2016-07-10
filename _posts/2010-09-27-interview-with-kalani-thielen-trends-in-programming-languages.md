---
id: 358
title: 'Interview with Kalani Thielen: Trends in Programming Languages'
date: 2010-09-27T09:30:45+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=358
permalink: /2010/interview-with-kalani-thielen-trends-in-programming-languages/
categories:
  - Coding
  - Lab49
tags:
  - 'C#'
  - Cayenne
  - Clojure
  - DSLs
  - Epigram
  - 'F#'
  - Haskell
  - Kalani Thielen
  - Lab49
  - Lisp
  - OCaml
  - Python
  - Ruby
  - Scala
  - Scheme
---
Last week I interviewed colleague [Kalani Thielen](http://blog.lab49.com/archives/author/kthielen), Lab49&#8217;s resident expert on programming language theory. We discussed some of the new languages we&#8217;ve seen this decade, the recent functional additions to imperative languages, and the role DSLs will play in the future. Read on for the full interview.<!--more-->

**DM** F#, Clojure, and Scala are all fairly new and popular languages this decade, the former two with striking resemblance to OCaml and Lisp, respectively, and the lattermost being more original in syntax. In what way do these languages represent forward thinking in language design, or how do they fail to build upon lessons learned in more venerable languages like Haskell?

**KT** It’s a common story in the world of software that time brings increasingly precise and accurate programs.  Anybody who grew up playing games on an Atari 2600 has witnessed this firsthand.  The image of a character has changed from one block, to five blocks, to fifty blocks, to (eventually) thousands of polygons.  By this modern analog of the Greeks’ method of exhaustion, Mario’s facial structure has become increasingly clear.  This inevitable progression, squeezed out between the increasing sophistication of programmers and the decreasing punishment of Moore’s Law, has primarily operated at three levels: the images produced by game programs, the logic of the game programs themselves, and finally the programming languages with which programs are produced.  It’s this last group that deserves special attention today.

The lambda calculus (and its myriad derivatives) exemplifies this progression at the level of programming languages.  In the broadest terms, you have the untyped lambda calculus at the least-defined end (which closely fits languages like Lisp, Scheme, Clojure, Ruby and Python), and the calculus of constructions at the most-defined end (which closely fits languages like Cayenne and Epigram).  With the least imposed structure, you can’t solve the halting problem, and with the most imposed structure you (trivially) can solve it.  With the language of the untyped lambda calculus, you get a blocky, imprecise image of what your program does, and in the calculus of constructions you get a crisp, precise image.

Languages like F#, Scala and Haskell each fall somewhere in between these two extremes.  None of them are precise enough to accept only halting programs (although Haskell is inching more and more toward a dependently-typed language every year).  Yet all of them are more precise than (say) Scheme, where fallible human convention alone determines whether or not the “+” function returns an integer or a chicken.  But even between these languages there is a gulf of expressiveness.  Where a language like C is monomorphically-typed (and an “int\_list” must remain ever distinguished from a “char\_list”), F# introduces principled polymorphic types (where, pun intended, you can have “’a list” for any type ‘a).  Beyond that, Haskell (and Scala) offer bounded polymorphism so that constraints can be imposed (or inferred) on any polymorphic type – you can correctly identify the “==” function as having type “Eq a => a -> a -> Bool” so that equality can be determined only on those types which have an equivalence relation, whereas F# has to make do with the imprecise claim that its “=” function can compare any types.

No modern programming language is perfect, but the problems that we face today and the history of our industry points the way toward ever more precise languages.  Logic, unreasonably effective in the field of computer science, has already set the ideal that we’re working toward.  Although today you might hear talk that referential transparency in functional languages makes a parallel map safe, tomorrow it will be that a type-level proof of associativity makes a parallel _reduce_ safe.  Until then, it’s worth learning Haskell (or Scala if you must), where you can count the metaphorical fingers on Mario’s hands, though not yet the hairs of his moustache.

**DM** One might assume that increased &#8220;precision&#8221; in a language would come at the cost of increased complexity in concepts and/or syntax. In a research scenario, the ability to solve the halting problem certainly has its merits, but is that useful for modern commercial application development, and, if so, does it justify the steeper learning curve?

**KT** It&#8217;s absolutely true that languages that can express concepts more precisely also impose a burden on programmers, and that a major part of work in language design goes into making that burden as light as possible (hence type-inference, auto roll/unroll for recursive types, pack/unpack for existential types, etc).

However &#8212; to your point about the value of that increased precision – I would argue that it&#8217;s even _more_ important in commercial application development than in academia.  For example, say you&#8217;ve just rolled out a new pricing server for a major client.  Does it halt?  There&#8217;s a lot more riding on that answer than you&#8217;re likely to find in academia.  And really, whether or not it halts is just one of the simplest questions you can ask.  What are its time/space characteristics?  Can it safely be run in parallel?  Is it monotonic?  In our business, these questions translate into dollars and reputation.  Frankly I think it&#8217;s amazing that we&#8217;ve managed to go on this long without formal verification.

**DM** Most dynamic languages have some air of functional programming to them, while still being fundamentally imperative. Even more rigorous imperative languages like C# are picking up on a more functional style of programming. The two languages you mentioned earlier, Cayenne and Epigram, are both functional languages. Are we moving toward a pure functional paradigm, or will there continue to be a need for imperative/functional hybrids?

**KT** What is a &#8220;functional language&#8221;?  If it&#8217;s a language with first-class functions, then C is a functional language.  If it&#8217;s a language that disallows hidden side-effects, then Haskell isn&#8217;t a functional language.  I think that, at least as far as discussing the design and development of programming languages is concerned, it&#8217;s well worth getting past that sort of &#8220;sales pitch&#8221;.

I believe that we&#8217;re moving toward programming languages that allow programmers to be increasingly precise about what their programs are supposed to do, up to the level of detail necessary.  The great thing about referential transparency is that it makes it very easy to reason about what a function does &#8212; it&#8217;s almost as easy as high school algebra.  However, if you&#8217;re not engaged in computational geometry, but rather need to transfer some files via FTP, there&#8217;s just no way around it.  You&#8217;ve got to do this, then this, then this, and you&#8217;ll need to switch to something more complicated, like Hoare logic, to reason about what your program is doing.  Even there, you have plenty of opportunity for precision &#8212; you expect to use hidden side-effects but only of a certain type (transfer files but don&#8217;t launch missiles).

But maybe the most important tool in programming is logic itself.  It&#8217;s a fundamental fact – well known in some circles as the &#8220;Curry-Howard isomorphism&#8221; – that programs and proofs are equivalent in a very subtle and profound way.  This fact has produced great wealth for CS researchers, who can take the results painstakingly derived by logicians 100 years ago and (almost mechanically) publish an outline of their computational analog.  Yet, although this amazing synthesis has taken place rivaling the unification of electricity and magnetism, most programmers in industry are scarcely aware of it.  It&#8217;s going to take some time.

I think there&#8217;s a good answer to your question in that correspondence between logic and programming.  The function, or &#8220;implication connective&#8221; (aka &#8220;->&#8221;), is an important tool and ought to feature in any modern language.  As well, there are other logical connectives that should be examined.  For example, conjunction is common (manifested as pair, tuple, or record types in a programming language), but disjunction (corresponding to variant types) is less common though no less important.  Negation (corresponding to continuations consuming the negated type), predicated quantified types, and so on.  The tools for building better software are there, but we need to work at recognizing them and putting them to good use.

Anybody interested in understanding these logical tools better should pick up a copy of Benjamin Pierce&#8217;s book _Types and Programming Languages_.

**DM** Many frameworks have a one or more DSLs to express things as mundane as configuration to tasks as complicated as UI layout, in an effort to be more express specific kinds of ideas more concisely. Do you see this as an expanding part of the strategy for language designers to increase expressiveness in code? Is it possible that what we consider &#8220;general purpose languages&#8221; today will become more focused on marshalling data from one DSL to another, or will DSLs continue to remain a more niche tool?

**KT** I guess it depends on what you mean by &#8220;DSL&#8221;.  Like you say, some people just have in mind some convenient shorthand serialization for data structures (I&#8217;ve heard some people refer to a text format for orders as a DSL, for example).  I&#8217;m sure that will always be around, and there&#8217;s nothing really profound about it.

On the other hand, by &#8220;DSL&#8221; you could mean some sub-Turing language with non-trivial semantics.  For example, context-free grammars or makefiles.  Modern programming languages, like Haskell, are often used to embed these &#8220;sub-languages&#8221; as combinator libraries (the &#8220;Composing Contracts&#8221; paper by Simon Peyton-Jones et al is a good example of this).  I think it&#8217;s likely that these will continue as valuable niche tools.  If you take monadic parser combinators for example, it&#8217;s very attractive the way that they fit together within the normal semantics of Haskell, however you&#8217;ve got to go through some severe mental gymnastics to determine for certain what the space/time characteristics of a given parser will be.  Contrast that with good old LALR(1) parsers, where if the LR table can be derived you know for certain what the space/time characteristics of your parser will be.

On the third hand, if a &#8220;DSL&#8221; is a data structure with semantics of any kind, your description of a future where programs are written as transformations between DSLs could reasonably describe the way that compilers are written today.  Generally a compiler is just a sequence of semantics-preserving transformations between data structures (up to assembly statements).  I happen to think that&#8217;s a great way to write software, so I hope it&#8217;s the case that it will become ever more successful.