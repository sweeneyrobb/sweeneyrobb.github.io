---
title: A C# developer's take on F# - Discovery to Production
layout: post
description: 
categories: [blog]
tags: [f#, powershell]
comments: true
modified: 2014-10-07T12:15:25-08:00
---

I had done some discovery around F# back when it first came out. At that point it didn't seem like a viable option for end to end application development. I knew then, that F# was very powerful language but didn't quite seem ready for primetime. Today however, is a different story.

I recently, I grew excited of F# after I read [Why F#](http://bit.ly/NfwLHK) written by Scott Wlaschin ([@ScottWlaschin](https://twitter.com/ScottWlaschin)). Looking at these points made me very interested in investigating further. Here Wlaschin says I can be more concise, write fewer lines of code and have fun. How could I not investigate further.

One of my first questions was; can I do all of the things I could in C#? Scott Wlaschin runs another site [F# for fun and profit](http://fsharpforfunandprofit.com/) in which he posts another great article [Anything C# can do…](http://fsharpforfunandprofit.com/posts/completeness-anything-csharp-can-do/) Okay great! Now can I interop with .NET seamlessly? He's got [another article](http://fsharpforfunandprofit.com/posts/completeness-seamless-dotnet-interop/) addressing this too. Well cheers to Scott Wlaschin for putting this information together. Because, now he's piqued my interest.

I spent the next 3 days re-acclimating myself to the F# language. All the while thinking about how to apply F# to my day to day development needs. The following is a disjointed explanation of my current process and how I believe F# really hits home for me.

Lazy bones
----------
I'm lazy. I've been a developer for the past 15 years. I've grown tired of typing code for the sake of writing code ([Ain't Nobody Got Time for That](http://www.youtube.com/watch?v=zGxwbhkDjZM)); record types (POCO), boilerplate, intermediate types, overloads, etc. I've taken advantage of any means possible for not having to write “boring” code (if you haven't done this already the `Edit | Paste Special | Paste JSON/XML As Classes`	 menu in Visual Studio 2012/2013 is an amazing time saver). Writing all this work just to finally get to writing the “cool stuff”.

Some of the things I have done recently to avoid writing a long C# code-base to discover an approach, is to start with a [REPL approach](http://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) using PowerShell. This gives me the ability to get a lot done very quickly by taking advantage of the pipeline, dynamic typing and large set of commandlets available. Once I have an algorithm I'm happy with I'll port the PowerShell to C#. The languages are similar enough that this process doesn't take too long. If I'm lucky I have a junior dev that can crank it out for me :)

This is my process for the most part, better or worse. In this I realize that most of my discovery code still has to be transposed as I would rarely use PowerShell to build an production application. PowerShell is very good but still dynamically typed making it somewhat difficult test and it's typing system is rather limited. No gripes here, PowerShell wasn't designed for a hardcore developer, it just happens to make for a nifty tool to get things done.

Typing is important (to me)!
----------------------------
I know that I've made a large focus (and will continue to) on typing. It is a personal opinion of mine, that dynamically typed languages feel unsafe, when used without tests. Writing tests to address the lack of a static type system, isn't something I really want to do. I understand this is a very sensitive topic to some; I don't want to get into a battle; this is just my personal opinion.

Goodbye PowerShell, Hello F#!
-----------------------------
I see now that F# can replace PowerShell in my current process of using F# for discovery. Two of the main reasons I use PowerShell is:

* It provides me real-time feedback
* The pipeline to move my data from one function to another without a bunch of formality

Interpreted
-----------
I'm able to REPL my code and get real-time feedback on the nature of my application. This type of exploration helps me understand what my application is doing, during the writing. I'm able to make adjustments without having to refactor and recompile. I can do this directly in Visual Studio with the `<Alt> + ;` or `<Alt> + <Enter>` key bindings. I can also run F# interactive (fsi.exe) and evaluate using typing `;;` into the editor.

Pipeline
--------
I love the pipeline in PowerShell and the pipe forward operator `|>` is pretty much the same thing. This is a way to take data and apply the data to a function. This helps get rid of formally naming data that is only used to pass to another function.

C# Extension Methods have helped, a large amount, allowing the chaining methods together. This begins to fall apart when returning different data types. Some creative brackets, nesting and lamba expressions later, you might achieve what comes very gracefully in F#.

Two birds, one stone
--------------------
Using F# I'm also able to take advantage of some other cool and powerful features of the F# language:

Immutable (by default)
----------------------
Most variables are immutable by default. This means we cannot change it's value once it is set. This may seem odd coming from an impetrative world but when you adjust it actually reduces the amount of errors that mutable code brings (changing state and/or purpose). Because variables must be initialized with a value; that there are fewer null checks. There may be times where you want to mutate a variable. You must use the mutable keyword to allow variables to change and the mutate operator `<-` to change values. Having a separate operator for this is terrific, as becomes quickly recognizable in code, what is mutable and what is not.
Static Typing / Native Types

Won't go into this much, but, satisfying the compiler, a large percentage means you are in fairly good shape. As an example consider a scenario, where we are updating a discriminating union and not updating Active Pattern matching statements to complete the new scenarios. My example here is a discriminating union that supports only Read, Update, Delete.

	type command =	| Read
			| Update
			| Delete
	
	let executeCommand command = 
		match command with
		| Update -> printfn "updating"
		| Delete -> printfn "deleting"
		| Read -> printfn "reading"
	
	executeCommand Update

Now say that we want to extend our command type to include “Create”

	type command = | Create
				   | Read
				   | Update
				   | Delete

The compiler complain, saying that the pattern match expression is incomplete. Now, imagine some large C# codebase and trying to find all the dependent statements with one small update. Hopefully, you have tests to back everything and that everything is wrapped and handled in some common library. Making the update less painful, but, this requires composition and disciple. We get this for free with F# AND we are avoiding nulls, without a single null check. YAY!

Production Ready (no more transposing to C#)
--------------------------------------------
F# being able to do all things necessary for both a PowerShell and C# replacement, there is little reason to transpose F# to C#. Rather, I can now build discovery code that I can use for production. This realization is hard for me to believe.

Conclusion
----------
F# seems to hit a wonderful sweet spot for me. Bridging the gaps between my discovery and delivery processes. The language has been thought through, all the way down to the native types (tuples are amazing). With a growing community and large amount of work to get F# a first class .NET language—I believe I will be making some changes. Maybe not 100% but F# will most definitely be my first stop.

Resources
---------
* [http://tryfsharp.org](http://tryfsharp.org) – Very good in-browser F# interactive and basic tutorial
* [http://fsharpforfunandprofit.com](http://fsharpforfunandprofit.com) – Very good tutorials / guides to common questions regarding F#
* [http://fsharp.org](http://fsharp.org) – Great community surrounding anything F#
* [http://channel9.msdn.com/Tags/fsharp](http://channel9.msdn.com/Tags/fsharp) – Channel 9 video's on F#
* [http://sergeytihon.wordpress.com](http://sergeytihon.wordpress.com) – Sergy Tihon ([@sergey_tihon](http://twitter.com/sergey_tihon)) puts out a terrific weekly F# round-up.
