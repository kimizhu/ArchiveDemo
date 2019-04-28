# Practice thinking like a compiler tester

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/10/2007 3:03:00 PM

-----

I don’t know why but for some reason I find this little recursive algorithm I ran across in the compiler the other day to be completely hilarious.

For every class in your program we have to emit metadata. For various historical reasons, there are strict rules we must follow about the order in which metadata is emitted:

  - Every class must be emitted exactly once.  (We cannot omit emitting any class and we cannot emit the same class twice.)
  - A derived class must be emitted after its base class.
  - A nested inner class must be emitted after its outer class.

(To simplify this discussion I am omitting interfaces.  A class must also be emitted after all of its interfaces, but for the purpose of this discussion they are just like having multiple base classes.)

We have a recursive algorithm that goes something (but not quite\!) like this:

 

foreach(Class c in TopLevelClasses)  
    Emit(c);  
…  
void Emit(Class c)  
{  
    if (c == null || c.Emitted) return; // don’t emit twice\!  
    Emit(c.BaseClass);  
    Emit(c.OuterClass);  
    // OK, now we are ready to really emit c,  
    // because its base and outer classes are really emitted.  
    ReallyEmit(c); // This must only happen once per class, so mark us as emitted.  
    c.Emitted = true;  
    // Recurse on c’s inner classes, so that we don’t miss them.  
    foreach(Class i in c.InnerClasses)  
        Emit(i);  
}

Does that look correct to you?

Note that at this point we have already guaranteed that base classes form a tree.  You cannot start going up the base class chain and end up back where you started.  So there is no infinite recursive descent along the base class.  Similarly, we already know that outer/inner classes make a well-formed tree.  We’re not going to go up the chain of outer classes and end up back where we started, so there is no infinite recursive descent there either.

We are also not concerned about generic classes, whether the classes in the generic type constraints have been emitted, etc.  Assume C\# 1.0 classes.

Nevertheless, there is still a bug in this algorithm.  Can you find a set of legal C\# classes which causes this algorithm to violate one of the preconditions I mentioned?

This one is pretty easy. I’ll post an example tomorrow, propose a fix to the algorithm, and pose a slightly harder version of the puzzle then.

