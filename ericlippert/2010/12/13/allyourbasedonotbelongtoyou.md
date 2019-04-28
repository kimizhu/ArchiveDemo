# All your base do not belong to you

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/13/2010 7:00:00 AM

-----

People sometimes ask me why you can’t do this in C\#:

 

class GrandBase  
{  
  public virtual void M() { Console.WriteLine("GB"); }  
}  
  
class Base : GrandBase  
{  
  public override void M() { Console.WriteLine("B"); }  
}  
  
class Derived : Base  
{  
  public override void M()  
  {  
    Console.WriteLine("D");  
    base.base.M(); // illegal\!  
  }  
}

The author of the most-derived class here wishes to call its GrandBase implementation of M, rather than the Base implementation of M. It wishes to do an "end-run" around its base class.

This is not legal in C\#, and is a bad programming practice. If you find yourself in a position where you want to do an end-run around your base class, odds are good that there is a larger flaw in your type hierarchy that needs to be fixed.

The fact that Base is derived from GrandBase is a “public” part of the surface area of Base. But the fact that Base.M overrides GrandBase.M is not part of that public surface area; by inheriting from GrandBase, Base is promising to provide an implementation of M, but whether that implementation is an override, or merely defers directly to the GrandBase implementation is an *implementation detail* of Base. The Derived class should not know or care how Base fulfills its contract; merely that it does so satisfactorily.

Moreover, when the author of Derived derived it from Base, presumably that developer did so because they liked Base and wanted to re-use its implementation details. If you don’t like the implementation details of Base then *don’t derive from it in the first place*; derive from GrandBase directly.

This is also a bad idea and therefore illegal because it is fragile. Suppose we have a slightly more complex scenario. Let’s add a protected virtual method P to GrandBase. Suppose GrandBase.M calls P, and suppose Base overrides P. If Base.M sets up some state that Base.P depends on, then Derived doing an end-run around Base.M means that GrandBase.M will call Base.P without Base.M setting up the state it needs.

 

class GrandBase  
{  
  public virtual void M() { Console.WriteLine("GB"); P(); }  
  protected virtual void P() { }  
}  
  
class Base : GrandBase  
{  
  public override void M()  
  {  
    Console.WriteLine("B");  
    // We know there is about to be a call to P.  
    SetUpStateForP();  
  }  
  
  protected override void P()  
  {  
     // Use the state set up by M  
     ...

This could have an impact on security or correctness. It is hard enough to design correct, secure, robust implementations of virtual methods; let’s not make it any harder.

Now, I note that **this is only a rule of C\#, not a rule of the CLR**. The CLR *does* allow a language to implement a feature whereby a non-virtual call is done to a virtual method that skips arbitrarily far down the class hierarchy. You cannot rely upon the CLR enforcing this rule of C\#. However, the CLR does not allow non-virtual calls *from a non-derived type*. If you made another type, C, that was not derived from GrandBase then a non-virtual call to GrandBase.M would not be verifiable. Interestingly enough, this rule applies even to nested types; the CLR verifier does not allow a nested type to do a non-virtual call to a virtual method of its container's base class.

