# Back in the saddle

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/28/2006 1:28:00 PM

-----

Hey everyone, I'm back. Sorry for the two-month absence there. We've been absolutely INSANELY busy trying to figure out how to make the LINQ features work in C\# 3.0 for the last two months. Add on top of that the fact that I've been running lights for an amateur theatre production for the last couple weeks -- I've had absolutely no time for blogging.

There's a huge amount more that I could go into in my series on regular expressions and finite automata. A dauntingly huge amount. I'd really like to explore some of these ideas. I could show how there is a finite automaton for every regular language and a regular language for every FA, and then find some non-regular languages, look at push-down automata, Turing machines, nondeterministic Turing machines, computable and uncomputable numbers, really interesting stuff. We could also go into practical concerns about how to build devices that parse real-world languages like Jscript or C\#.

But I just don't have the kind of time right now it would take to do that kind of full treatment. I want to get back into shorter topics. In the last few months that I've been exploring the C\# compiler I've found all kinds of interesting nooks and crannies and weird ambiguities and flaws in the implementation. And I'm still getting questions about scripting – in fact, there has been a recent flurry of interest in Jscript and Jscript.NET, which seem to be making a bit of a renaissance.

Therefore I thought I might change gears for a bit and get back into more practical language issues. I'm going to start with further proof that premature optimization really is the root of all evil.

