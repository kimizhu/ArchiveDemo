# Future Breaking Changes, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/30/2007 10:00:00 AM

-----

We on the C\# team hate making breaking changes.

As my colleague Neal called out in [his article on the subject from a couple years ago](http://blogs.msdn.com/nealho/archive/2005/11/22/496101.aspx), by “breaking change” we mean a change in the compiler behaviour which causes an existing program to either stop compiling entirely, or start behaving differently when recompiled. We hate them because they can cause intense customer pain, and are therefore barriers to upgrading. We work hard on improving the quality of the C\# toolset every day and want our customers to gain the benefits of upgrading; they cannot do that if doing so causes more pain than the new features are worth.

Which is not to say that we do not make breaking changes occasionally. We do. But in those circumstances where we have to, we try to mitigate the pain as much as possible. For example:

  - We try to ensure that breaking changes only affect unusual corners of the language which real production code is unlikely to hit.
  - When we do cause breaking changes, we sometimes introduce heuristics and warnings which detect the situation and warn the developer.
  - If possible, breaking changes should move the implementation into compliance with the published standard. (Often breaking changes are a result of fixing a compliance bug.)
  - We try to communicate the reasoning behind a breaking change crisply and succinctly.
  - And so on.

I could write a whole series of blog articles about specific breaking changes – and in fact, I have done so many times over the years. (For example, [here](http://blogs.msdn.com/ericlippert/archive/2004/06/10/152831.aspx), [here](http://blogs.msdn.com/ericlippert/archive/2005/10/19/482796.aspx), [here](http://blogs.msdn.com/ericlippert/archive/2006/04/06/570126.aspx), [here](http://blogs.msdn.com/ericlippert/archive/2006/05/24/606278.aspx) and [here](http://blogs.msdn.com/ericlippert/archive/2007/04/16/chained-user-defined-explicit-conversions-in-c.aspx).)

Given that we hate breaking changes, clearly we want to ensure that *as we add new features to the C\# language we do not break existing features*. [New features start out with points against them and have to justify their benefits](http://blogs.msdn.com/ericgu/archive/2004/01/12/57985.aspx). If the feature is a breaking change, that is hugely more points against it.

For example, adding generics to C\# 2.0 was a breaking change. This program, legal in C\# 1.0, no longer compiles:

 

class Program  {  
    static void M(bool x, bool y) {}  
    static void Main() {  
        int N = 1, T = 2, U = 3, a = 4, b = 5;  
        M(N \< T, U \> (a+b));  
    }  
}

because we now think that N is a generic method of one argument. But the compelling benefit of generics greatly outweighed the pain of this rather contrived example, so we took the breaking change. (And [the error message now produced diagnoses the problem](http://blogs.msdn.com/ericlippert/archive/2006/07/07/659259.aspx).)

But what I want to talk about in this set of articles is something a bit more subtle than these specific breaking changes.

Given that we hate breaking changes, we want to ensure that as we add new features to the C\# language *we are not setting ourselves up to have to make breaking changes in the future*. If we are, then the feature needs to be justified against not only any breaking changes it is *presently* introducing, but also against the *potential* for breaking changes in the future. *We do not want to introduce a new feature that makes it harder for us to introduce entirely different features in the future unless the proposed new feature is really great.*

[Implicitly typed lambdas](http://blogs.msdn.com/ericlippert/archive/2007/01/10/lambda-expressions-vs-anonymous-methods-part-one.aspx) are an example of a feature which will cause us innumerable breaking change headaches in the future, but we believe that the compelling user benefit of them is so high (and the breaks are sufficiently isolated to corner cases) that we are willing to take that pain.

Next time on FAIC, I’ll describe in detail how it is that implicitly typed lambdas are going to make it harder to get new features into the C\# type system because of potential breaking changes.

