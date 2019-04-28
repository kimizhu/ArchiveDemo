# What's the Difference? Part Three: fixed vs. fixed

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/27/2009 9:43:00 AM

-----

I got an email the other day that began:

> *I have a question about fixed sized buffers in C\#:* 
> 
>  
> 
> unsafe struct FixedBuffer { public fixed int buffer\[100\]; }
> 
> *Now by declaring buffer as fixed it is not movable...*

And my heart sank. This is one of those deeply unfortunate times when subtle choices made in the details of language design encourage misunderstandings.

When doing pointer arithmetic in unsafe code on a managed object, you need to make sure that the garbage collector does not move the memory you're looking at. If a collection on another thread happens while you're doing pointer arithmetic on an object, the pointer can get all messed up. Therefore, C\# classifies all variables as "fixed" or "movable". If you want to do pointer arithmetic on a movable object, you can use the "fixed" keyword to say "this local variable contains data which the garbage collector should not move." When a collection happens, the garbage collector needs to look at all the local variables for in-flight calls (because of course, stuff that is in local variables needs to stay alive); if it sees a "fixed" local variable then it makes a note to itself to not move that memory, even if that fragments the managed heap. (This is why it is important to keep stuff fixed for as little time as possible.) So typically, we use "fixed' to mean "fixed in place".

But that's not what "fixed" means in this context; this means "the buffer in question is fixed in size to be one hundred ints" -- basically, it's the same as generating one hundred int fields in this structure.

Obviously we often use the same keyword to mean conceptually the same thing. For example, we use the keyword "internal" in many ways in C\#, but all of them are conceptually the same. It is only ever used to mean "accessibility to some entity is unrestricted to all code in this assembly".

Sometimes we use the same keyword to mean two completely different things, and rely upon context for the user to figure out which meaning is intended. For example:

 

var results = from c in customers **where** c.City == "London" select c;

vs

 

class C\<T\> **where** T : IComparable\<T\>

It should be clear that "where" is being used in two completely different ways: to build a filter clause in a query, and to declare a type constraint on a generic type parameter.

We cause people to run into trouble when one keyword is used in two different ways but the difference is *subtle*, like our example above. The user's email went on to ask a whole bunch of questions which were predicated on the incorrect assumption that a fixed-in-size buffer is automatically fixed in place in memory.

Now, one could say that this is just an unfortunate confluence of terms; that "fixed in size" and "fixed in place" just happen to both use the word "fixed" in two different ways, how vexing. But the connection is deeper than that: **you cannot safely access the data stored in a *fixed-in-size* buffer unless the container of the buffer has been *fixed in place*.** The two concepts are actually quite *strongly related* in this case, but not at all *the same*.

On the one hand it might have been less confusing to use *two* keywords, say "pinned" and "fixed". But on the other hand, both usages of "fixed" are only valid in unsafe code. A key assumption of all unsafe code features is that if you are willing to use unsafe code in C\#, then you are already an expert programmer who fully understands memory management in the CLR. That's why we make you write "unsafe" on the code; it indicates that you're turning off the safety system and you know what you're doing.

A considerable fraction of the keywords of C\# are used in two or more ways: fixed, into, partial, out, in, new, delegate, where, using, class, struct, true, false, base, this, event, return and void all have at least two different meanings. Most of those are clear from the context, but at least the first three -- fixed, into and partial -- have caused enough confusion that I've gotten questions about the differences from perplexed users. I'll take a look at "into" and "partial" next.

