# Even More Conversion Trivia, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/1/2007 10:24:00 AM

-----

Reader Barry Kelly came up with the solution I was thinking of for my trivia question yesterday. If the value in question is, say, a nullable integer which has no value then casting it explicitly to object results in the null object. Calling a virtual method on a null object results in an exception at runtime. However, using it implicitly as an object boxes up the nullable int as a valid ValueType, which overrides ToString. So that's the difference: the one without the cast produces an empty string, the one with the cast throws an exception.

Honourable mention to reader Bill Wagner, who points out that if the object implements an interface with a ToString method then there are two ToString slots, one from the base class and one from the interface. Casting to object can cause overload resolution to pick the base version over the interface version, which might otherwise be preferred. (This reminds me of another fun fact about C\# overload resolution which I will blog about at a later date.)

As Barry Kelly points out, the CLR took a design change with Visual Studio 2005 Beta 2 to implement the special case that a nullable without a value becomes a null ValueType rather than a boxed ValueType when boxed. How then do we implement the desired behaviour, ie, that calling ToString without a cast creates a boxed ValueType regardless of whether the nullable has a value?

Fortunately the CLR implemented a special kind of virtual call, called the "constrained" virtual call. Without the ability to constrain the transformation of a value to a particular type when making a virtual call, certain operations become difficult to codegen. (In particular, it is hard to do virtual calls on type parameters of generic types, because you might not know whether the type parameter is going to be a value type or a reference type under construction.) If you take a look at the IL we generate for something like foo.ToString() on a nullable int, it looks like:

 

ldsflda    valuetype \[mscorlib\]System.Nullable\`1\<int32\> Test::foo  
constrained. valuetype \[mscorlib\]System.Nullable\`1\<int32\>  
callvirt   instance string \[mscorlib\]System.Object::ToString()

Which tells the CLR to box up that thing as is, rather than doing a boxing operation. Had we inserted a cast in there we would have generated

 

ldsfld     valuetype \[mscorlib\]System.Nullable\`1\<int32\> Test::foo  
box        valuetype \[mscorlib\]System.Nullable\`1\<int32\>  
callvirt   instance string \[mscorlib\]System.Object::ToString()

instead.

