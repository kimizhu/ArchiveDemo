# So many interfaces, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/8/2011 7:51:00 AM

-----

In [my earlier article from April 2011](http://blogs.msdn.com/b/ericlippert/archive/2011/04/04/so-many-interfaces.aspx) on interface implementation I noted that C\# supports a seldom-used feature called "interface re-implementation". This feature is useful when you need it but unfortunately is one of those features that can bite you if you use it incorrectly or accidentally.

Every interface method of every interface you implement in a class or struct has to be "mapped" to a method in the type (either a method directly implemented by the type or a method that the type obtained via inheritance, doesn't matter) that implements the interface method. That's pretty straightforward. The idea of interface re-implementation is basically that if you **re-state on a derived class** that you implement an interface that was **already implemented by a base class**, then when analyzing the derived class, we abandon all the information we had previously computed about the "mappings" in the base class.

This is useful particularly in the case where a base class does an explicit implementation of an interface and you would like to replace its implementation with your own:

interface I  
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
}

Echo does not have a public, protected, or internal virtual method M that Foxtrot can override; if Foxtrot wants to replace Echo's behaviour then its only chance to do so is via interface re-implementation.

Like I said, that can be useful but you should be aware that this is a sharp tool that can cut you in some situations. Here's a situation we found ourselves in recently on the Roslyn team where we managed to cut ourselves with our own tool\!

We have two very similar interfaces, IInternal and IPublic. We intend that our component be used by the public -- you guys -- via the IPublic interface. As an internal implementation detail, we also have an internal interface IInternal that we use to communicate with a particular subsystem that has been provided for us by another team within Microsoft. The methods of IInternal are a superset of the methods of IPublic:

internal interface IInternal  
{  
  void M();  
  void N();  
}  
  
public interface IPublic  
{  
  void M();  
}

public abstract class Bravo : IInternal (\*)  
{  
  // These are our internal workings that we use to communicate  
  // with our private implementation details;  
  // they are explicitly implemented methods of an internal interface,  
  // so they cannot be called by the public.  
  void IInternal.M() { ... }  
  void IInternal.N() { ... }  
}

public abstract class Charlie: Bravo, IPublic  
{  
  // class Charlie, on the other hand, wants to expose M both  
  // as a method of Charlie and as an implicit implementation  
  // of IPublic.M:  
  public void M() { ... }  
}

public sealed class Delta : Charlie, IInternal  
{  
  // Delta is a derived class of Bravo via Charlie; it needs to  
  // change how IInternal.N behaves, so it re-implements the  
  // interface and provides a new, overriding implementation:  
  void IInternal.N() { ... }  
}

This is wrong. There are three methods that Delta must provide: IInternal.M, IInternal.N and IPublic.M. IPublic.M is implemented in Delta by Charlie, as usual. But we are doing an interface re-implementation of IInternal, so we start fresh. What is the implementation of IInternal.N? Obviously the explicitly implemented method in Delta. But what is the implementation of IInternal.M? It's the public method in Charlie, not the invisible-to-everyone explicit method in Bravo. When an instance of Delta is passed off to the internal subsystem it will now call the public method Charlie.M instead of the correct code: the explicit implementation in Bravo.

There is no way for Delta to get to Bravo's implementation of IInternal.M; that is a private implementation detail of Bravo. The interface re-implementation pattern is simply a bad technique to use here; the scenario is too complicated for it to work without you cutting yourself accidentally. A better pattern is to have only one implementation of the internal interface that then defers to an internal-only method:

public abstract class Bravo : IInternal  
{  
  void IInternal.M() { this.InternalM(); }  
  internal virtual void InternalM() { ... }  
  void IInternal.N() { this.InternalN(); }  
  internal virtual void InternalN() { ... }  
}

public abstract class Charlie: Bravo, IPublic  
{  
  public void M() { ... }  
}

public sealed class Delta: Charlie  
{  
  internal override void InternalN() { ... }   
}

And now Delta can even call base.InternalN if it needs to call Bravo.InternalN.

Once more we see that **designing for inheritance is trickier than you might think**.

-----

(\*) People are sometimes surprised that this works. The accessibility domain of a derived class must be a subset of the accessibility domain of its base class; that is, you cannot derive a public class from an internal base class. The same is not true of interfaces; interfaces can be as private as you like. In fact, a public class can implement a private interface nested inside itself\!

