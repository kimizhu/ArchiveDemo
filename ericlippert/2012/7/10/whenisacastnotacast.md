# When is a cast not a cast?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/10/2012 9:01:32 AM

-----

I'm asked a lot of questions about conversion logic in C\#, which is not that surprising. Conversions are common, and the rules are pretty complicated. Here's some code I was asked about recently; I've stripped it down to its essence for clarity:

class C\<T\> {}  
class D  
{  
  public static C\<U\> M\<U\>(C\<bool\> c)  
  {  
    return **something**;  
  }  
}  
public static class X  
{  
  public static V Cast\<V\>(object obj) { return (V)obj; }  
}

where there are three possible texts for "**something**":

Version 1: (C\<U\>)c  
Version 2: X.Cast\<C\<U\>\>(c);  
Version 3: (C\<U\>)(object)c

Version 1 fails at compile time. Versions 2 and 3 succeed at compile time, and then fail at runtime if U is not bool.

**Question: Why does the first version fail at compile time?**

Because the compiler knows that the only way this conversion could possibly succeed is if U is bool, but U can be anything\! The compiler assumes that most of the time U is not going to be constructed with bool, and therefore this code is almost certainly an error, and the compiler is bringing that fact to your attention.

**Question: Then why does the second version succeed at compile time?**

Because the compiler has no idea that a method named X.Cast\<V\> is going to perform a cast to V\! All the compiler sees is a call to a method that takes an object, and you've given it an object, so the compiler's work is done. The method is a "black box" from the caller's perspective; the compiler does not look inside that box to see whether the mechanisms in that box are likely to fail given the input. This "cast" is not really a cast from the compiler's perspective, it's a method call.

**Question: So what about the third version? Why does it not fail like the first version?**

This one is actually the same thing as the second version; all we've done is inlined the call to X.Cast\<V\>, *including the intermediate conversion to object\!* That conversion is relevant.

**Question: In both the second and third cases, the conversion succeeds at compile time because there is a conversion to object in the middle?**

That's right. The rule is: if there is a conversion from a type S to object, then there is an explicit conversion from object to S. (\*)

By making a conversion to object before doing the "offensive" conversion, you are basically telling the compiler "please throw away the compile-time information you have about the type of the thing I am converting". In the third version we do so explicitly; in the second version we do so sneakily, by making an implicit conversion to object when the argument is converted to the parameter type.

**Question: So this explains why compile-time type checking doesn't seem to work quite right on LINQ expressions?**

Yes\! You would think that the compiler would disallow nonsense like:

from bool b in new int\[\] { 123, 345 } select b.ToString();

because obviously there is no conversion from int to bool, so how can range variable b take on the values in the array? Nevertheless, this succeeds because the compiler translates this to

(new int\[\] { 123, 345 }).Cast\<bool\>().Select(b=\>b.ToString())

and the compiler has no idea that passing a sequence of integers to the extension method Cast\<bool\> is going to fail at runtime. That method is a black box. You and I know that it is going to perform a cast, and that the cast is going to fail, but the compiler does not know that.

And maybe we do not actually know it either; perhaps we are using some library other than the default LINQ-to-objects query provider that *does* know how to make conversions between types that the C\# language would not normally allow. This is actually an extensibility feature masquerading as a compiler deficiency: it's not a bug, it's a feature\!

-----

(\*) You'll note that I did not say "there is an explicit conversion from object to every type", because there isn't. Can you think of a type S that cannot be converted to object?

