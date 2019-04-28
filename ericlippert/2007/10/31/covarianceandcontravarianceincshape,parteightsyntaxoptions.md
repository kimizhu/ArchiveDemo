# Covariance and Contravariance in C\#, Part Eight: Syntax Options

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/31/2007 10:48:00 AM

-----

As I discussed last time, were we to introduce interface and delegate variance in a hypothetical future version of C\# we would need a syntax for it. Here are some possibilities that immediately come to mind.

**Option 1:**

 

interface IFoo\<+T, -U\> { T Foo(U u); }

The CLR uses the convention I have been using so far in this series of “+ means covariant, - means contravariant”. Though this does have some mnemonic value (because + means “is compatible with a bigger type”), most people (including members of the C\# design committee\!) have a hard time remembering exactly which is which.

This convention is also used by the Scala programming language.

**Option 2:**

 

interface IFoo\<T:\*, \*:U\> { …

This more graphically indicates “something which is extended by T” and “something which extends U”.  This is similar to Java’s “wildcard types”, where they say “? extends U” or “? super T”.

Though this isn’t terrible, I think it’s a bit of a conflation of the notions of *extension* and *assignment compatibility*. I do not want to imply that IEnumerable\<Animal\> is a *base* of IEnumerable\<Giraffe\>, even if Animal is a *base* of Giraffe. Rather, I want to say that IEnumerable\<Giraffe\> is *convertible to* IEnumerable\<Animal\>, or *assignment compatible*, or some such thing. **I don’t want to conceptually overwork the inheritance mechanism.** It's bad enough IMO that we conflate base classes with base interfaces.

**Option 3:**

 

interface IFoo\<T, U\> where T: covariant, U: contravariant { …

Again, not too bad. The danger here is similar to that of the plus and minus: that no one remembers what “contravariant” and “covariant” mean. This has the benefit at least that you can do a web search on the keywords and get a reasonable explanation.

**Option 4:**

 

interface IFoo\<\[Covariant\] T, \[Contravariant\] U\>  { …

Similar to option 3.

**Option 5:**

 

interface IFoo\<out T, in U\> { …

We are taking a different tack with this syntax. In all the options so far we have been describing how the *user* of the interface may treat the interface with respect to the type system rules for implicit conversions – that is, what are the legal variances on the type parameters. Here we are instead describing this in the language of how the *implementer* of the interface intends to use the type parameters.

I like this one a lot; the down side of this is of course that, as I described a few posts ago, you end up with situations like

 

delegate void Meta\<out T\>(Action\<T\> action);

where the "out" T is clearly used in an input position.

**Option 6:**

Do something else I haven’t thought of. Anyone who has bright ideas, please leave comments.

**Next time:** what problems are introduced by adding this kind of variance?

