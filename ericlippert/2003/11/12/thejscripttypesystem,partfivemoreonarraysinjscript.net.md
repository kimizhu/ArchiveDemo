# The JScript Type System, Part Five: More On Arrays In JScript .NET

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/12/2003 1:05:00 PM

-----

As I was saying the other day, CLR arrays and JScript arrays are totally different beasts. It is hard to imagine two things being so different and yet both called the same thing. Why did the CLR designers and the JScript designers start with the same desire -- create an array system -- and come up with completely different implementations? 

 

 

Well, the CLR implementers knew that dense, nonassociative hard-typed arrays are easy to make **fast and efficient**. Furthermore, such arrays **encourage the programmer to keep homogenous data in strictly bounded tables**. That makes **large programs that do lots of data manipulation easier to understand**. Thus, languages such as C++, C\# and Visual Basic have arrays like this, and thus they are the basic built-in array type in the CLR.

 

 

Sparse, associative, soft-typed arrays are not particularly fast but they are **far more dynamic and flexible** than Visual Basic-style arrays. They make it easy to store heterogeneous data in any table **without worrying about picky details** like exactly how big that table is. In other words, they are **scripty**. Languages such as JScript and Perl have arrays like this.

 

 

JScript .NET has **both** very dynamic, scripty arrays and more strict CLR arrays, making it **suitable for both rapid development of scripts and programming in the large**. But like I said, making these two very different kinds of arrays work well together is not trivial. 

 

 

JScript .NET supports the creation of multidimensional hard-typed arrays. As with single-dimensional arrays, **the array size is not part of the type**. To annotate a variable as containing a hard-typed multidimensional array the syntax is to follow the type with brackets containing commas. For example, to annotate a variable as containing a two dimensional array of Strings you would say:

 

 

var multiarr : String\[,\];

 

 

The number of commas between the brackets plus one is equal to the rank of the array. (By this definition if there are no commas between the brackets then it is a rank-one array, as we have already seen.)

 

 

A multidimensional array is allocated with the new keyword as you might expect:

 

 

multiarr = new String\[4,5\];

multiarr\[0,0\] = "hello";

 

 

Notice that **hard-typed array elements are always accessed with a comma-separated list of integer indices**. There must always be **exactly one index for each dimension in the array**. You can't use the ragged array syntax \[0\]\[0\].   

 

 

There are certain situations in which you know that a variable or function argument will refer to a hard-typed CLR array but you do not actually know the element type or the rank, just that it is an array. Should you find yourself in one of these (rather rare) situations there is a special annotation for a CLR array of unknown type and rank:

 

 

var sysarr : System.Array;

sysarr = new String\[4,5\];

sysarr = new double\[10\];

 

 

As you can see, a variable of type System.Array may hold any CLR array of any type and rank. However, there is a drawback. Variables of type System.Array **may not be indexed directly because the rank is not known**. This is illegal:

 

 

var sysarr : System.Array;

sysarr = new String\[4,5\];

sysarr\[1,2\] = "hello";  // ILLEGAL, System.Arrays are not indexable

 

 

Rather, to index a System.Array you must call the GetValue and SetValue methods **with an array of indices:**

 

 

var sysarr : System.Array;

sysarr = new String\[4,5\];

sysarr.SetValue("hello", \[1,2\]);

 

 

The rank and size of a System.Array can be determined with the Rank, GetLowerBound and GetUpperBound members.

 

 

Thinking about this a bit now, I suppose that we *could* have detected at compile time that a System.Array was being indexed, and constructed the call to the getter/setter appropriately for you, behind the scenes.  But apparently we didn't.  Oh well.

 

 

Next time: mixing and matching JScript and CLR arrays.

