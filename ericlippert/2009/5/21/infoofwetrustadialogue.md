# In Foof We Trust: A Dialogue

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/21/2009 10:54:00 AM

-----

**User**: The typeof(T) operator in C\# essentially means “compiler, generate some code that gives me an object at runtime which represents the type you associate with the name T”. It would be nice to have similar operators that could take names of, say, methods and give you other metadata objects, like method infos, property infos, and so on. This would be a more pleasant syntax than passing ugly strings and types and binding flags to various Reflection APIs to get that information.

**Eric**: I agree, that would be a lovely sugar. This idea has been coming up during the C\# design meeting for almost a decade now. We call the proposed operator “infoof”, since it would return arbitrary metadata info. We whimsically insist that it be pronounced “in-foof”, not “info-of”.

**User**: So implement it already, if you’ve been getting requests for the better part of ten years\! What are you waiting for?

**Eric**: First, as I’ve [discussed before on my blog](http://blogs.msdn.com/ericlippert/archive/2008/10/08/the-future-of-c-part-one.aspx), we have a very limited time and money budget for design, implementation, testing, documentation and maintenance, so we want to make sure we’re spending it on the highest-value features. This one has never made it over that bar.

Second, the feature seems at first glance to be simple, but in fact has a rather large “design surface” and would require adding a fair amount of new syntax to the language.

Third, it is nothing that you cannot do already with existing code; it’s not *necessary*, it’s just *nice*.

All these are enough “points against” the feature that it’s never made it into [positive territory](http://blogs.msdn.com/ericgu/archive/2004/01/12/57985.aspx). It will stay on the list for hypothetical future versions of the language, but I would not expect it to ever happen.

**User**: Can you elaborate on some of the problems?

**Eric**: Let’s start with a simple one. Suppose you have infoof(Bar) where Bar is an overloaded method. Suppose there is a specific Bar that you want to get info for. This is an *overload resolution* problem. How do you tell the overload resolution algorithm which one to pick? The existing overload resolution algorithm requires either argument expressions or, at the very least, argument types.

Expressions seem problematic. Either they are evaluated unnecessarily, perhaps producing unwanted side effects, or you have a bunch of expressions in code that looks like a function call but isn’t actually. Either way is potentially confusing. So we’d want to stick with types.

**User**: Well then, it ought to be straightforward then. infoof(Bar(int, int)), done.

**Eric**: I notice that you’ve just introduced new syntax; nowhere in C\# previously did we have a parenthesized, comma-separated list of types. But that’s not at all hard to parse.

**User**: Exactly. So go with that.

**Eric**: OK smart guy, what about [this](http://blogs.msdn.com/ericlippert/archive/2006/04/05/569085.aspx) [strangely familiar case](http://blogs.gotdotnet.com/ericlippert/archive/2006/04/06/odious-ambiguous-overloads-part-two.aspx)?

 

class C\<T\>  
{  
  public void Bar(int x, T y) {}  
  public void Bar(T x, int y) {}  
  public void Bar(int x, int y) {}  
}  
class D : C\<int\>  
{ …  

You have some code in D that says infoof(Bar(int, int)). There are three candidates. Which do you pick?

**User**: Well, which would overload resolution pick if you called Bar(10, 10) ? Overload resolution tiebreaker rules say to pick the non-generic version when faced with this awful situation.

**Eric**: Great. Except that’s not the one I wanted. I wanted the method info of the second version. If your proposed syntax picks one of them then you need to give me a syntax to pick the others.

**User**: Hmm. It's hard because overload resolution actually gives you no way to force the call to go to one of the two “generic param” methods in this bizarre case, so some mechanism *stronger* than overload resolution needs to be invented if you want the feature to allow getting info of *any* method.

But this is a deliberately contrived corner case; just cut it. You don’t have to support this scenario.

**Eric**: And now you start to see how this always goes in the design meeting\! It is very easy to design a feature that hits what initially looks like 80% of the cases. And then you discover that the *other 80%* of the cases are not covered, and the feature either takes 160% of its budget, or you end up with a confusing, inconsistent and weak feature, or, worse, both.

And how do we know what scenarios to cut? We have to design a feature that does *something* in *every possible case*, even if that something is an error. So we have to at least spend the time to consider every possible case during the design. There are a lot of weird cases to consider\!

**User**: What are some of the other weird cases?

**Eric**: Just off the top of my head, here are a few. (1) How do you unambiguously specify that you want a method info of an specific explicit interface implementation? (2) What if overload resolution would have skipped a particular method because it is not accessible? It is legal to get method infos of methods that are not accessible; metadata is always public even if it describes private details. Should we make it impossible to get private metadata, making the feature weak, or should we make it possible, and make infoof use a subtly different overload resolution algorithm than the rest of C\#?  (3) How do you specify that you want the info of, say, an indexer setter, or a property getter, or an event handler adder?

There are lots more goofy edge cases.

**User**: I’m sure that we could come up with a syntax for all of those cases.

**Eric**: Sure. None of these problems are unsolvable. But they almost all require careful design and new syntax, and we would soon end up spending most of our design, implementation and testing budget on this trivial “sugar” feature, budget that we could be spending on more valuable features that solve real problems.

**User**: Well, how about you do the easy 80% and make “the other 80%” into error cases? Sure, the smaller feature is weaker and might be inconsistent, but something is better than nothing, and it’ll be cheaper to just do the easy ones.

**Eric**: That doesn’t actually make it cheaper a lot of the time. Any time we make a corner case into an error we need to carefully specify exactly what the error case is so that we can implement it correctly and test it confidently. Every error case still spends design, implementation and test budget that could have been spent elsewhere.

We would need to come up with a sensible error message, localize the error message into I don’t know how many languages, implement the code that detects the weird scenarios and gives the right error, test the error cases and document them. And we would need to deal with all the users who push back on each error case and request that it work for their specific scenario.

Trying to scope the feature down so that it is smaller *adds* work, and it is possible that it adds more work in the long run than simply making the larger version of the feature work in the first place. [There are no cheap features](http://blogs.msdn.com/ericlippert/archive/2003/10/28/53298.aspx).

**User**: Bummer. Because most of the time I really just want to get the method info of the method I’m currently running, for logging purposes during my test suites. It seems a shame to have to solve all these overload resolution problems just to get information about what’s happening at runtime.

**Eric**: Maybe you should have said that in the first place\! The aptly named [GetCurrentMethod](http://msdn.microsoft.com/en-us/library/system.reflection.methodbase.getcurrentmethod.aspx) method does that.

**User**: So what you’re telling me is no infoof?

**Eric**: It’s an awesome feature that pretty much everyone involved in the design process wishes we could do, but there are good practical reasons why we choose not to. If there comes a day when designing it and implementing it is the best way we could spend our limited budget, we’ll do it. Until then, use Reflection.

