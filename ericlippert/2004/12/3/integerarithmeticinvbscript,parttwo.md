# Integer Arithmetic in VBScript, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/3/2004 8:10:00 AM

-----

Here's another recent question I've received on bit twiddling in VBScript:

You discussed the issues with interpreting error results that come back interpreted as signed longs last year. Suppose we have a large unsigned long value, something like E18F4994. VBScript returns this value as -510703212. How can we go from this to the "representation" that a C user would get, the value 3784264084? Or given a string containing that representation, "3784264084", how can a VBScript user work out the hex? Indeed, I did discuss that last year, [here](http://blogs.msdn.com/ericlippert/archive/2003/10/22/53267.aspx). The key to solving the problem in JScript and VBScript is the same. Consider a 32 bit pattern interpreted as a signed integer and an unsigned integer. If the high bit is zero, both interpretations agree**. If the high bit is one then the signed interpretation is equal to the unsigned interpretation minus 2^32. ** Or, stated the other way, the representation of a negative number is determined by subtracting it from 2^32 and then using the representation of that unsigned number. This makes constructing the conversion functions you want pretty easy: Function ReinterpretSignedAsUnsigned(ByVal x)  
  If x \< 0 Then x = x + 2^32  
  ReinterpretSignedAsUnsigned = x  
End Function Function UnsignedDecimalStringToHex(ByVal x)  
  x = CDbl(x)  
  If x \> 2^31 - 1 Then x = x - 2^32  
  UnsignedDecimalStringToHex = Hex(x)  
End Function print \&He18F4994 ' -510703212  
print ReinterpretSignedAsUnsigned(\&hE18F4994) ' 3784264084  
print UnsignedDecimalStringToHex("3784264084") ' "E18F4994" You might wonder why it is that we use such a goofy way to represent negative integers as "if the high bit is set then interpret it as an unsigned integer but subtract 2^32". The "obvious" way to represent negative integers is to declare that the high bit is the "sign bit" and then just have a 31 bit integer. In that system 00000000000000000000000000000111 = \&h00000007 = 7  
10000000000000000000000000000111 = \&h80000007 = -7 very simple and straightforward, right? However, that system has one minor problem, and the system we actually use has a major advantage. The minor problem is that in this system there are two zeros -- a "positive zero" and a "negative zero", which is darn weird. That's a pretty minor problem though -- a problem shared, in fact, by floating point numbers. A 64 bit float consists of a 52 bit unsigned integer, a sign bit, and eleven bits of exponent; if you can twiddle the bits then it's possible to represent +0 and -0 differently in a float, though of course it is silly to do so. (The exact details of how zeros, infinities, nans and denormals work in floating point arithmetic is a subject for another day.) The major advantage of the "subtract off 2^32" representation for negative numbers becomes apparent when you notice this interesting fact: **adding 2^32 to a 32 bit integer is a no-op.** You'd go to add the 33rd bit, and there isn't one there, so nothing happens. Think about that in the context of implementing subtraction. You want to calculate 10 - 3: 10 - 3  
\= 0000000A + (-3)  
\= 0000000A + (-3) + 2^32, since in 32 bit arithmetic, adding 2^32 is a no-op.  
\= 0000000A + (2^32 - 3), but that *is* representable as a 32 bit integer  
\= 0000000A + FFFFFFFFD  
\= 00000007, because we throw away the high bit that doesn't fit into the 32 bit integer Get it? If you represent negative integers this way then you don't have to build another circuit on your chip to handle subtraction. You just build a circuit that handles unsigned integer addition. **Integer subtraction and unsigned integer addition are the same operation at the bit level**.

