# Ambiguous Optional Parentheses, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/20/2010 7:13:00 AM

-----

(﻿This is part one of a three-part series on C\# language design issues involving elided parentheses in constructor calls. Part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/09/23/ambiguous-optional-parentheses-part-two.aspx).)

Another amusing question from [StackOverflow](http://stackoverflow.com/questions/3661025/), which I present here as a dialogue, as is my wont:

> In C\# 3.0 the object initializer (and collection initializer) syntax permits both new Foo { Bar = 123 } and new Foo() { Bar = 123 }. If the default constructor is used then the argument list is optional. Why did the design committee decide to make the argument list optional?

That decision was made in 2004, before I started on the C\# team, so any reasoning about it will be somewhat conjectural. In browsing the archive of design notes there is no place where this decision is specifically documented with pros and cons, but I did discover a few things in looking at the notes.

First, even back when the idea was first raised, the sample code fragments in the notes for possible syntaxes do not include empty parameter lists. It seems that people thought this was the natural way to go from the beginning.

Second, I was also interested to discover that the design team was not entirely happy at first with the proposed syntax (of following the constructor call with a parenthesized list of side effects) but several efforts to find a better syntax that did not have ambiguities with existing programs failed to come up with anything better.

And third, it seems pretty natural to omit the arguments when initializing an object or collection because that's how arrays do it too: new int\[\] {1, 2, 3} and new List\<int\> {1, 2, 3} are obviously analogues.

> OK, then, more generally: how do you decide when it is acceptable to make this sort of minor syntactic sugar?

The first question must be "is there a good reason why the argument list is required?" We cannot make it *optional* if it is *required*, obviously. In this case there is no compelling reason why it should be required. An empty argument list is redundant and adds no value, so if it can be elided, that's sort of nice. Furthermore, the feature hits a bit of a "sweet spot"; the most likely scenario for using an object or collection initializer is precisely when a mutable object has properties that are not set by any constructor, and such objects often have a default constructor. It seems reasonable that most of the time the argument list is going to be empty, so again, this feature is nice.

However, "sort of nice" only cuts it if the costs are low.

> What costs do you consider?

We consider:

**Design** costs: is it going to take a lot of time to design the feature, verify that the design is sound, and write a specification? This is a tiny part of a larger feature; the cost is pretty low.

**Compiler development** costs: in this case we were going to have to completely rewrite the "object creation expression" parser logic anyway; making the argument list optional was a tiny fraction of the cost of the larger feature.

**IDE development** costs:: Does the proposed feature make it difficult to analyze an incomplete program as you are typing it? The IDE has to do that; features which make the IDE have to work much harder are bad features because the IDE has to be really snappy; it has to do semantic analysis between keystrokes. This sugar doesn't seem to add any complex lexical, grammatical or semantic analysis ambiguities. It seems to work well with refactorings, and other complex analysis that the IDE has to do.

**Testing** costs: does the feature introduce lots of scenarios that are difficult to test? In this case, no. It's very easy to test this sort of optional syntax, and again, the test cost of this part of the feature was a small fraction of the cost of testing the whole thing.

**User Education** costs: does the feature introduce syntax that will require a lot of documentation? Are we going to have to produce videos and blog posts about it? Are we going to get support calls from customers who are confused by the feature? For this sugar, no, no, and no.

**Maintenance** costs: what will the "bug tail" look like? Do we anticipate that we're likely to miss some crazy interaction with some other feature that causes a hard-to-reproduce and hard-to-fix bug? Or is it likely that we'll get it right the first time? In this case I do not recall there being any bugs reported against the optional-ness of the argument list. It's easy to get right.

**Opportunity** costs: by doing this sugar, are we spending lots of time and effort that we could be spending on something better for customers? Not really; again, this was a small fraction of the cost of the whole object-and-collection-initializers feature.

**Future **costs: Does the proposed feature "cut off" any anticipated future features, or make them more expensive and difficult? No. It seems unlikely that we're going to say in the future "well, we could have added metaprogramming to C\# if only we had not made argument lists optional."

All the costs are low and the feature is nice to have, so we did the feature.

> Why did you not add even more sugar at the same time?

Next time\!

(﻿This is part one of a three-part series on C\# language design issues involving elided parentheses in constructor calls. Part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/09/23/ambiguous-optional-parentheses-part-two.aspx).)

