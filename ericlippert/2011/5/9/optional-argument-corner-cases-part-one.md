<div id="page">

# Optional argument corner cases, part one

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/9/2011 7:29:00 AM

-----

<div id="content">

<div class="mine">

(This is part one of a series on the corner cases of optional arguments in C\# 4. Part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/05/12/optional-argument-corner-cases-part-two.aspx).)

In C\# 4.0 we added "optional arguments"; that is, you can state in the declaration of a method's parameter that if certain arguments are omitted, then constants can be substituted for them:

<span class="code"> </span>

void M(int x = 123, int y = 456) { }

can be called as <span class="code">M()</span>, <span class="code">M(0)</span> and <span class="code">M(0, 1)</span>. The first two cases are treated as though you'd said <span class="code">M(123, 456)</span> and <span class="code">M(0, 456)</span> respectively.

This was a controversial feature for the design team, which had resisted adding this feature for almost ten years despite numerous requests for it and the example of similar features in languages like C++ and Visual Basic. Though obviously convenient, the convenience comes at a pretty high price of bizarre corner cases that the compiler team definitely needs to deal with, and customers occasionally run into by accident. I thought I might talk a bit about a few of those bizarre corner cases.

First off, a couple having to do with interfaces. Let's take a look at how optional arguments interact with "implicit" interface implementations.

<span class="code"> </span>

interface IFoo  
{  
  void M(int x = 123);  
}  
class Foo : IFoo  
{  
  public void M(int x){}  
}

Here the method M is implicitly implemented by Foo's method M. What happens here?

<span class="code"> </span>

Foo foo = new Foo();  
foo.M();

An overload resolution error, that's what. Just because Foo's declaration of M happens to implicitly be the implementation of IFoo.M does not mean that the overload resolution algorithm figures that out and takes advantage of it. In fact, things could be quite confusing if it did; suppose we had:

<span class="code"> </span>

interface IFoo  
{  
  void M(int x = 123);  
}  
interface IBar  
{  
  void M(int x = 456);  
}  
class Foo : IFoo, IBar  
{  
  public void M(int x){}  
}

Foo's method M implements both interface methods implicitly; which default parameter value should it pay attention to?

The rule here is straightforward: when considering a method on a class, the optional arguments of the class's implementation are the only ones that are considered to be "in effect" during overload resolution. The optional parameters on the interface implementation only come into effect when calling via the interface:

<span class="code"> </span>

IFoo foo = new Foo();  
foo.M(); // Legal

This also explains why the compiler gives you a stern warning when you do an explicit interface implementation:

<span class="code"> </span>

interface IAbc  
{  
  void M(int x) {}  
}  
class Abc : IAbc  
{  
  void IAbc.M(int x = 123) {}  
}

The compiler warns you that the default value on the parameter is meaningless. The only way to call the method is via the interface, and how does the compiler know when resolving the call to the interface method what implementation to look up the default argument values on? The whole point of interfaces is to decouple the method from the implementation, so there's no way to know\!

**Next time:** some more thoughts on inconsistencies between class and interface optional arguments.

(This is part one of a series on the corner cases of optional arguments in C\# 4. Part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/05/12/optional-argument-corner-cases-part-two.aspx).)

</div>

</div>

</div>

