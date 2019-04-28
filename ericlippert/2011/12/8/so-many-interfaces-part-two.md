<div id="page">

# So many interfaces, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/8/2011 7:51:00 AM

-----

<div id="content">

<div class="mine">

In [my earlier article from April 2011](http://blogs.msdn.com/b/ericlippert/archive/2011/04/04/so-many-interfaces.aspx) on interface implementation I noted that C\# supports a seldom-used feature called "interface re-implementation". This feature is useful when you need it but unfortunately is one of those features that can bite you if you use it incorrectly or accidentally.

Every interface method of every interface you implement in a class or struct has to be "mapped" to a method in the type (either a method directly implemented by the type or a method that the type obtained via inheritance, doesn't matter) that implements the interface method. That's pretty straightforward. The idea of interface re-implementation is basically that if you **re-state on a derived class** that you implement an interface that was **already implemented by a base class**, then when analyzing the derived class, we abandon all the information we had previously computed about the "mappings" in the base class.

This is useful particularly in the case where a base class does an explicit implementation of an interface and you would like to replace its implementation with your own:

<span class="code">interface I  
{  
  void M();  
}  
class Echo : I  
{  
  void I.M() { ... }  
}  
class Foxtrot : Echo, I  
{  
  void I.M() { ... }  
}</span>

<span class="code">Echo</span> does not have a public, protected, or internal virtual method M that <span class="code">Foxtrot</span> can override; if <span class="code">Foxtrot</span> wants to replace <span class="code">Echo</span>'s behaviour then its only chance to do so is via interface re-implementation.

Like I said, that can be useful but you should be aware that this is a sharp tool that can cut you in some situations. Here's a situation we found ourselves in recently on the Roslyn team where we managed to cut ourselves with our own tool\!

We have two very similar interfaces, <span class="code">IInternal</span> and <span class="code">IPublic</span>. We intend that our component be used by the public -- you guys -- via the <span class="code">IPublic</span> interface. As an internal implementation detail, we also have an internal interface <span class="code">IInternal</span> that we use to communicate with a particular subsystem that has been provided for us by another team within Microsoft. The methods of <span class="code">IInternal</span> are a superset of the methods of <span class="code">IPublic</span>:

<span class="code">internal interface IInternal  
{  
  void M();  
  void N();  
}  
  
public interface IPublic  
{  
  void M();  
}</span>

<span class="code">public abstract class Bravo : IInternal (\*)  
{  
  // These are our internal workings that we use to communicate  
  // with our private implementation details;  
  // they are explicitly implemented methods of an internal interface,  
  // so they cannot be called by the public.  
  void IInternal.M() { ... }  
  void IInternal.N() { ... }  
}</span>

public abstract class Charlie: Bravo, IPublic  
{  
  // class Charlie, on the other hand, wants to expose M both  
  // as a method of Charlie and as an implicit implementation  
  // of IPublic.M:  
  public void M() { ... }  
}

<span class="code">public sealed class Delta : Charlie, IInternal  
{  
  // Delta is a derived class of Bravo via Charlie; it needs to  
  // change how IInternal.N behaves, so it re-implements the  
  // interface and provides a new, overriding implementation:  
  void IInternal.N() { ... }  
}</span>

This is wrong. There are three methods that <span class="code">Delta</span> must provide: <span class="code">IInternal.M</span>, <span class="code">IInternal.N</span> and <span class="code">IPublic.M</span>. <span class="code">IPublic.M</span> is implemented in <span class="code">Delta</span> by <span class="code">Charlie</span>, as usual. But we are doing an interface re-implementation of <span class="code">IInternal</span>, so we start fresh. What is the implementation of <span class="code">IInternal.N</span>? Obviously the explicitly implemented method in Delta. But what is the implementation of <span class="code">IInternal.M</span>? It's the public method in <span class="code">Charlie</span>, not the invisible-to-everyone explicit method in <span class="code">Bravo</span>. When an instance of <span class="code">Delta</span> is passed off to the internal subsystem it will now call the public method <span class="code">Charlie.M</span> instead of the correct code: the explicit implementation in <span class="code">Bravo</span>.

There is no way for <span class="code">Delta</span> to get to <span class="code">Bravo</span>'s implementation of <span class="code">IInternal.M</span>; that is a private implementation detail of <span class="code">Bravo</span>. The interface re-implementation pattern is simply a bad technique to use here; the scenario is too complicated for it to work without you cutting yourself accidentally. A better pattern is to have only one implementation of the internal interface that then defers to an internal-only method:

<span class="code">public abstract class Bravo : IInternal  
{  
  void IInternal.M() { this.InternalM(); }  
  internal virtual void InternalM() { ... }  
  void IInternal.N() { this.InternalN(); }  
  internal virtual void InternalN() { ... }  
}</span>

public abstract class Charlie: Bravo, IPublic  
{  
  public void M() { ... }  
}

<span class="code">public sealed class Delta: Charlie  
{  
  internal override void InternalN() { ... }   
}</span>

And now <span class="code">Delta</span> can even call <span class="code">base.InternalN</span> if it needs to call <span class="code">Bravo.InternalN</span>.

Once more we see that **designing for inheritance is trickier than you might think**.

-----

(\*) People are sometimes surprised that this works. The accessibility domain of a derived class must be a subset of the accessibility domain of its base class; that is, you cannot derive a public class from an internal base class. The same is not true of interfaces; interfaces can be as private as you like. In fact, a public class can implement a private interface nested inside itself\!

</div>

</div>

</div>

