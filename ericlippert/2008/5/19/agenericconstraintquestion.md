# A Generic Constraint Question

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/19/2008 10:01:00 AM

-----

Here's a question I get fairly frequently: the user desires to create a generic type Bar\<T\> constrained such that T is guaranteed to be a Foo\<U\>, for some U.

The way they usually try to write this is

 

class Bar\<T\>  where T : Foo\<T\> {...

which of course then does not work. The user discovers this when they type

 

 new Bar\<Foo\<string\>\>()

and get an error stating that the constraint has been violated. Which it certainly has. The constraint says that T must be Foo\<T\>, so therefore string must be Foo\<string\>, which clearly it is not.

There are a few ways to solve this problem. One is to simply introduce a new type parameter:

 

 class Bar\<T, U\> where T : Foo\<U\>

this means that you now have some redundancy:

 

new Bar\<Foo\<string\>, string\>()

But let's stop and think for a moment about what the user desired in the first place. Suppose there were a way to specify what we sometimes call a "mumble type" on the C\# design team:

 

class Bar\<T\>  where T : Foo\<?\> {...

Think about the consequences of that on the caller side; the compiler could verify that new Bar\<Foo\<string\>\>() was legal. But what could the compiler verify on the callee side?

 

class Bar\<T\>  where T : Foo\<?\> {  
   public void Blah(T t) {  
    t.

t dot what? We don't know anything about T except that it is Foo\<something\>. What methods, properties, fields or events of Foo could we access? Only those that in no way consume the generic type, which seems like precious little. If Foo had a whole lot of stuff that wasn't generic, why was it made generic in the first place?

But this then leads to an insight; if the user owns the Foo\<T\> class, then they could do precisely that:

 

public abstract class FooBase  
{  
  private FooBase() {} // Not inheritable by anyone else  
  public class Foo\<U\> : FooBase {...generic stuff ...}  
  ... nongeneric stuff ...  
}

public class Bar\<T\> where T: FooBase { ... }  
...  
new Bar\<FooBase.Foo\<string\>\>()

and now everyone is happy. The compiler restricts the construction of Bar to FooBase on the caller side. The compiler ensures that every runtime instance of FooBase is an instance of Foo\<T\>. And the compiler allows the callee side to use all the non-generic methods on FooBase.

Personally, I would go for the first solution myself. But it's edifying to try and find additional solutions, even if they are a bit odd.

