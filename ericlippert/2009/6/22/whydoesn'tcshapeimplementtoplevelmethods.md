# Why Doesn't C\# Implement "Top Level" Methods?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/22/2009 11:39:00 AM

-----

C\# requires that every method be in some class, even if it is a static method in a static class in the global namespace. Other languages allow "top level" functions. A[recent stackoverflow post asks why that is.](http://stackoverflow.com/questions/1024171/why-c-is-not-allowing-non-member-functions-like-c/1027853#1027853)

I am asked "why doesn't C\# implement feature X?" all the time. The answer is always the same: because no one ever designed, specified, implemented, tested, documented and shipped that feature. All six of those things are necessary to make a feature happen. All of them cost huge amounts of time, effort and money. Features are not cheap, and we try very hard to make sure that we are only shipping those features which give the best possible benefits to our users given our constrained time, effort and money budgets.

I understand that such a general answer probably does not address the specific question.

In this particular case, the clear user benefit was in the past not large enough to justify the complications to the language which would ensue. By restricting how different language entities nest inside each other we (1) restrict legal programs to be in a common, easily understood style, and (2) make it possible to define "identifier lookup" rules which are comprehensible, specifiable, implementable, testable and documentable.

By restricting method bodies to always be inside a struct or class, we make it easier to reason about the meaning of an unqualified identifier used in an invocation context; such a thing is always an invocable member of the current type (or a base type). 

Now, JScript.NET has this feature. (And in fact, JScript.NET goes even further; you can have program statements "at the top level" too.) A reasonable question is "why is this feature good for JScript but bad for C\#?"

First off, I reject the premise that the feature is "bad" for C\#. The feature might well be good for C\#, just not *good enough* compared to its costs (and to the *opportunity cost* of doing that feature *instead of a more valuable feature*.) The feature might become good enough for C\# if its costs are lowered, or if the compelling benefit to customers becomes higher.

Second, the question assumes that the feature is good for JScript.NET. Why is it good for JScript.NET?

It's good for JScript.NET because JScript.NET was designed to be a "scripty" language as well as a "large-scale development" language. "JScript classic"'s original design as a scripting language requires that "a one-line program actually be one line". If your intention is to make a language that allows for rapid development of short, simple scripts by novice developers then you want to minimize the amount of "ritual incantations" that must happen in every program. In JScript you do not want to have to start with a bunch of using clauses and define a class and then put stuff in the class and have a Main routine and blah blah blah, all this ritual just to get Hello World running.

C\# was designed to be a large-scale application development language geared towards pro devs from day one; it was never intended to be a scripting language. It's design therefore encourages enforcing the *immediate* organization of even small chunks of code into *components*. **C\# is a component-oriented language.** We therefore want to encourage programming in a component-based style and discourage features that work against that style.

This is changing. "REPL" languages like F\#, long popular in academia, are increasing in popularity in industry. There's a renewed interest in "scripty" application programmability via tools like Visual Studio Tools for Applications. These forces cause us to re-evaluate whether "a one line program is one line" is a sensible goal for hypothetical future versions of C\#. Hitherto it has been an explicit non-goal of the language design.

**(As always, whenever I discuss the hypothetical "next version of C\#", keep in mind that we have not announced any next version, that it might never happen, and that it is utterly premature to think about feature sets or schedules. All speculation about future versions of unannounced products should be taken as "for entertainment purposes only" musings, not as promises about future offerings.)**

We are therefore *considering* adding this feature to a hypothetical future version of C\#, in order to better support "scripty" scenarios and REPL evaluation. When the existence of powerful new tools is predicated upon the existence of language features, that is points towards getting the language features done.

UPDATE: [More thoughts on considerations motivating this potential change here](http://blogs.msdn.com/ericlippert/archive/2009/06/24/it-already-is-a-scripting-language.aspx).

