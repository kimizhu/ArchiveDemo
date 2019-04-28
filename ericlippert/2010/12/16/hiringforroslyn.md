# Hiring for Roslyn

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/16/2010 6:28:00 AM

-----

A couple years ago I made a blog posting called "The Managed Languages Team Is Hiring" mere hours before our senior management announced that our hiring goals had been met and told me to please stop recruiting people. [That was a little embarrassing](http://blogs.msdn.com/b/ericlippert/archive/2008/06/10/an-apology.aspx). This time I have been assured that, *really truly*, we do have open positions on my team.

 

This gives me the opportunity to more clearly articulate what has hitherto been speculation and rumour. Yes, as many of you have deduced we have split the managed languages team into two teams. One is concentrating on the "async" feature that is now in CTP as well as researching and developing other possible new language features. The second team, code-named "Roslyn", is working on longer-lead tasks, namely a massive rearchitecture of the C\# and VB compilers to support "compiler as a service" (CaaS) scenarios. The latter is the team I actually work on, along with a number of other developers and architects (like Neal Gafter, Matt Warren, Peter Golde and of course Anders Hejlsberg just to name a few.) We are looking less at specific language features and more at how to build a compiler architecture that is amenable to use as a foundation for modern tools.

 

When I joined the C\# team during the effort to build C\# 3 there were basically two "silos" of code on the C\# team: the compiler team and the IDE team had each written their own custom semantic analyzer. (They shared a lexer and parser.) We realized that the semantic analysis burden imposed by LINQ was large, and that doing the same work twice was not a good idea. We made a major investment in enabling the IDE to consume the compiler's semantic analyzer so that we would have only one lexer, parser and analyzer suitable for both "batch compile" and "interactive" scenarios. That was a big success.

 

However, the interface to the C\# 3 compiler's semantic analysis services is **exceedingly complex** and **narrowly tuned** to our specific IDE scenarios. And it is inconsistent with VB's similar system. And it is written in unmanaged C++, not in C\# or VB, making it difficult to use it from managed code. There is a huge opportunity for the tools space if we can make a semantic analysis service that more than just our own IDE team can consume, with a consistent interface across the two premier .NET languages that is itself easily accessible from managed languages.

 

This is a enormous undertaking given the small size of our team; we have been working on it for some time now and **we have plenty yet to do**. The Roslyn team is somewhat understaffed right now; we are looking to hire someone at the "Developer 2" level. We will *consider* "fresh out of college" candidates but the ideal candidate we're looking for would have solid industry experience with both use of and creation of code analysis tools. We need someone who is crazy in love with compilers, who has thorough knowledge of C\# and/or VB, and who wants to work with the awesome squad of senior architects and engineers we've already assembled.

 

The text of the job posting is below; if you are interested in this position please **do not** send your resume to me directly or post it here as a comment; [use the mechanisms on the career site](https://careers.microsoft.com/JobDetails.aspx?ss=&pg=0&so=&rw=1&jid=28776&jlang=EN) to make sure that your information gets entered into our database. If you want to mention in your application that you learned about this job here, that would be great, but not necessary.

 

Finally, why "Roslyn"? I was the one who came up with this silly name. Almost all Microsoft product code names are now city or town names. It's unlikely that the lovely and historic town of [Roslyn, Washington](http://en.wikipedia.org/wiki/Roslyn,_Washington) is going to object to us using their name. But if we chose the name of a copyrighted character or fictional location from a book or movie then there might be a legal issue there, and obviously names already taken by other products are right out.

 

But more importantly **my office has a [northern exposure](http://en.wikipedia.org/wiki/Northern_Exposure#Production)**.

 

Ha ha ha ha ha ha\!\!\! I crack myself up.

 

-----

> Job Category: Software Engineering: Development  
> Location: United States, WA, Redmond  
> Job ID: [734698 28776](https://careers.microsoft.com/JobDetails.aspx?ss=&pg=0&so=&rw=1&jid=28776&jlang=EN)  
> Division: Server & Tools Business C\# and VB are the flagship languages targeting the .NET platform. The overwhelming majority of code running on .NET is written in VB or C\# using Visual Studio. The C\# and VB team build both the language compilers and the Visual Studio editing experience around those languages, and the compiler team is looking for a new team member to help us deliver on a bold, new undertaking.
> 
> Until now, the VB and C\# compilers have been used as black boxes. You put text in, and you get out a binary file. In our long-lead project, code name Roslyn, we’re changing that dynamic by building an API that exposes compilers’ analysis engines. Exposing parse trees and types, expression binding, and assembly production through an API enables a world of new scenarios including REPL, C\# and VB as scripting languages, and more. In order to do this right, we’re re-examining the compilation pipeline at the most fundamental level and designing an immutable model written top to bottom in managed code.
> 
> Technical challenges on the Roslyn compiler team come in many forms. Ever noticed how fast the C\# compiler is? It’s not an accident. What about lambda functions? How do those things work under the covers? If you were on the Roslyn compiler team, you’d be developing high performance implementations of C\# and VB language features that developers use every day. How does Visual Studio produce such accurate and quick IntelliSense? It’s the compiler guts back there, and you could help build the foundation that enables scenarios like that.
> 
> As a developer on the Roslyn compiler team, you’ll work in an environment filled with smart, industry-recognized folks working on a next-generation product that directly affects the experience of thousands of developers like yourself every day. And if you’re seeking a more direct interaction with customers, on the Roslyn compiler team you will develop an in-demand expertise in the compiler implementation that allows you to speak as a domain expert in online forums and at conferences.
> 
> We’re looking for a developer who wants to be challenged, has sharp intellect, and possesses solid collaboration skills. The ideal candidate will have a strong architectural sense, a history of producing robust and maintainable solutions, and a passion for delivering genuine customer value. Knowledge of the .NET managed execution environment is a big plus and the ability to drive open issues to resolution is a must. We program in VB and C\#.

