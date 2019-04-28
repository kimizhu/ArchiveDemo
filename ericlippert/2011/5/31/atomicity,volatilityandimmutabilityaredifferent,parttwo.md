# Atomicity, volatility and immutability are different, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/31/2011 2:56:24 PM

-----

Last time we established that an "atomic" read or write of a variable means that in multithreaded scenarios, you never end up with "halfway mutated" values in the variable. The variable goes from unmutated to mutated directly, with no intervening state. I also talked a bit about how making fields of a struct "readonly" has no effect on atomicity; when the struct is copied around, it may be copied around four bytes at a time regardless of whether its fields are marked as "readonly" or not.

There is a larger problem though with reasoning about "readonly" fields in a struct beyond their non-atomicity. Yes, when you read from readonly fields in a struct on multiple threads without any locking, you can get inconsistent results due to race conditions. But the situation is actually worse than that; "readonly" fields need not give you results that you think are consistent even on one thread\! Basically, "readonly" fields in a struct are the moral equivalent of the struct author writing a cheque without having the funds to back it.

I originally meant to write a lengthy article about this fact, but then I discovered that Joe Duffy already had done a great job. [Check out his article here](http://www.bluebytesoftware.com/blog/2010/07/01/WhenIsAReadonlyFieldNotReadonly.aspx).

Briefly summing up: since a struct does not "own" its own storage, a "readonly" on a field of a struct only means that the compiler will prevent the author of the code from writing directly to the field outside of a constructor. The actual variable that the struct is borrowing storage from need not be "readonly" and is free to change as the owner of that variable sees fit. Consider for example:

 

struct S  
{  
    readonly int x;  
    public S(int x) { this.x = x; }  
    public void M(ref S other)  
    {  
        int oldX = this.x;  
        other = new S(456);  
        int newX = this.x;  
    }  
}

Since "this.x" is a readonly field you might think that newX and oldX always have the same value. But if you say:

 

S s = new S(123);  
s.M(ref s);

then "this" and "other" are both aliases for "s", and "s" is free to change\!

Returning now to atomicity, I said last time that I'd discuss ways in which you can get more or less atomicity than the C\# language guarantees. Basically the C\# language guarantees that reads and writes of references are atomic, and reads and writes of built-in value types of four bytes or smaller (int, uint, float, char, bool, and so on) are guaranteed to be atomic, and that's it.

The CLI specification actually makes stronger guarantees. The CLI guarantees that reads and writes of variables of **value types that are the size (or smaller) of the processor's natural pointer size** are atomic; if you are running C\# code on a 64 bit operating system in a 64 bit version of the CLR then reads and writes of 64 bit doubles and long integers are also guaranteed to be atomic. The C\# language does not guarantee that, but the runtime spec does. (If you are running C\# code in some environment that is not implemented by some implementation of the CLI then of course you cannot rely upon that guarantee; contact the vendor who sold you the runtime if you want to know what guarantees they provide.)

Another subtle point about atomic access is that the underlying processor only guarantees atomicity when the variable being read or written is associated with storage that is aligned to the right location in memory. Ultimately the variable will be implemented as a pointer to memory somewhere. On a 32 bit operating system, that pointer has to be evenly divisible by 4 in order for the read or write to be guaranteed to be atomic, and on a 64 bit operating system it has to be evenly divisible by 8. If you do something crazy like:

 

int\* pInt1 = \&intArr\[0\];  
byte\* pByte = (byte\*)pInt;  
pByte += 6;  
int\* pInt2 = (int\*) pByte;  
\*pInt2 = 0x000DECAF;

then you are not guaranteed that the write which writes half the bytes to intArr\[1\] and the other half to intArr\[2\] is atomic. The underlying two array slots may be observed to be updated one after the other.

Now, the CLR does guarantee that all fields of all structs that are of the natural pointer size (or smaller) will by default be aligned in memory such that they do not cross a boundary like this, and therefore reads and writes of fields will be atomic. However, the C\# language does permit you to tell the CLR "do not use the default structure packing rules; [I will tell you how to lay out the bytes in a structure](http://msdn.microsoft.com/en-us/library/system.runtime.interopservices.structlayoutattribute.aspx)". (Perhaps because you have a byte buffer from some unmanaged code and you're using unsafe code to get a pointer to a struct that puts some structure onto those bytes.) If you tell the CLR to put an int field inside the structure such that it falls across an alignment boundary then the access to that field will not be atomic, and that's your problem to deal with.

**Next time**: what is "volatile", and how does it relate to "atomic" in C\#? The next episode will be somewhat delayed as I am in California this week attending yet another wedding. It seems like half my friends decided to get married in a row this year\! The wedding I officiated this past weekend went off without a hitch. The highlight: I finally got an opportunity to say "Frodo, bring forth the ring\!" in a context where it made sense.

