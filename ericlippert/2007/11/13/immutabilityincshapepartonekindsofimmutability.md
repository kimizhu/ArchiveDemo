# Immutability in C\# Part One: Kinds of Immutability

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/13/2007 12:56:00 PM

-----

[I said in an earlier post that I believe that immutable objects are the way of the future in C\#.](http://blogs.msdn.com/ericlippert/archive/2007/10/04/path-finding-using-a-in-c-3-0-part-two.aspx) I stand by that statement while at the same time noting that it is at this point sufficiently vague as to be practically meaningless\! “Immutable” means different things to different people; different kinds of immutability have different pros and cons. I’d like to spend some time over the next few weeks talking about possible directions that C\# could go to improve the developer experience when writing programs that use immutable objects, as well as giving some practical examples of the sort of immutable object programming you can do today.

(Again, I want to emphasize that in these sorts of “future feature” posts we are **all playfully hypothesizing and brainstorming about ideas for entirely hypothetical future versions of C\#.** We have not yet shipped C\# 3.0 and have not announced that there will ever be any future version of the language. Nothing here should be construed as any kind of promise or announcement; we’re just geeks talking about programming languages, ‘cause that’s what we do.)

So, disclaimers out of the way, what kinds of immutability are there? Lots. Here’s just a few. Note that these categories are not necessarily mutually exclusive\!

**Realio-trulio immutability:**

There’s nothing you can do to the number one that changes it. You cannot paint it purple (‡), make it even or get it angry. It’s the number one, it is eternal, implacable and unchanging. Attempting to do something to it – say, adding three to it – doesn’t change the number one at all. Rather, it produces an entirely different and also immutable number. If you cast it to a double, you don’t change the integer one; rather, you get a brand new double.

Strings, numbers and the null value are all truly immutable.

C\# allows you to declare truly immutable named fields with the const keyword. The compiler ensures that the only things that are allowed to go into const fields are truly immutable things – numbers, strings, null. (See the section of the standard on “constant expressions” for details.)

**Write-once immutability:**

Fields marked as const have to be compile-time constants, which is a bit of a pain if what you want to do is have a field which never changes but nevertheless cannot be computed until runtime. For example, in a later post I’m going to define an immutable stack class which has this code:

 

    public sealed class Stack\<T\> : IStack\<T\>  
    {  
        private sealed class EmptyStack : IStack\<T\>  
        { /\* ... \*/ }  
        private static readonly EmptyStack empty = new EmptyStack();  
        public static IStack\<T\> Empty { get { return empty; } }

I will want to create a singleton empty stack. Clearly it is not a compile-time constant, so I cannot make the field const. But I want to say “once this thing is initialized it is never going to change again.” That’s what the readonly modifier ensures. Basically it’s a “write only once” field. Not exactly immutable, since obviously it changes exactly once, from null to having a value. But pretty darn immutable.

**Popsicle immutability:**

...is what I whimsically call a slight weakening of write-once immutability. One could imagine an object or a field which remained mutable for a little while during its initialization, and then got “frozen” forever. This kind of immutability is particularly useful for immutable objects which circularly reference each other, or immutable objects which have been serialized to disk and upon deserialization need to be “fluid” until the entire deserialization process is done, at which point all the objects may be frozen.

There is at present no really universal convention for how to declare a freezable object, and there certainly is no support in the compiler for this kind of immutability.

**Shallow vs deep immutability:**

Consider a write-once field containing an array:

 

public class C {  
    private static readonly int\[\] ints = new int\[\] { 1, 2, 3 };  
    public static int\[\] Ints { get { return ints; } }

The value of the field cannot be changed; C.ints = null; would be illegal even from inside the class. This is a sort of “referential” immutability. But there is nothing immutable at all about the array itself\! C.Ints\[1\] = 100; is still perfectly legal from outside the class.

The ints field is “shallowly” immutable. You can rely upon it being immutable to a certain extent, but once you reach a point where there is a reference to a mutable object, all bets are off.

Obviously the opposite of shallow immutability is “deep” immutability; in a deeply immutable object it is immutable all the way down.

If we had immutability in the type system, something like the far stronger kind of “const” in C/C++, then a hypothetical future compiler could verify that an object marked as deeply immutable had only deeply immutable fields.

Objects which are truly madly deeply immutable have a lot of great properties. They are 100% threadsafe, for example, since obviously there will be no conflicts between readers and (non-existant) writers. They are easier to reason about than objects which can change. But their strict requirements may be more than we need, or more than is practical to achieve.

**Immutable facades:**

Since the contents of an array (though, interestingly enough, not its size) may be changed arbitrarily, it’s a bad idea to expose data that you want to be logically read-only in a public array field. To make this a bit easier, the base class library lets you say

 

public class C {  
    private static readonly intarray = new int\[\] { 1, 2, 3 };  
    public static readonly ReadOnlyCollection\<int\> ints = new ReadOnlyCollection\<int\>(intarray);  
    public static ReadOnlyCollection\<int\> Ints { get { return ints; } }

The read-only collection has the interface of a regular collection; it just throws an exception every time a method which would modify the collection is called. However, clearly the underlying collection is still mutable. Code inside C could mutate the array members.

Another down side of this kind of immutability is that the compiler is unable to detect attempts to modify the collection. Attempts to, say, add new members to the collection will fail at runtime, not at compile time.

This sort of immutability is a special case of...

**Observational immutability:**

Suppose you’ve got an object which has the property that every time you call a method on it, look at a field, etc, you get the same result. From the point of view of the caller such an object would be immutable. However you could imagine that behind the scenes the object was doing lazy initialization, memoizing results of function calls in a hash table, etc. The “guts” of the object might be entirely mutable.

What does it matter? Truly deeply immutable objects never change their internal state at all, and are therefore inherently threadsafe. An object which is mutable behind the scenes might still need to have complicated threading code in order to protect its internal mutable state from corruption should the object be called on two threads “at the same time”.

**Summing up:**

Holy goodness, this is complicated\! And we have just barely touched upon the deeply complex relationship between immutability of objects and “purity” of methods, which opens up huge cans of worms.

So, smart people, what do you think? Are there forms of immutability which I did not touch upon here that you like to take advantage of in your programs? Are there any particular forms of immutability which you would like to see made easier to use in C\#?

**Next time:** let’s get a little more practical. I already implemented an immutable stack in my A\* series, but that was pretty special-purpose. We’ll take a look at how one might implement a general-purpose immutable stack today in C\# 3.0. We'll then expand that to immutable queues, trees, etc. (And I might even discuss **how one could take advantage of typesafe covariance when designing interfaces for immutable data structures,** oh frabjous day\!)

(‡) A dear old friend of mine from school who happens to be a [grapheme-colour synaesthete](http://en.wikipedia.org/wiki/Grapheme-color_synesthesia) tells me that of course you cannot paint the number one purple because it is already blue. Silly me\!

