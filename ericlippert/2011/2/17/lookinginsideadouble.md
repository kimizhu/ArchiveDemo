# Looking inside a double

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/17/2011 6:36:00 AM

-----

Occasionally when I'm debugging the compiler or responding to a user question I'll need to quickly take apart the bits of a double-precision floating point number. Doing so is a bit of a pain, so I've whipped up some quick code that takes a double and tells you all the salient facts about it. I present it here, should you have any use for it yourself. (Note that this code was built for comfort, not speed; it is more than fast enough for my purposes so I've spent zero time optimizing it.)

To understand the format of a double and why it is the way it is, see [my earlier articles on the subject](http://blogs.msdn.com/b/ericlippert/archive/tags/floating+point+arithmetic/).

This code uses the **Rational** class from the **Microsoft Solver Foundation**; [you can download the code from here](http://code.msdn.microsoft.com/solverfoundation) if you haven't got it already. There are some handy tools in there\! In this particular case I needed a type that could represent a rational number of arbitrarily high precision. Building your own naive implementation of rationals is very straightforward, particularly if you already have a BigInteger type to be the numerator and denominator. But why re-invent the wheel?

The action of the code below is extremely straightforward. I make a struct that turns a 64 bit double into a 64 bit unsigned integer, since it is easier to get the bits out of an integer. I then build a bunch of tiny little extension methods that make it easier to work with bits, with rationals, and so on, so that the mainline code does not become an awful mess. I hate seeing bit twiddling in mainline code, you know what I mean?

-----

 

using System;  
using System.Collections.Generic;  
using System.Text;  
using Microsoft.SolverFoundation.Common;

class Program  
{  
    static void Main()  
    {  
        double original = -256.325;  
        MyDouble d = original;

        Console.WriteLine("Raw sign: {0}", d.Sign);  
        Console.WriteLine("Raw exponent: {0}", d.ExponentBits.Join());  
        Console.WriteLine("Raw mantissa: {0}", d.MantissaBits.Join());

        var signchar = d.Sign == 0 ? '+' : '-';

        if (d.Exponent == 0 && d.Mantissa == 0)  
        {  
            Console.WriteLine("Zero: {0}0", signchar);  
            return;  
        }  
        else if (d.Exponent == 0x7ff && d.Mantissa == 0)  
        {  
            Console.WriteLine("Infinity: {0}Infinity", signchar);  
            return;  
        }  
        else if (d.Exponent == 0x7ff)  
        {  
            Console.WriteLine("NaN");  
            return;  
        }

        bool subnormal = d.Exponent == 0;  
        var two = (Rational)2;  
        var fraction = subnormal ? Rational.Zero : Rational.One;  
        var adjust = subnormal ? 1 : 0;  
        for (int bit = 51; bit \>= 0; --bit)  
            fraction += d.Mantissa.Bit(bit) \* two.Exp(bit - 52 + adjust);  
        fraction = fraction \* two.Exp(d.Exponent - 1023);  
        if (d.Sign == 1)  
            fraction = -fraction;

        Console.WriteLine(subnormal ? "Subnormal" : "Normal");  
        Console.WriteLine("Sign: {0}", signchar);  
        Console.WriteLine("Exponent: {0}", d.Exponent - 1023);  
        Console.WriteLine("Exact binary fraction: {0}.{1}", subnormal ? 0 : 1, d.MantissaBits.Join());  
        Console.WriteLine("Nearest approximate decimal: {0}", original);  
        Console.WriteLine("Exact rational fraction: {0}", fraction.ToString());  
        Console.WriteLine("Exact decimal fraction: {0}", fraction.ToDecimalString());  
    }  
}

struct MyDouble  
{  
    private ulong bits;  
    public MyDouble(double d)  
    {  
        this.bits = (ulong)BitConverter.DoubleToInt64Bits(d);  
    }

    public int Sign  
    {  
        get  
        {  
            return this.bits.Bit(63);  
        }  
    }

    public int Exponent  
    {  
        get  
        {  
            return (int)this.bits.Bits(62, 52);  
        }  
    }

    public IEnumerable\<int\> ExponentBits  
    {  
        get  
        {  
            return this.bits.BitSeq(62, 52);  
        }  
    }

    public ulong Mantissa  
    {  
        get  
        {  
            return this.bits.Bits(51, 0);  
        }  
    }

    public IEnumerable\<int\> MantissaBits  
    {  
        get  
        {  
            return this.bits.BitSeq(51, 0);  
        }  
    }

    public static implicit operator MyDouble(double d)  
    {  
        return new MyDouble(d);  
    }  
}

static class Extensions  
{  
    public static int Bit(this ulong x, int bit)  
    {  
        return (int)((x \>\> bit) & 0x01);  
    }

    public static ulong Bits(this ulong x, int high, int low)  
    {  
        x \<\<= (63 - high);  
        x \>\>= (low + 63 - high);  
        return x;  
    }

    public static IEnumerable\<int\> BitSeq(this ulong x, int high, int low)  
    {  
        for(int bit = high; bit \>= low; --bit)  
            yield return x.Bit(bit);  
    }

    public static Rational Exp(this Rational x, int y)  
    {  
        Rational result;  
        Rational.Power(x, y, out result);  
        return result;  
    }

    public static string ToDecimalString(this Rational x)  
    {  
        var sb = new StringBuilder();  
        x.AppendDecimalString(sb, 50000);  
        return sb.ToString();  
    }

    public static string Join\<T\>(this IEnumerable\<T\> seq)  
    {  
        return string.Concat(seq);  
    }  
}

