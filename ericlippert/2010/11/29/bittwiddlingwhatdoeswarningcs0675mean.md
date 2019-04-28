# Bit twiddling: What does warning CS0675 mean?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/29/2010 10:16:30 AM

-----

From the sublime level of continuation passing style we go back to the mundane level of twiddling individual bits.

 

int i = SomeBagOfBits();  
ulong u = SomeOtherBagOfBits();  
ulong result = u | i; // combine them together

Whoops, that's an error. **"Operator | cannot be applied to operands of type int and ulong."** There are bitwise-or operators defined on int, uint, long and ulong, but none between int and ulong. You cannot use the int version because the ulong might not fit, and you cannot use the ulong version because the int might be negative.

I demand that the compiler do my bidding regardless\!

 

ulong result = u | (ulong) i;

There, now the compiler *has* to choose the ulong operator, and the explicit conversion from int to ulong we know never fails.

Argh, now we've got a warning\! **"CS0675: Bitwise-or operator used on a sign-extended operand; consider casting to a smaller unsigned type first."**

I am [often asked](http://stackoverflow.com/questions/4058960/how-to-supress-c-warning-cs0675-bitwise-or-operator-used-on-a-sign-extended-op/4059174#4059174) what the meaning of this warning is. The crux of the matter is that **the conversion from int to ulong does sign extension**. Let's make a more concrete example:

 

int i = -17973521; // In hex that is FEEDBEEF  
ulong u = 0x0123456700000000;  
ulong result = u | (ulong)i;  
Console.WriteLine(result.ToString("x"));

What is the expected result? Most people expect that the result is 1234567FEEDBEEF. It is not. It is FFFFFFFFFEEDBEEF. Why? Because when converting an int to a ulong, first **the int is converted to a long so that the sign information is not lost**. The long -17973521 is in hex FFFFFFFFFEEDBEEF. *That* long is then converted to that ulong, which is then or'd in the natural way to produce the unexpected result.

The compiler warns you in this case because this pattern is almost always wrong. To eliminate the warning, first decide whether you *want* the sign extension or not. If you do not want it, then **follow the advice given in the warning.** The warning text says "consider casting to a smaller unsigned type" for a reason\!

 

ulong result = u | (uint)i;

This tells the compiler "first convert the int to a uint then convert the uint to a ulong". Since all the math is now done in unsigned types, there is no sign to extend.

If for some strange reason what you want is the sign extension semantics then tell the compiler that explicitly:

 

ulong result = u | (ulong)(long)i;

And add a comment pointing out why you are doing this crazy thing, please.

However, better to not get into this bizarre situation altogether. First off, if you can avoid bit twiddling, do so. I very rarely have to twiddle a bit in a "primitive" integral type these days. Use enums with the Flags attribute if you want to represent a set of bit flags in a compact space. Second, if you must use primitive types then don't be doing bitwise operations on signed values; that is almost never the right thing to do. Third, try to avoid doing bitwise operations on operands of different bit size; that is also waving a red flag.

