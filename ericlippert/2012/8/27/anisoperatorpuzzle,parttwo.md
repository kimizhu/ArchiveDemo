# An "is" operator puzzle, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/27/2012 9:59:56 AM

-----

As I said [last time](http://blogs.msdn.com/b/ericlippert/archive/2012/08/23/an-quot-is-quot-operator-puzzle-part-one.aspx), that was a pretty easy puzzle: either FooBar, or the type of local variable x, can be a type parameter. That is:

void M\<FooBar\>()  
{  
  int x = 0;  
  bool b = x is FooBar;  // legal, true if FooBar is int.  
  FooBar fb = (FooBar)x; // illegal  
}

or

struct FooBar { /\* ... \*/ }  
void M\<X\>()  
{  
  X x = default(X);  
  bool b = x is FooBar; // legal, true if X is FooBar  
  FooBar fb = (FooBar)x; // illegal  
}

This not only illustrates an interesting fact about "is" -- that an "is" expression can result in true even if the corresponding cast would be illegal -- but also an interesting fact about casts. Generally speaking, a cast is allowed if the conversion either is know at compile time to always succeed, or possibly succeed. But in these cases we have a situation where the cast could possibly succeed but is still illegal. What's up with that? There are two main factors that come to mind, based on [the dual nature of casts that I've mentioned before](http://blogs.msdn.com/b/ericlippert/archive/2009/03/19/representation-and-identity.aspx): a cast can mean "*I know that this value is of the given type*, even though the compiler does not know that, the compiler should allow it", and a cast can mean "*I know that this value is not of the given type*; generate special-purpose, type-specific code to convert a value of one type to a value of a different type."

But neither of these things are logical when type parameters are involved. In the context of the first meaning, a cast between a type parameter and a regular type essentially means "I know that the type parameter supplied is of the given type." But in that case, why do you have a type parameter in the first place? It's like having an integer formal parameter and then asserting that it is always twelve. Why did you have the parameter at all if you know ahead of time what the argument will be?

And in the context of the second meaning, in generic code we have no way to generate the special-purpose conversion logic. Let's take our first example code, above. If FooBar is double then we have to generate different code for int-to-double than if FooBar is long. We don't have a cheap and easy way to generate that code, and if user-defined conversions are involved then we need to do overload resolution at runtime. We added that feature to C\# 4; if you want to generate fresh code at runtime that can do arbitrary conversions, use *dynamic*.

**Next time:** we'll explore the question "under what circumstances will the "is" operator give a warning at compile time stating that the "is" is unnecessary?"

