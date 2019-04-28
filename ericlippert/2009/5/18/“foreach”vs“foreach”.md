# “foreach” vs “ForEach”

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/18/2009 10:13:00 AM

-----

A number of people have asked me why there is no Microsoft-provided “ForEach” sequence operator extension method. The List\<T\> class has such a method already of course, but there’s no reason why such a method could not be created as an extension method for all sequences. It’s practically a one-liner:

public static void ForEach\<T\>(this IEnumerable\<T\> sequence, Action\<T\> action)  
{  
  // argument null checking omitted  
  foreach(T item in sequence) action(item);  
}

My usual response to “why is feature X not implemented?” is that of course [all features are unimplemented](http://blog.ryjones.org/2005/07/12/product-development/) until someone designs, implements, tests, documents and ships the feature, and no one has yet spent the money to do so. And yes, though I have famously pointed out that [even small features can have large costs](http://blogs.msdn.com/ericlippert/archive/2003/10/28/53298.aspx), this one really *is* dead easy, obviously correct, easy to test, and easy to document. Cost is always a factor of course, but the costs for this one really are quite small.

Of course, that cuts the other way too. If it is so cheap and easy, then you can do it yourself if you need it. And really what matters is not the *low cost*, but rather the *net benefit*. As we’ll see, I think the benefits are also very small, and therefore the net benefit might in fact be negative. But we can go a bit deeper here. I am *philosophically* opposed to providing such a method, for two reasons.

The first reason is that doing so violates the functional programming principles that all the other sequence operators are based upon. Clearly **the sole purpose of a call to this method is to cause side effects.** The purpose of an expression is to compute a value, not to cause a side effect. The purpose of a statement is to cause a side effect. The call site of this thing would look an awful lot like an expression (though, admittedly, since the method is void-returning, the expression could only be used in a “statement expression” context.) It does not sit well with me to make the one and only sequence operator that is only useful for its side effects.

The second reason is that doing so adds zero new representational power to the language. Doing this lets you rewrite this perfectly clear code:

foreach(Foo foo in foos){ statement involving foo; }

into this code:

foos.ForEach((Foo foo)=\>{ statement involving foo; });

which uses almost exactly the same characters in slightly different order. And yet the second version is harder to understand, harder to debug, and introduces closure semantics, thereby potentially changing object lifetimes in subtle ways.

When we provide two subtly different ways to do exactly the same thing, we produce confusion in the industry, we make it harder for people to read each other’s code, and so on. Sometimes the benefit added by having two different textual representations for one operation (like query expressions versus their underlying method call form, or + versus String.Concat) is so huge that it’s worth the potential confusion. But the compelling benefit of query expressions is their readability; this new form of “foreach” is certainly no more readable than the “normal” form and is arguably worse.

If you don’t agree with these philosophical objections and find practical value in this pattern, by all means, go ahead and write this trivial one-liner yourself.

UPDATE: [The method in question has been removed from the "Metro" version of the base class library.](http://social.msdn.microsoft.com/Forums/en-US/winappswithcsharp/thread/758f7b98-e3ce-41e5-82a2-109f1df446c2)

