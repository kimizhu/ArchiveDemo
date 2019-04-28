# What's Up With Hungarian Notation?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/12/2003 4:12:00 PM

-----

I mentioned Hungarian Notation in my last post -- a topic of ongoing religious controversy amongst COM developers. Some people swear by it, some people swear about it.

The anti-Hungarian argument usually goes something like this:

"What is the point of having these ugly, hard-to-read prefixes in my code which tell me the type? I already know the type because of the declaration\! If I need to change the type from, say, unsigned to signed integer, I need to go and change every place I use the variable in my code. The benefit of being able to glance at the name and know the declaring type is not worth the maintenance headache."

For a long time I was mystified by this argument, because that's not how I use Hungarian at all. Eventually I discovered that **there are two completely contradictory philosophical approaches to Hungarian Notation.** Unfortunately, each can be considered "definitive", and the bad one is in widespread use.

The one I'll call "the sensible philosophy" is the one actually espoused by Charles Simonyi in his original article. Here's a quote from [Simonyi's paper](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dnvs600/html/HungaNotat.asp):

> **The basic idea is to name all quantities by their types**. \[...\] the concept of "type" in this context is determined by the set of operations that can be applied to a quantity. The test for type equivalence is simple: could the same set of operations be meaningfully applied to the quantities in questions? If so, the types are thought to be the same. If there are operations that apply to a quantity in exclusion of others, the type of the quantity is different. \[...\] **Note that the above definition of type \[...\] is a superset of the more common definition, which takes only the quantity's representation into account**. Naturally, if the representations of x and y are different, there will exist some operations that could be applied to x but not y, or the reverse.

(Emphasis added.)

What Simonyi is saying here is that the **point of Hungarian Notation is to extend the concept of "type" to encompass semantic information in addition to storage representation information.**

There is another philosophy which I call "the pointless philosophy". That's the one espoused by Charles Petzold in "Programming Windows". On page 51 of the fifth edition he says

> Very simply, the variable name begins with a lowercase letter or letters that denote the data type of the variable.  For example \[...\] the i prefix in iCmdShow stands for "integer".

And that's all\! According to Petzold, **Hungarian is for connoting the storage type of the variable.**

All of the arguments raised by the anti-Hungarians (with the exception of "its ugly") are arguments against the pointless philosophy\! And I agree with them: that is in fact a pointless interpretation of Hungarian notation which is more trouble than it is worth.

But Simonyi's original insight is extremely powerful\! When I see a piece of code that says

iFoo = iBar + iBlah;

I know that there are a bunch of integers involved, but I don't know the semantics of any of these. But if I see

cbFoo = cchBar + cbBlah;

then I know that there is a serious bug here\! Someone is adding a count of bytes to a count of characters, which will break on any Unicode or DBCS platform. Hungarian is a concise notation for semantics like "count", "index", "upper bound", and other common programming concepts.

In fact, back in 1996 I changed every variable name in the VBScript string library to have its proper Hungarian prefix. I found a considerable number of DBCS and Unicode bugs just by doing that, bugs which would have taken our testers weeks to find by trial and error.

By using the semantic approach rather than the storage approach we eliminate the anti-Hungarian arguments:

> I already know the type because of the declaration\!

No, the Hungarian prefix tells you the semantic usage, not the storage type. A cBar is a count of Bars whether the storage is a ushort or a long.

> If I need to change the type from, say, unsigned to signed integer, I need to go and change every place I use the variable in my code.

Annotate the semantics, not the storage. If you change the semantics of a variable then you need to also change every place it is used\!

> The benefit of being able to glance at the name and know the declaring type is not worth the maintenance headache.

But the benefit of knowing that you will never accidentally assign indexes to counts, or add apples to oranges, is worth it in many situations.

**UPDATE**: Joel Spolsky has written a similar article: [Making Wrong Code Look Wrong](http://www.joelonsoftware.com/articles/Wrong.html). Check it out\!

