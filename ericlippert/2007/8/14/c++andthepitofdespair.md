# C++ and the Pit Of Despair

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/14/2007 11:25:00 AM

-----

[Raymond has an interesting post today](http://blogs.msdn.com/oldnewthing/archive/2007/08/14/4374222.aspx) about two subtle aspects of C\#: how order of evaluation in an expression is specified as strictly left-to-right, and how the rules regarding local shadowing ensure that an identifier has exactly one meaning in a local scope. He makes an educated guess that the reason for these sorts of rules is to "*reduce the frequency of a category of subtle bugs*".

I'd like to take this opportunity to both confirm that guess and to expand upon it a bit.

**Eliminating Subtle Bugs** 

You remember in *The Princess Bride* when Westley wakes up and finds himself locked in [The Pit Of Despair](http://www.wavcentral.com/movies/pbride.html) with a hoarse albino and the sinister six-fingered man, Count Rugen? The principle idea of a pit of despair is twofold. First, that it is a *pit*, and therefore easy to fall into but difficult work to climb out of. And second, that it induces *despair*. Hence the name.

I often think of C++ as my own personal Pit of Despair Programming Language. Unmanaged C++ makes it so easy to fall into traps. Think buffer overruns, memory leaks, double frees, mismatch between allocator and deallocator, using freed memory, umpteen dozen ways to trash the stack or heap -- and those are just some of the memory issues. There are lots more "gotchas" in C++. C++ often throws you into the Pit of Despair and you have to climb your way up the Hill of Quality. (Not to be confused with scaling the Cliffs of Insanity. That's different.)

Now, [as I've said before, the design of C\# is not a subtractive process](http://blogs.msdn.com/ericlippert/archive/2007/05/14/why-are-overloaded-operators-always-static-in-c.aspx). It is not "C++ with the stupid parts taken out". But that said, it would be rather foolish of us to not look at what problems people have typically had with other languages and work to ensure that those exact same problems do not crop up for C\# users. I would like C\# to be a "Pit of Quality" language, *a language where its rules encourage you to write correct code in the first place*. You have to work quite hard to write a buffer overrun bug into a C\# program, and that's on purpose.

I have never written a buffer overrun in C\#. I have never written a bug where I accidentally shadowed a variable in another scope in C\#. I have never used stack memory after the function returned in C\#. I've done all those things in C++ multiple times, and it's not because I'm an idiot, it's because C++ makes it easy to do all those things accidentally and C\# makes it very hard. Making it easy to do good stuff is obviously goodness; *thinking about how to make it hard to do bad is actually more important.*

Now, given that the design of C\# is not subtractive, we have to consider the pros and cons of each decision. Is there any compelling user *benefit* in a *deliberate* *failure* to specify what order functions in an expression are evaluated? The only benefit I can think of is "not breaking some existing users of two existing incompatible implementations by declaring one of the implementations to be wrong", which is the situation that the C standardization committee frequently found itself in, I'm sure. When C\# was a new language that wasn't an issue, so we were free to pin that down early. Doing so has compelling benefits; it prevents subtle bugs, and as I'll discuss in a moment, there are other benefits as well.

So, long story short, yes, designing the language so as to prevent certain categories of subtle bugs is a huge priority for us. However, there are other reasons too. For instance:

**Uncertainty sometimes has high value but only in special contexts**

Like Vroomfondel said, "*we demand rigidly defined areas of doubt and uncertainty\!*" Ideally, those areas should be small and should afford some way for users to eliminate the uncertainty. Nondeterministic finalization is a good example. We deliberately do not specify when and in what order finalizers run because:

1.  the vast majority of the time it makes no difference,
2.  relying on a particular timing or ordering of finalization some small percentage of the time is probably a subtle bug waiting to happen,
3.  specifying it would require us to simplify the implementation to the point where it actually could be specified, thereby destroying much of its value; there is value in that complexity
4.  specifying it ties our hands to make algorithm improvements in the future

But we do provide a mechanism (the "using" statement) whereby if you do need to ensure that a finalizer runs at a particular point, there is an easy syntax for it.

With the possible exception of point 2, the order of evaluation of sub-expressions does not have these problems. It often does make a difference, the implementation is already simple enough that the specification is trivial, and its unlikely that we are going to come up with some incredible improvement in the algorithm which determines what order subexpressions are evaluated in. And if by some miracle we do, the specification does take care to call out that if we can prove that out-of-order evaluation cannot introduce subtle bugs, then we reserve the right to do it.

**Uncertainty is hard on the language implementation team**

When I am implementing part of the language specification, of course I want great freedom to decide *how to implement* a particular chunk of semantics. The language specification is not in the business of telling me whether we should be using a hand-written recursive descent parser or a parser-generator that consumes the grammar, etc. But I hate, hate, *hate* when I'm reading the specification and I'm left with a choice of *what semantics to implement*. I *will* choose wrong. I know this, because [when I ask you guys what the "intuitively obvious" thing to do is, significant fractions of the population disagree](http://blogs.msdn.com/ericlippert/archive/2006/06/27/648681.aspx)\!

Uncertainty is also hard on our testers -- they would like to know whether the language is correct, and being told "the specification is unclear, therefore whatever we do is correct" makes their jobs harder because that's a lie. Crashing horribly is probably not correct. Deadlocking is probably not correct. Somewhere on that spectrum between clearly correct behaviour and clearly incorrect behaviour is a line, and one of testing's jobs is to ensure that the developers produced an implementation that falls on the "correct" side of that line. Not knowing precisely where the line is creates unnecessary work for them, and they are overworked already.

And uncertainty is hard on our user education people. They want to be able to write documentation which clearly describes what semantics the implementation actually implements, and they want to be able to write sample programs to illustrate it.

**Uncertainty erodes confidence** 

And finally, uncertainty erodes confidence in our users that we have a quality product that was designed with deep thought, implemented with care, and behaves predictably and sensibly. People entrust the operation of multi-million dollar businesses to their C\# code; they're not going to do that if we can't even reliably tell them what "(++i) + i + (i++)" does\!

\*\*\*\*\*

Next time on FAIC I think I'll talk a bit about a related principle of language design: how we think about issues involving "breaking changes". I'll be expanding upon [an excellent article written a couple of years ago by my colleague Neal](http://blogs.msdn.com/nealho/archive/2005/11/22/496101.aspx), so you might want to start there.

