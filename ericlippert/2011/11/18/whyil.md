# Why IL?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/18/2011 9:22:17 AM

-----

One of the earliest and most frequently-asked questions we got [when we announced the Roslyn project](http://blogs.msdn.com/b/ericlippert/archive/2011/10/19/the-roslyn-preview-is-now-available.aspx) was "is this like LLVM for .NET?"

No, Roslyn is not anything like LLVM for .NET. LLVM stands for [Low-Level Virtual Machine](http://en.wikipedia.org/wiki/LLVM); as I understand it (admittedly never having used it), compiler "front ends" take in code written in some language -- say C++ -- and spit out equivalent code written in the LLVM language. Another compiler then takes the code written in the LLVM language and writes that into optimized machine code.

We already have such a system for .NET; in fact, .NET is entirely built upon it and always has been, so Roslyn isn't it. The C\#, VB and other compilers take in programs written in those languages and spit out code written in the [Common Intermediate Language](http://en.wikipedia.org/wiki/Common_Intermediate_Language) (CIL, or also commonly MSIL or just IL). Then another compiler -- either the jitter, which runs "just in time" at runtime, or the NGEN tool which runs before runtime -- translates the IL into optimized machine code that can actually run on the target platform.

I'm occasionally asked why we use this strategy; why not just have the C\# compiler write out optimized machine code directly, and skip the middleman? Why have two compilers to go from C\# to machine code when you could have one?

There are a number of reasons, but they pretty much all boil down to one good reason: **the two-compilers-with-an-intermediate-language system is much less expensive in our scenario.**

That might seem counterintuitive; after all, now we have two languages to specify, two languages to analyze, and so on. To understand why this is such a big win you have to look at the larger picture.

Suppose you have n languages: C\#, VB, F\#, JScript .NET, and so on. Suppose you have m different runtime environments: Windows machines running on x86 or x64, XBOX 360, phones, Silverlight running on the Mac... and suppose you go with the one-compiler strategy for each. **How many compiler back-end code generators do you end up writing?** For each language you need a code generator for each target environment, so you end up writing n x m code generators.

Suppose instead you have every language generate code into IL, and then you have one jitter per target environment. How many code generators do you end up writing?  One per language to go to IL, and one per environment to go from IL to the target machine code. That's only n + m, which is far less than n x m for reasonably-sized values of n and m.

Moreover, there are other economies in play as well. IL is deliberately designed so that it is very easy for compiler writers to generate correct IL. I'm an expert on the semantic analysis of the C\# language, not on efficient code generation for cellular phone chipsets. If I had to write a new backend code generator for every platform the .NET framework runs on, I'd be spending all my time doing a bad job of writing code generators instead of doing a good job writing semantic analyzers.

The cost savings go the other way too; if you want to support a new chipset then you just write yourself a jitter for that chipset and all the languages that compile to IL suddenly start working; you only had to write \*one\* jitter to get n languages on your new platform.

This cost-saving strategy of putting an intermediate language in the middle is not at all new; [it goes back to at least the late 1960's](http://en.wikipedia.org/wiki/O-code_machine). My favourite example of this strategy is the [Infocom Z-Machine](http://en.wikipedia.org/wiki/Zmachine); the Infocom developers wrote their games in a language (Zork Implementation Language) that compiled to an intermediate Z-Code language, and then wrote Z-Machine interpreters for a variety of different platforms; as a result they could write n games and have them run on m different platforms at a cost of n + m, not n x m. (This approach also had the enormous benefit that they could implement *virtual memory management* on hardware that did not support virtual memory natively; if the game was too big to fit into memory, the interpreter could simply discard code that wasn't being used at the moment and page it back in again later as needed.)

Next time I'll talk a bit about why IL is specified the way it is.

