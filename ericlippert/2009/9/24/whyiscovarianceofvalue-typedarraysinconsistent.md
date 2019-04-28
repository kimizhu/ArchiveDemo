# Why is covariance of value-typed arrays inconsistent?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/24/2009 9:53:00 AM

-----

Another interesting [question from StackOverflow](http://stackoverflow.com/questions/1178973/why-does-my-c-array-lose-type-sign-information-when-cast-to-object/1179094#1179094):

uint\[\] foo = new uint\[10\];  
object bar = foo;  
Console.WriteLine("{0} {1} {2} {3}",         
  foo is uint\[\], // True  
  foo is int\[\],  // False  
  bar is uint\[\], // True  
  bar is int\[\]); // True  
  

What the heck is going on here?

This program fragment illustrates an interesting and unfortunate inconsistency between the CLI type system and the C\# type system.

The CLI has the concept of "assignment compatibility". If a value x of known data type S is "assignment compatible" with a particular storage location y of known data type T, then you can store x in y. If not, then doing so is not verifiable code and the verifier will disallow it.

The CLI type system says, for instance, that subtypes of reference type are assignment compatible with supertypes of reference type. If you have a string, you can store it in a variable of type object, because both are reference types and string is a subtype of object. But the opposite is not true; supertypes are not assignment compatible with subtypes. You can't stick something only known to be object into a variable of type string without first casting it.

Basically "assignment compatible" means "*it makes sense to stick these exact bits into this variable*". The assignment from source value to target variable has to be "[representation preserving](http://blogs.msdn.com/ericlippert/archive/2009/03/19/representation-and-identity.aspx)". [](http://blogs.msdn.com/ericlippert/archive/2009/03/19/representation-and-identity.aspx)

One of the rules of the CLI is "*if X is assignment compatible with Y then X\[\] is assignment compatible with Y\[\]*".

That is, arrays are covariant with respect to assignment compatibility. As I've discussed already, [this is actually a broken kind of covariance](http://blogs.msdn.com/ericlippert/archive/2007/10/17/covariance-and-contravariance-in-c-part-two-array-covariance.aspx).

That is *not* a rule of C\#. C\#'s array covariance rule is "*if X is a **reference type** implicitly convertible to **reference type** Y (via a reference or identity conversion) then X\[\] is implicitly convertible to Y\[\]*". That is a subtly different rule\!

In the CLI, uint and int are assignment compatible; therefore uint\[\] and int\[\] are too. But in C\#, the conversion between int and uint is *explicit*, not *implicit*, and these are *value types*, not *reference types*. So in C\# it is *not* legal to convert an int\[\] to a uint\[\]. But it *is* legal in the CLI. So now we are faced with a choice.

1\) Implement "is" so that when the compiler cannot determine the answer statically, it actually calls a method which checks all the C\# rules for identity-preserving convertibility. This is slow, and 99.9% of the time matches what the CLR rules are. But we take the performance hit so as to be 100% compliant with the rules of C\#.

2\) Implement "is" so that when the compiler cannot determine the answer statically, it does the incredibly fast CLR assignment compatibility check, and live with the fact that this says that a uint\[\] is an int\[\], even though that would not actually be legal in C\#.

We chose the latter. It is unfortunate that C\# and the CLI specifications disagree on this minor point but we are willing to live with the inconsistency.

So what's going on here is that in the "foo" cases, the compiler can determine statically what the answer is going to be according to the rules of C\#, and generates code to produce "True" and "False". But in the "bar" case, the compiler no longer knows what exact type is in bar, so it generates code to make the CLR answer the question, and the CLR gives a different opinion.

