<div id="page">

# Covariance and Contravariance in C\#, Part Eight: Syntax Options

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/31/2007 10:48:00 AM

-----

<div id="content">

<div class="mine">

As I discussed last time, were we to introduce interface and delegate variance in a hypothetical future version of C\# we would need a syntax for it. Here are some possibilities that immediately come to mind.

**Option 1:**

<span class="code"> </span>

interface IFoo\<+T, -U\> { T Foo(U u); }

The CLR uses the convention I have been using so far in this series of “<span class="code">+</span> means covariant, <span class="code">-</span> means contravariant”. Though this does have some mnemonic value (because <span class="code">+</span> means “is compatible with a bigger type”), most people (including members of the C\# design committee\!) have a hard time remembering exactly which is which.

This convention is also used by the Scala programming language.

**Option 2:**

<span class="code"> </span>

interface IFoo\<T:\*, \*:U\> { …

This more graphically indicates “something which is extended by <span class="code">T</span>” and “something which extends <span class="code">U</span>”.  This is similar to Java’s “wildcard types”, where they say “<span class="code">? extends U</span>” or “<span class="code">? super T</span>”.

Though this isn’t terrible, I think it’s a bit of a conflation of the notions of *extension* and *assignment compatibility*. I do not want to imply that <span class="code">IEnumerable\<Animal\></span> is a *base* of <span class="code">IEnumerable\<Giraffe\></span>, even if <span class="code">Animal</span> is a *base* of <span class="code">Giraffe</span>. Rather, I want to say that <span class="code">IEnumerable\<Giraffe\></span> is *convertible to* <span class="code">IEnumerable\<Animal\></span>, or *assignment compatible*, or some such thing. **I don’t want to conceptually overwork the inheritance mechanism.** It's bad enough IMO that we conflate base classes with base interfaces.

**Option 3:**

<span class="code"> </span>

interface IFoo\<T, U\> where T: covariant, U: contravariant { …

Again, not too bad. The danger here is similar to that of the plus and minus: that no one remembers what “contravariant” and “covariant” mean. This has the benefit at least that you can do a web search on the keywords and get a reasonable explanation.

**Option 4:**

<span class="code"> </span>

interface IFoo\<\[Covariant\] T, \[Contravariant\] U\>  { …

Similar to option 3.

**Option 5:**

<span class="code"> </span>

interface IFoo\<out T, in U\> { …

We are taking a different tack with this syntax. In all the options so far we have been describing how the *user* of the interface may treat the interface with respect to the type system rules for implicit conversions – that is, what are the legal variances on the type parameters. Here we are instead describing this in the language of how the *implementer* of the interface intends to use the type parameters.

I like this one a lot; the down side of this is of course that, as I described a few posts ago, you end up with situations like

<span class="code"> </span>

delegate void Meta\<out T\>(Action\<T\> action);

where the "out" <span class="code">T</span> is clearly used in an input position.

**Option 6:**

Do something else I haven’t thought of. Anyone who has bright ideas, please leave comments.

**Next time:** what problems are introduced by adding this kind of variance?

</div>

</div>

</div>

