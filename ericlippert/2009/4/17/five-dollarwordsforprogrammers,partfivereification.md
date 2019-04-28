# Five-Dollar Words For Programmers, Part Five: Reification

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/17/2009 10:36:00 AM

-----

Today, another in [my series on awesomely arcane words for programmers](http://blogs.msdn.com/ericlippert/archive/tags/Big+Words/default.aspx). **Reification** is the process of turning something that is normally thought of as an **abstract concept** into something more **concrete**. It’s from the Latin *res facere*, “thing making”.

[![Crown](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/FiveDollarWordsForProgrammersPartFiveRei_8482/Crown_3.jpg "Crown")](http://en.wikipedia.org/wiki/File:Str%C3%B6hl-Regentenkronen-Fig._11.png "The Imperial State Crown")Thing-making happens all the time in non-computer-related domains. Queen Elizabeth, her throne and her crown, for example, are all concrete representations of the abstract concept of “sovereignty”. A judge’s gavel or a statue of a blind woman holding scales make a concrete representation of the abstract concept of “justice”.

You also see reification in [teleological](http://en.wikipedia.org/wiki/Teleology) rhetoric. “[Information wants to be free](http://en.wikipedia.org/wiki/Information_wants_to_be_free)”, for example, [fallaciously](http://en.wikipedia.org/wiki/Reification_\(fallacy\)) reifies “information” for rhetorical impact. And it succeeds in doing so - obviously this is some compelling rhetoric that speaks to lots of people. It makes an interesting point in a memorable way.(\*) But hopefully no one seriously believes that the abstract concept of “information” is actually *a thing with desires or beliefs*. Only concrete entities can actually want something.

In the world of programming language design and implementation, [reification](http://en.wikipedia.org/wiki/Reification_\(computer_science\)) is a huge part of what we do; designing languages and building compilers is all about taking extremely abstract concepts like “variable”, “procedure”, “type”, “set”, “class”, and so on, and making a device that reifies those concepts into more concrete textual entities that can be declared and manipulated. Our goal for LINQ in C\# 3.0 was to reify ideas like “filter”, “projection”, “group”, “order”, and “query” into concrete entities that could be directly expressed in the text of the program. Adding [homoiconicity](http://blogs.msdn.com/ericlippert/archive/2009/03/23/five-dollar-words-for-programmers-part-three-homoiconic.aspx) via expression trees reifies those concepts even further, making them manipulable as runtime objects.

Some of the most interesting areas in the computing industry are those where reifications are still inchoate or contradictory. Consider the tarball that is modern bleeding-edge asynchronous programming, for example. Pretty much everyone agrees that we need some decent way to solidly reify the notion of “asynchronous work”, but how? Looking around, I find that people reify this idea (or closely related ideas) in a dozen different ways. Workflows, coroutines, observable collections, futures and promises come immediately to mind; I’m sure there are many more. This lack of consensus makes the language designers’ jobs much harder.

\*\*\*\*\*\*\*\*\*\*\*\*\*

(\*) The alert reader will have just wryly noted that I have *indulged in the very rhetorical fallacy that I’m pointing out*. I’ve fallaciously reified a *quotation* into a thing that can *succeed*, can *speak* and can *make a point*. Really I should be making these attributions to the concrete *people* who speak that statement, not to the abstract *statement* itself.

And isn’t that ironic, don’t you think? Who would’ve thought? It figures.

