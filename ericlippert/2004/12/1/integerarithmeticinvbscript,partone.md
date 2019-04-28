# Integer Arithmetic In VBScript, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/1/2004 12:02:00 PM

-----

I've received some questions recently on how integer arithmetic works in VBScript, so I thought I might spend a few entries talking about some low-level bit twiddling topics. Here's one of the mails I got this morning:

There seems to be a limitation on the largest number that the VBScript "mod" operator works against. I was wondering if you could explain this limitation and how to overcome it. I have tried type casting using CDbl or CCur, but that does not seem to overcome the problem. The mod operator, just to refresh your memory, returns the remainder of an integer division. So, 25 mod 7 is 4, because 7 goes into 25 three times leaving a remainder of 4. And indeed, the writer is correct. 2147483647 mod 2 returns 1, but 2147483648 mod 2 produces a type mismatch error. Since that's 2^31, you should be able to guess at what's going on here **-- the** mod operator only operates on values that can be fit into a 32 bit signed integer. Let me describe precisely what we do. The mod operator takes two arguments which of course can both be of any variant type. The type conversions work like this:

  - If either argument is a byref variant then we dereference to get the underlying variant and replace the argument with the result.
  - If either argument is an object with a default property then we fetch the object's default property (if one exists) and replace the argument with the result.
  - If either argument is anything other than

Empty, Null, a 16/32 bit signed integer, a 32/64 bit float, a currency, a date, a string, a Boolean, or an unsigned byte then we raise an error.

If either argument is Null then we return Null.

Otherwise, if either argument is a 32 bit signed integer, a 32/64 bit float, a currency, a date or a string then **both arguments are converted to 32 bit signed integers**. **This may throw a type mismatch error if the conversion cannot be performed.** We compute the 32 bit signed integer which is the modulus, and return.

Otherwise, if either argument is Empty, 16 bit signed integer, or Boolean then we convert both arguments to 16 bit signed integers, compute the 16 bit signed integer which is the modulus, and return.

Otherwise, the only scenario left is the rare but possible case that both arguments are unsigned bytes. We compute the modulus and return an unsigned byte.

(Incidentally, the "integer division" operator in VBScript behaves pretty much exactly the same, as you'd expect given the obvious relationship between the two operators.) VBScript provides modular arithmetic on signed 32/16 bit integers and unsigned bytes, and that's it. If you want to implement a modulus function that operates on currencies or floats or some other data type, you'll have to do the work yourself. How hard it is to write such a beast depends upon the desired range. We give you an operator that works over the range of a 32 bit signed integer, which seems like plenty to me. If you want something that works over the range of, say, a 53 bit signed integer, then this works: Function MyMod(ByVal a, ByVal b)  
    a = Fix(CDbl(a))  
    b = Fix(CDbl(b))  
    MyMod = a - Fix(a/b) \* b  
End Function This works just fine on 2147483648. In fact, it works well right up to 9007199254740990. But try x = MyMod(9007199254740991, 2) and you'll get zero, even though this is obviously an odd number. 64 bit floats stop having integer-level accuracy when they exceed 2^53, so you'll get crazy results if you try. (Currencies are only integer-accurate to about 50 bits, so they just make the situation worse.) If you require integer arithmetic beyond 53 bits -- like, say, you're writing your own RSA implementation for some crazy reason, and need to do modular arithmetic on 1000+ bit integers -- then you'll need to use special techniques. Developing libraries that manipulate arbitrarily large numbers efficiently is definitely character-building, but I'd recommend that you do it in C or C\# or some other hard-typed language designed for bit twiddling, not in VBScript.

