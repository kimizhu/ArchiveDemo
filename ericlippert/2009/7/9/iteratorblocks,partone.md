# Iterator Blocks, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/9/2009 10:11:00 AM

-----

There is a constant tension in language design between solving general problems and solving specific problems; finding the right spot on the general-to-specific spectrum can be quite tricky.

The design of iterator blocks yields (ha ha) a germane example. At almost every step along the way, there are opportunities for making choices that determine whether a more general or a more specific problem is being solved.

Let’s start with the high-level design of the feature. Iterator blocks are, as the name implies, all about making it easier to write code that **iterates over some collection of items** in a natural way. This is a pretty darn general scenario; there are lots of potential collections (stacks, queues, lists, dozens of different kinds of trees…) containing arbitrarily many different types of items, and lots of ways to iterate over them (in order, post order, pre order, filtered, projected, grouped, sorted…)

We *could* have made it much more general. Our iterator blocks can be seen as a weak kind of [coroutine](http://en.wikipedia.org/wiki/Coroutine). We could have chosen to implement full coroutines and just made iterator blocks a special case of coroutines. And of course, coroutines are in turn less general than first-class [continuations](http://blogs.msdn.com/ericlippert/archive/tags/Continuation+Passing+Style/default.aspx); we could have implemented continuations, implemented coroutines in terms of continuations, and iterators in terms of coroutines.

But we didn’t. We decided that the sweet spot for the high-level scenario-driven design of the feature was iterators over collections, and so we concentrated on that. Some adventurous people use the fact that iterators are “poor-man’s coroutines” as a shortcut to building coroutine-like systems that have only a tenuous connection to the semantics of iterating over the items in a collection, but those are the exception, not the rule. Their scenarios certainly did not drive the design of the feature.

We want to balance the generality of the feature against the high cost of that generality. Premature optimization is often cited as the root of **all** evil, but I don’t think that’s quite true. Premature generality is responsible for a lot of evil too\! As we’ll see, a lot of the time when faced with a design decision we take the [YAGNI](http://en.wikipedia.org/wiki/YAGNI) position; we choose to implement a little bit less generality to get a large cost savings, with the assumption that hardly anyone would benefit from that spending.

Over our next few fabulous adventures in coding I’ll discuss some of the reasons for the seemingly odd restrictions on the generality of iterator blocks – things like, why can there be no unsafe code in iterator blocks? Why can an iterator block not take a ref or out parameter? Why can’t you put a yield in a finally? And so on.

