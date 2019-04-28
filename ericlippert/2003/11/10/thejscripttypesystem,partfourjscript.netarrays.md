# The JScript Type System, Part Four: JScript .NET Arrays

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/10/2003 12:51:00 PM

-----

 

As I mentioned in [an earlier entry](http://blogs.gotdotnet.com/ericli/commentview.aspx/117917ce-15ca-49ee-a366-805fbcdadb54), one of the major differences between JScript .NET and JScript Classic is that JScript .NET now supports optional type annotations on variables.  The number of built-in primitive types has also increased dramatically.  JScript .NET adds value types boolean, byte, char, decimal, double, float, int, long, sbyte, short, uint, ulong and ushort.  In addition, JScript .NET integrates its type system with the CLR type system -- a string in JScript has all the properties and methods of the string prototype *and* all the properties and methods of a System.String.  Backwards compatibility and interoperability with the CLR were two very important design criteria.

 

 

The primitive types are pretty straightforward though.  Some more interesting stuff happens when we think about how complex types like arrays interoperate between JScript .NET and the CLR.  I [already discussed](http://blogs.gotdotnet.com/ericli/PermaLink.aspx/5d45cae2-bb41-4925-8878-bb88ecf8dd14) some of these issues regarding JScript and VBScript arrays, so let's quickly review the terminology:

 

 

A **sparse** array may have "holes" in the valid indices. A sparse array with three elements might have elements 0, 10 and 1000 defined but nothing in between. The opposite of a sparse array is a **dense** array. In a dense array all the indices between the lowest and highest indices are valid indices. A dense array with elements 0 and 1000 has 1001 elements.

 

 

A **fixed-size** array has a particular valid range of indices. Typically the size is fixed when the array is created and it may not be changed. A **variable-sized** array does not have any maximum size. Elements may be added or removed at any time.

 

 

A **single-dimensional** array maps a single index onto a value. A **multi-dimensional** array may require any number of indices to fetch a value.

 

 

The number of dimensions an array has is called its **rank**. (Other terms such as **dimensionality** or **arity** are occasionally used but we will stick to rank.)

 

 

A **static-typed** array or **hard-typed** array is an array where every element is of the same type. A **dynamically-typed** or **soft-typed** array may have elements of any type.

 

 

An **associative** array is an array where the indices are strings. A **nonassociative** array has integer indices.

 

 

A **literal** array is a JScript .NET array defined in the actual source code, much as "abcde" is a literal string or 123.4 is a literal number. In JScript .NET a literal array is a comma-separated list of items inside square brackets:

 

 

var arr = \[10, 20, "hello"\];

var item = arr\[1\]; // 20

 

 

JScript arrays are **sparse**, **variable-sized**, **single-dimensional**, **soft-typed** **associative** arrays. CLR arrays are the opposite in every way\! They are **dense,** **fixed-size**, **multi-dimensional**, **hard-typed** **nonassociative** arrays. It is hard to imagine two more different data structures with the same name.  Making them interoperate at all was a pain in the rear, believe me.

 

 

This is a pretty big topic, so I think I'll split it up over a few entries.  Let me talk a bit about annotation and typing, and we'll pick up where we left off tomorrow.

 

 

Traditional JScript arrays are soft-typed; they can store heterogeneous data:

 

 

var arr = new Array();

arr\[0\] = "hello";

arr\[1\] = 123.456;

arr\[2\] = new Date();

 

 

CLR arrays, on the other hand, are hard-typed. Every element of a CLR array is the same type as every other element. This difference motivates the type annotation syntaxes for each. In JScript .NET the traditional arrays are annotated with the Array type and hard-typed CLR arrays are annotated with the type of the element followed by \[\]:

 

 

var jsarr : Array = new Array()

jsarr\[0\] = "hello";

 

 

var sysarr : double\[\] = new double\[10\];

sysarr\[0\] = 123.4

 

 

Note that CLR arrays are fixed-size, but **the size is not part of the type annotation**; sysarr can be a one-dimensional array of double of any size. This is perfectly legal, for example:

 

 

var sysarr : double\[\] = new double\[10\];

sysarr\[0\] = 123.4

sysarr = new double\[5\];

 

 

This throws away the old array and replaces it with a new, smaller array. But **once a hard-typed array is created it may not be resized.** 

 

 

Tomorrow: true multidimensional arrays in JScript .NET.

