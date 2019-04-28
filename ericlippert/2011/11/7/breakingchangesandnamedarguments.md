# Breaking changes and named arguments

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/7/2011 11:00:15 AM

-----

Before I get into the subject of today's post, **thanks so much to all of you who have given us great feedback on the Roslyn CTP. Please keep it coming**. I'm definitely going to do some articles on Roslyn in the future; the past few weeks I have been too busy actually implementing it to write much about it.

-----

We introduced "named and optional arguments" to C\# 4.0; that's the feature whereby you can omit some optional arguments from a method call, and specify some of them by name. This makes the code more self-documenting; it also makes it much easier to deal with methods in legacy object models that have lots and lots of parameters but only a few are relevant. The most common commentary I've seen about the "named" side of that feature is people noting that this introduces a new kind of breaking change. The commentary usually goes something like:

*Suppose you are making a library for consumption by others. Of course if you change details of the public surface area of the library, you can break consumers of the library. For example, if you add a new method to a class then you might make an existing program that compiled just fine now turn into an error-producing program because a call to the old method is now ambiguous with a call to the new method. In the past, changing only the name of a formal parameter was not a "breaking change", because consumers always passed arguments positionally. Recompilation would not produce different results. In a world with named arguments though, it is now the case that changing the name of a formal parameter can cause working code to break.*

Though that commentary is certainly well-intentioned and informative, it is not *quite* accurate. The implication is that this breaking change is something new, when in fact it has *always* been the case that a library provider could break consumers by changing only the name of a formal parameter. **Visual Basic has always supported calling methods with named parameters**. In the past, changing the name of a formal parameter was a breaking change for any consumers using Visual Basic, and that's potentially a lot of people. Now it becomes a potential breaking change for even more users. Adding a new feature to C\# doesn't change the fact that the risk was always there\!

Library providers should *already* have been cautious about changing the names of formal parameters, and they should continue to be cautious now.

