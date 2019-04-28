# Danger, Will Robinson\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/3/2011 10:41:00 AM

-----

As long-time readers of this blog know, I am often asked why a particular hunk of bad-smelling code does *not* produce a compiler warning.

"Why not?" questions are inherently hard to answer because they turn causation on its head; normally we ask what caused a particular thing to happen, not what caused a particular thing to not happen. Therefore, rather than attack that question directly I like to rephrase the question into questions about the proposed feature. (A warning is of course a feature like any other.) By answering these questions we can see whether the compiler team is likely to spend its limited budget on the feature rather than prioritizing something else. Some questions I think about when considering "why did you not implement this warning?" are:

**Did someone think of it?**

If no one on the compiler team ever thinks of the possibility of a particular warning then obviously it won't happen. As an example of that, I was recently asked why:

 

using(null) statement;

does not produce a warning. That code is legal; it is equivalent to

 

IDisposable temp = null;  
try  
{  
  statement;  
}  
finally  
{  
  if (temp \!= null) temp.Dispose();  
}

Of course the code is completely pointless; all it does is introduce a try-finally block that adds nothing useful to the program. Why don't we give a warning for this useless code? Well, the primary reason is that (to my knowledge) *no one on the compiler team ever thought that a customer would do such a thing*, and therefore, never thought to design, implement, test and document a compiler feature to deal with the situation.

Now that we *have* thought of it, we can see if it meets any of our other criteria, like, *is it plausible that a user would type this accidentally thinking that it does something sensible?* But I'm getting ahead of myself.

**Is the proposed warning even possible? Is it really expensive?**

Sometimes people propose warnings that would require [solving the Halting Problem](http://blogs.msdn.com/b/ericlippert/archive/2011/02/24/never-say-never-part-two.aspx) (which is unsolvable in general) or solving an [NP-hard problem](http://blogs.msdn.com/b/ericlippert/archive/2007/03/28/lambda-expressions-vs-anonymous-methods-part-five.aspx) (which is in practice unsolvable in a reasonable amount of time). Proposed warnings like "*warn me if there is no possible way for this overload to be chosen*" or "*warn me if there is an input which guarantees that this recursive method has unbounded recursion*" or "*warn me if this argument can ever be null in this program*" require a level of analysis that is either in principle impossible, or possible but better performed by tools custom made for that purpose, like the Code Contracts engine. **Warning scenarios should be cheap for the compiler to disambiguate from non-warning scenarios.**

**Is the code being warned about both *plausible* and *likely to be wrong*? **Is the potentially "wrong" code actually sensible in some scenario?****

There are an infinite number of ways to write completely crazy code; the point of warnings is not to identify all possible crazy code, but rather to identify code that is plausible but likely to be wrong. We don't want to waste time and effort implementing features warning about ridiculous scenarios that in practice never arise.

Sometimes code seems to be both plausible and wrong, but is intentional for a reason that is not immediately obvious. The best example of this is [our suppression of warnings about write-only locals](http://blogs.msdn.com/b/ericlippert/archive/2007/04/23/write-only-variables-considered-harmful-or-beneficial.aspx); a local might be read by the developer during debugging even if it is never read in the program.

Machine-generated code is also often crazy-seeming but correct and intentional. It can be hard enough to write code generators that generate legal code; writing code generators that change their output when they determine that they're generating code that could produce warnings is burdensome. (For example, consider the machine-generated code that you get when you create a new project in Visual Studio. [It had better compile without warnings](http://blogs.msdn.com/b/ericlippert/archive/2010/01/25/why-are-unused-using-directives-not-a-warning.aspx)\!) However, we are much more concerned with catching errors in code written by humans than we are about making machines happy.

Obviously we only want to warn about code that has a high likelihood of actually being wrong. If we warn about correct code then we are encouraging users to change correct code, probably by changing it into incorrect code. A **warning should be a bug-*preventing* feature, not a bug-*encouraging* feature**. This leads me to my next point:

**Is there a clear way to rewrite correct code that gets an unwanted warning into correct code that does not get a warning?**

Warnings ideally ought to be easily turn-off-able. For example:

 

bool x = N();  
...  
if (x == null ) Q();

That gives a warning that == null on a non-nullable value type is always false. Clearly you can rewrite the code to eliminate the warning; if you intended it to always be false then get rid of the whole statement, or you can turn the variable into a nullable bool, or whatever.

A warning where there is no way to write the code so that the warning disappears is very irritating to people who actually do mean the seemingly-wrong thing. You can always turn a warning off with a \#pragma of course, but that's a horribly ugly thing to force someone to do.

**Will a new warning turn large amounts of existing code into errors?**

Lots of people compile with "warnings as errors" turned on. [Every time we add a warning that causes a lot of existing code to produce warnings, we potentially break a lot of people](http://blogs.msdn.com/b/ericlippert/archive/2007/05/15/should-we-produce-warnings-for-unused-uninitialzied-internal-fields.aspx). Even if the warnings are good ones, breaking a lot of people is points against doing the feature. Such warnings are perhaps best handled in another tool, which brings me to:

**Does the compiler team have to do this work? Or can another tool produce the warning?**

Microsoft provides tools like [FxCop](http://msdn.microsoft.com/en-us/library/dd264939\(v=VS.100\).aspx), [StyleCop](http://stylecop.codeplex.com/) and [Code Contracts](http://msdn.microsoft.com/en-us/devlabs/dd491992.aspx) to do far more extensive analysis of code than the C\# compiler performs. Similarly with third-party tools like [ReSharper](http://www.jetbrains.com/resharper/). If a warning can be implemented by one of those tools just as easily or more easily than it can be implemented in the compiler then that's less work for me, and that's *awesome*. Also, since FxCop examines compiled state, a warning in FxCop can catch problems in C\# and VB, without having to change either compiler.

**Will the user discover the error immediately upon testing anyway?**

The earlier a bug is discovered, the cheaper it is to fix. The opportunities to find bugs are, in order of exponentially increasing cost of fixing the bug:

\* When you type in the code  
\* When you compile the code  
\* When you test the code  
\* When your customer runs the code

Warnings about potential bugs that pop up while you are typing the code or compiling the code are bugs that never make it to the customer. However, a potential warning that points out a bug that would *always* be caught by testing is less compelling than a warning for a bug that *could* make it through testing to the customer.

For example, I was recently asked why the compiler does not provide a warning for this common typing error:

 

private decimal cost;  
public decimal Cost { get { return this.Cost; } }

Whoops, that's a stack overflow exception (or, on machines with tail recursion optimizations, an infinite loop) waiting to happen right there. The compiler can in principle determine cheaply that property getter does nothing other than call itself, so why doesn't the compiler warn about it? We could do all the work of identifying code that has this pattern, but why bother warning about a bug that will be caught anyway? The instant you test this code you will *immediately* discover the problem and it will be clear how to fix it. This is unlike our earlier example of "if (x == null)", where the fact that there is unreachable code might go completely unnoticed during testing, particularly if the common scenario is for x to be false.

**Summing up:**

Since that is quite the gauntlet for a feature to run, we find that usually the right thing to do for a proposed warning is "don't add the warning". The most likely situation for lots of new warnings to be added is when the warnings are surrounding a new feature that has new ways of being misused; we added quite a few warnings in C\# 4.0 for misuses of "dynamic", "no PIA", and "named and optional arguments". Warnings for new features have no existing code using them that would break.

That said, we can always improve; if you have ideas for warnings do keep them coming.

