# Strange, but legal

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/7/2011 6:51:00 AM

-----

*"Can a property or method really be marked as both abstract and override?"* one of my coworkers just asked me. My initial gut response was "of course not\!" but as it turns out, the Roslyn codebase itself has a property getter marked as both abstract and override. (Which is why they were asking in the first place.)

I thought about it a bit more and reconsidered. This pattern is quite rare, but it is perfectly legal and even sensible. The way it came about in our codebase is that we have a large, very complex type hierarchy used to represent many different concepts in the compiler. Let's call it "Thingy":

 

abstract class Thingy  
{  
  public virtual string Name { get { return ""; } }  
}

There are going to be a lot of subtypes of Thingy, and almost all of them will have an empty string for their name. Or null, or whatever; the point is not what exactly the value is, but rather that there is a sensible default name for almost everything in this enormous type hierarchy.

However, there is another abstract kind of thingy, a FrobThingy, which *always* has a non-empty name. In order to prevent derived classes of FrobThingy from accidentally using the default implementation from the base class, we said:

 

abstract class FrobThingy : Thingy  
{  
  public abstract override string Name { get; }  
}

Now if you make a derived class BigFrobThingy, you know that you have to provide an implementation of Name for it because it will not compile if you don't.

