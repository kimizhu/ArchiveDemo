# Iterators at the Summer Games

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/26/2009 10:09:00 AM

-----

[Ed "Scripting Guy" Wilson](http://blogs.technet.com/heyscriptingguy/) was kind enough to ask me to be a [guest commentator](http://blogs.technet.com/heyscriptingguy/archive/2009/06/25/hey-scripting-guy-event-10-solutions-from-expert-commentators-beginner-and-advanced-the-1-500-meter-race.aspx) at this years [Summer Scripting Games](http://blogs.technet.com/heyscriptingguy/archive/tags/2009+Summer+Scripting+Games/default.aspx), which have just completed.

I've been working on a series for this blog about some unusual cases in the design of the "iterator block" feature in C\# 2.0; this bit from my commentary is going to be germane to that series. I thought I'd post it here as an appetizer before we get into the main courses of the series. The series on iterators will probably run throughout July.

The problem for event ten of the scripting games was to write a script which changes the priority of every new process that has a particular name.

\*\*\*\*\*\*\*\*\*\*\*

There’s an odd thing that you learn when working on developer tools: The people who *design* and *build* the tools are often not the experts on the actual real-world *use* of those tools. I could tell you anything you want to know about the VBScript parser or the code generator or the runtime library, but I’m no expert on writing actual scripts that solve real problems. This is why I was both intrigued and a bit worried when the Scripting Guys approached me and asked if I’d like to be a guest commentator for the 2009 Summer Scripting Games.

I wrote this script the same way most scripters approach a technical problem that they don’t immediately know how to solve; I searched the Internet for keywords from the problem domain to see what I could come up with. Of course, I already knew about our MSDN documentation, I had a (very) basic understanding of WMI, and I knew that the Scripting Guys had a massive repository of handy scripts.

My initial naïve thought was that I would have to go with a polling solution; sit there in a loop, querying the process table every couple of seconds, waiting for new processes to pop up. Fortunately, my Internet searches quickly led me to discover that process startup *events* can be treated as an endless *collection* of objects returned by a WMI query.

That got me thinking about **the powerful isomorphism between *events* and *collections*.**

A collection typically uses a “pull” model — the consumer asks for each item in the collection one at a time as needed, and the call returns when the item is available. Events typically work on a “push” model — the consumer registers a method that gets called every time the event fires. But not necessarily; the WMI provider implements events on a “pull” model. The event is realized as a collection of “event objects.” It can be queried like any other collection. Asking for the next event object that matches the query simply blocks until it is available.

Similarly, **collection iterators could be implemented on a “push” model**. They could call a method whenever the next item in the collection becomes available. The next version of the CLR framework is likely to have standard interfaces that represent “observable collections”, that is, collections that “push” data to you, like events do. The ability to treat events as collections and collections as events can lead to some interesting and powerful coding patterns.

\*\*\*\*\*\*\*\*\*\*\*

The rest of the solution and analysis is [here](http://blogs.technet.com/heyscriptingguy/archive/2009/06/25/hey-scripting-guy-event-10-solutions-from-expert-commentators-beginner-and-advanced-the-1-500-meter-race.aspx).

Have a good weekend\!

