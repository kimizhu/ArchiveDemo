# Optional argument corner cases, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/12/2011 9:29:35 AM

-----

(This is part two of a series on the corner cases of optional arguments in C\# 4. Part one is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/05/09/optional-argument-corner-cases-part-one.aspx). Part three is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/05/16/optional-argument-corner-cases-part-three.aspx). This portion of the series was inspired by [this StackOverflow question](http://stackoverflow.com/questions/4922714/why-are-c-4-optional-parameters-defined-on-interface-not-enforced-on-implementin/4923642#4923642).)

Last time we saw that the declared optional arguments of an interface method need not be optional arguments of an implementing class method. That seems potentially confusing; why not require that an implementing method on a class exactly repeat the optional arguments of the declaration?

Because the cure is worse than the disease, that's why.

First off, we saw last time that one method on a class could implement two methods of two different interfaces:

 

interface IABC  
{  
  void M(bool x = true);  
}  
  
interface IXYZ  
{  
  void M(bool x = false);  
}

class C : IABC, IXYZ  
{  
  public void M(bool x) {}  
}

If we require the implementation to repeat the default values then the methods cannot be implicitly implemented; at least one will have to be explicitly implemented.

However, that's a rare case and we can probably dismiss it as unlikely. There are other problems though.

Suppose you have an interface in your code:

 

public interface IFoo  
{  
  void M(int x, string y, bool z);  
}

and a hundred different classes that implement it. Then you decide that you want to say

 

public interface IFoo  
{  
  void M(int x, string y, bool z = false);  
}

Do you really want to have to change a hundred different declarations of the implementing method? That seems like a lot of burden to put on the developer, but we could do it.

Suppose we did. Now suppose that interface IFoo is not defined in your source code, but rather is provided to you by a third party. The third party makes a new version of their interface that has a default value for the parameter z. Now when all of their thousands of customers recompile, all of those thousands of customers have to update their source code to match the default parameter\! Requiring this redundancy causes the introduction of a default value to become a potentially large compilation-breaking change.

That's bad. But wait, it gets worse. Suppose you have this situation. IFoo is provided by third party FooCorp.  Base class Bar is provided by third party BarCorp:

 

public class Bar  
{  
  public void M(int x, string y, bool z) { ... }  
}

Note that Bar does not implement IFoo. You wish to use both FooCorp and BarCorp code in your assembly, where you say:

 

class Mine : Bar, IFoo  
{  
}

(The compiler allows this because M on Bar is a member of Mine, and therefore Mine implicitly implements IFoo.M.)

Now FooCorp ships a new version of their assembly with the default value on z. You recompile Mine and it tells you that no, you can't do that *because Bar doesn't have a matching default value for z*. But you didn't write Bar\! What are you supposed to do, call up BarCorp and ask them to ship you a new assembly just because FooCorp -- their competitor -- added a default value to a formal parameter of an interface? What are you supposed to do if BarCorp refuses? The method isn't even virtual, so you can't override it. The solution is ugly:

 

class Mine : Bar, IFoo  
{  
  public new void M(int x, string y, bool z = false)  
  {  
    base.M(x, y, z);  
  }  
}

Best to simply not require the redundancy in the first place.

**Next time:** making an argument optional does not change the signature of the method

(This is part two of a series on the corner cases of optional arguments in C\# 4. Part one is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/05/09/optional-argument-corner-cases-part-one.aspx). Part three is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/05/16/optional-argument-corner-cases-part-three.aspx).)

