# String interning and String.Empty

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/28/2009 9:23:00 AM

-----

Here's a curious program fragment:

 

object obj = "Int32";  
string str1 = "Int32";  
string str2 = typeof(int).Name;  
Console.WriteLine(obj == str1); // true  
Console.WriteLine(str1 == str2); // true  
Console.WriteLine(obj == str2); // false \!?

Surely if A equals B, and B equals C, then A equals C; that's the *transitive property* of equality. It appears to have been thoroughly violated here.

Well, first off, though the transitive property is desirable, this is just one of many situations in which equality is intransitive in C\#. You shouldn't rely upon transitivity *in general*, though of course there are many specific cases where it is valid. As an exercise, you might want to see how many other intransitivities you can come up with. Post 'em in the comments; I'd love to see what obscure ones you can come up with. (Incidentally, one of the interview questions I got when applying for this team was to invent a performant algorithm for determining intransitivities in a simplified version of the 'better method' algorithm.)

Second, what's happening here is we're mixing two different kinds of equality that just happen to use the same operator syntax. We're mixing *reference* equality with *value* equality. Objects are compared by reference; in the first and third comparison we are testing if the two object references both refer to exactly the same object. In the second comparison we are checking to see if the two strings have the same content, regardless of whether they are the same object or not. In fact, the compiler warns you about this situation; this should produce a "possible unintended reference comparison" warning.

That might need a bit more explanation. *In .NET you can have two strings that have identical content but are different objects*. When you compare those strings *as strings*, they're equal, but when you compare them *as objects*, they're not.

That explains why the second comparison is true -- it's a value comparison -- and why the third comparison is false -- it's a reference comparison. But it doesn't explain why the first and third comparisons are inconsistent with each other.

This is the result of a small optimization. If you have two identical string literals in one compilation unit then the code we generate ensures that *only one string object is created by the CLR for all instances of that literal within the assembly*. This optimization is called "string interning".

String.Empty is not a constant, it's a read-only field in another assembly. Therefore it is not interned with the empty string in your assembly; those are two different objects.

This explains why the first comparison is true: the two literals in fact get turned into the same string object. And it explains why the third comparison is false: the literal and the computed value are turned into different objects.

Knowing that, you can now make an educated guess as to why we have this bizarre behaviour:

 

object obj = "";  
string str1 = "";  
string str2 = String.Empty;  
Console.WriteLine(obj == str1); // true  
Console.WriteLine(str1 == str2); // true  
Console.WriteLine(obj == str2); // sometimes true, sometimes false?\!

Some versions of the .NET runtime automatically intern the empty string at runtime, some do not\!

But why, you might ask, do we not perform this interning optimization *at runtime* on *every string*? Why not aggressively turn all value-equal strings into reference-equal strings? Surely it is wasteful to have two identical strings around when you could have half as much memory.

The answer is that the [TANSTAAFL](http://en.wikipedia.org/wiki/TANSTAAFL) Principle applies here, bigtime. That is, **There Ain't No Such Thing As A Free Lunch**. Interning has two positive effects: it decreases memory consumption and decreases time required to compare two strings. (Because if all strings are interned at runtime then *all* string comparisons can be cheap reference comparisons.) But those positive effects have a cost: allocating a new string now requires that you do a search of all string objects in memory to see if you have one that matches already. In our existing optimization, the cost is small; we can know at compile time what string literals are in a given assembly and which are identical. With the proposed optimization, that cost is imposed at runtime, and it could be a very large fraction of the time spent allocating strings.

In order to keep the time cost down, you'd have to build a hash table of all strings in memory. That means either computing the hashes frequently, which is itself expensive in time, or storing the hashes somewhere. If we do the latter then suddenly we are *increasing* the memory burden for strings that are *not duplicated*. That is, our optimization makes the normal scenario -- the vast majority of pairs of strings are not equal to each other -- take up more memory, so that a rare scenario saves on memory. That seems like a bad bargain; you usually want to optimize for the likely case.

There are also serious lifetime problems with interned strings. When can they be safely garbage collected? What if a new copy of the string is created while the old one is being collected on another thread? The safest thing to do is to make interned strings immortal, which looks like a memory leak. Memory leaks are bad for performance, particularly when the optimization you're doing is an attempt to *save memory*. TANSTAAFL\!

In short, it is in the general case not worth it to intern all strings. *However, it might be worth it in some specific cases.* For example, if you were building a compiler in C\#, odds are good that you are going to be producing a lot of strings that are the same at runtime. Our C\# compiler is written in C++, in which we have written our own custom string interning layer so that we can do cheap reference comparisons on all strings in your program. Odds are good that "int" is going to appear tens, hundreds or thousands of times in a given program; it seems silly to allocate the same string over and over again. If you were writing a compiler in C\#, or had some other application in which you felt that it was worth your while to ensure that thousands of identical strings do not consume lots of memory, you can force the runtime to intern your string with the [String.Intern](http://msdn.microsoft.com/en-us/library/system.string.intern.aspx) method.

Conversely, if you hate interning with an unreasoning passion, you can force the runtime to turn off all string interning in an assembly with the [CompilationRelaxation](http://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.compilationrelaxationsattribute.aspx) attribute.

Anyway, to come back to the question of transitivity: object reference equality actually is transitive. It's also symmetric (A==B implies B==A) and reflexive (A==A), so it is an equivalence relation. Similarly, string value equality is transitive, symmetric and reflexive, since it uses a straight "character by character" ordinal comparison. But when you *mix* the two, then equality is no longer transitive. That's weird, but hopefully now understandable.

