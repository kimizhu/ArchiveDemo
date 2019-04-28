# Why Do Initializers Run In The Opposite Order As Constructors? Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/18/2008 10:25:12 AM

-----

As you might have figured out, the answer to last week's puzzle is "if the constructors and initializers run in their actual order then an initialized readonly field of reference type is guaranteed to be non null in any possible call. That guarantee cannot be met if the initializers run in the expected order."

Suppose counterfactually that initializers ran in the expected order, that is, derived class initializers run after the base class constructor body. Consider the following pathological cases:

 

class Base  
{  
    public static ThreadsafeCollection t = new ThreadsafeCollection();  
    public Base()  
    {  
        Console.WriteLine("Base constructor");  
        if (this is Derived) (this as Derived).DoIt();  
        // would deref null if we are constructing an instance of Derived  
        Blah();  
        // would deref null if we are constructing an instance of MoreDerived  
        t.Add(this);  
        // would deref null if another thread calls Base.t.GetLatest().Blah();  
        // before derived constructor runs  
    }  
    public virtual void Blah() { }  
}  
class Derived : Base  
{  
    readonly Foo derivedFoo = new Foo("Derived initializer");  
    public DoIt()  
    {  
        derivedFoo.Bar();  
    }  
}  
class MoreDerived : Derived  
{  
    public override void Blah() { DoIt(); }  
}  
  

Calling methods on derived types from constructors is dirty pool, but it is not illegal. And stuffing not-quite-constructed objects into global state is risky, but not illegal. I'm not recommending that you do any of these things -- please, do not, for the good of us all. I'm saying that it would be really nice if we could give you an ironclad guarantee that an initialized readonly field is always observed in its initialized state, and we cannot make that guarantee unless we run all the initializers first, and then all of the constructor bodies.

Note that of course, if you initialize your readonly fields in the constructor, then all bets are off. We make no guarantees as to the fields not being accessed before the constructor bodies run.

Next time on FAIC: how to get a question *not* answered.

