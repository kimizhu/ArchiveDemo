# References and Pointers, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/7/2011 9:50:35 AM

-----

Writing code in C\# is really all about the programmatic manipulation of *values*. A value is either of a value type, like an integer or a decimal, or it's a reference to an instance of a reference type, like a string or an exception. Values you manipulate always have a storage location that stores the value; those storage locations are called "variables". Often in a C\# program you manipulate the values by describing which variable you're interested in.

In C\# there are three basic operations you can do to variables:

\* Read a value from a variable  
\* Write a value to a variable  
\* Make an alias to a variable

The first two are straightforward. The last one is accomplished by the "ref" and/or "out" keywords:

 

void M(ref int x)  
{  
    x = 123;  
}  
...  
int y = 456;  
M(ref y);

The "ref y" means "make x an alias to the variable y". (I wish that the original designers of C\# had chosen "alias" or some other word that is less confusing than "ref", since many C\# programmers confuse the "ref" in "ref int x" with reference types. But we're stuck with it now.) While inside M, the variable x is just another name for the variable y; they are two names for the same storage location.

There's a fourth operation you can do to a variable in C\# that is not used very often because it requires unsafe code. You can take the address of a fixed variable and put that address in a pointer.

 

unsafe void M(int\* x)  
{  
    \*x = 123;  
}  
...  
int y = 456;  
M(\&y);

**The purpose of a pointer is to manipulate *a variable itself* as data, rather than manipulating the *value* of that variable as data.** **If x is a pointer then \*x is the associated variable.**

Clearly pointers are very similar to references, and in fact references are implemented behind the scenes with a special kind of pointer. However, you can do things with pointers that you cannot do with references. For example, this doesn't do anything useful:

 

int Difference(ref double x, ref double y)  
{  
    return y - x;  
}  
...  
double\[\] array = whatever;  
difference = Difference(ref array\[5\], ref array\[15\]);

That's illegal; it just takes the difference of the two doubles and tries to convert it to an int. But with pointers you can actually figure out how far apart in memory the two variables are:

 

unsafe int Difference(double\* x, double\* y)  
{  
    return y - x;  
}  
...  
double\[\] array = whatever;  
fixed(double\* p1 = \&array\[5\])  
  fixed(double\* p2 = \&array\[15\])  
    difference = Difference(p1, p2); // 10 doubles apart

You can do arithmetic on pointers, but you can't do that on refs because in C\# there is no way to say to a ref "I want to manipulate the storage location itself, rather than its contents". With pointers, again, **the pointer represents the storage location itself**; dereferencing the pointer with \* gives you access to **the variable** that lets you get or set the **value** of that storage location.

Similarly, you can compare a pointer to null, but you can't compare a ref to a variable to null; comparing the ref to null just checks to see if the contents of the variable are null; there is no such thing as a "null ref".

Another thing you can do with pointers is treat them as arrays; you can't do that with refs:

 

unsafe double N(double\* x)  
{  
  return x\[10\];  
}  
...  
double\[\] array = whatever;  
fixed(double\* p1 = \&array\[5\])  
  q = N(p1); // returns array\[15\];

All of this is of course **fraught with peril**. We make you mark the code as "unsafe" for a reason; **it is not safe to do any of this stuff**. When you use pointers directly **you are turning off the safety system** and taking responsibility yourself for ensuring that every operation on a pointer is one that makes sense. For example, suppose we had passed interior pointers from two difference arrays into Difference above. What would have happened? The result would not have been sensible; it doesn't make any sense to ask how many items are between two elements of two different arrays. It only makes sense to ask that question within one array. Suppose in the code above we had passed the address of array\[5\] and the array only had 7 elements. What happens when we try to get the fifteenth element? The managed safety system is turned off, so you would not get an array-index-out-of-bounds exception with pointers, you just get garbage or a crashed runtime.

Furthermore, note that the array has to be "fixed" before you can take an interior pointer to it. Fixing an array tells the garbage collector "someone has an interior pointer to this thing; do not move it during compaction until it is unfixed". That causes all kinds of problems. First, it can really mess up the ability of the GC to efficiently manage memory, because now there is a chunk of memory it is not allowed to move. And second, again, you are responsible for doing things safely; if you leave a copy of the pointer lying around and dereference it after the fixed statement has completed then there is no guarantee that the array is still there\! You could be dereferencing any old thing.

It's a bit unfortunate that it is such a pain to use interior pointers in an array in C\#, because doing so is often useful. We have many situations in the compiler where we would like to pass around locations of variables that are interior to arrays, compare their locations, and so on. Do we have to use unsafe code and fix the array in place to do so? Fortunately no\!

**Next time**: How to make a safe interior pointer to an array that you can still treat as a pointer, more or less.

