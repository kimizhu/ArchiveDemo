# Inheritance and Representation

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/19/2011 10:18:00 AM

-----

(Note: Not to be confused with [Representation and Identity](http://blogs.msdn.com/b/ericlippert/archive/2009/03/19/representation-and-identity.aspx))

Here's a question I got this morning:

class Alpha\<X\>  
  where X : class  
{}  
class Bravo\<T, U\>  
  where T : class  
  where U : T  
{  
  Alpha\<U\> alpha;  
}

This gives a compilation error stating that U cannot be used as a type argument for Alpha's type parameter X because U is not known to be a reference type. But surely U *is* known to be a reference type because U is constrained to be T, and T is constrained to be a reference type. Is the compiler wrong?

Of course not. Bravo\<object, int\> is perfectly legal and gives a type argument for U which is not a reference type. All the constraint on U says is that U must inherit from T (\*). int inherits from object, so it meets the constraint. All struct types inherit from at least two reference types, and some of them inherit from many more. (Enum types inherit from System.Enum, many struct types implement interface types, and so on.)

The right thing for the developer to do here is of course to add the reference type constraint to U as well.

That easily-solved problem got me thinking a bit more deeply about the issue. I think a lot of people don't have a really solid understanding of what "inheritance" means in C\#. It is really quite simple: a **derived type which inherits from a base type implicitly has all inheritable members of the base type.** That's it\! If a base type has a member M, then a derived type has a member M as well. (\*\*)

People sometimes ask me if *private* members are inherited; surely not\! What would that even mean? But yes, private members *are* inherited, though most of the time it makes no difference because the private member cannot be accessed outside of its accessibility domain. However, if the derived class is *inside* the accessibility domain then it becomes clear that yes, private members are inherited:

class B  
{  
  private int x;  
  private class D : B  
  {

D inherits x from B, and since D is inside the accessibility domain of x, it can use x no problem.

I am occasionally asked "but how can a value type, like int, which is 32 bits of memory, no more, no less, possibly inherit from object?  An object laid out in memory is way bigger than 32 bits; it's got a sync block and a virtual function table and all kinds of stuff in there."  Apparently lots of people think that *inheritance has something to do with how a value is laid out in memory*. But how a value is laid out in memory is an implementation detail, not a contractual obligation of the inheritance relationship\! When we say that int inherits from object, what we mean is that if object has a member -- say, ToString -- then int has that member as well. When you call ToString on something of compile-time type object, the compiler generates code which goes and looks up that method in the object's virtual function table at runtime. When you call ToString on something of compile-time type int, the compiler knows that int is a sealed value type that overrides ToString, and generates code which calls that function directly. And when you **box** an int, then at runtime we do lay out an int the same way that any reference-typed object is laid out in memory.

But there is no requirement that int and object be always laid out the same in memory just because one inherits from the other; all that is required is that there be some way for the compiler to generate code that honours the inheritance relationship.

\------------

(\*) or be identical to T, or possibly to inherit from a type related to T by some variant conversion.

(\*\*) Of course that's not *quite* it; there are some odd corner cases. For example, a class which "inherits" from an interface must have an implementation of every member of that interface, but it could do an explicit interface implementation rather than exposing the interface's members as its own members. This is yet another reason why I'm not thrilled that we chose the word "inherits" over "implements" to describe interface implementations. Also, certain members like destructors and constructors are not inheritable.

