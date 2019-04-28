# The JScript Type System Part Eight: The Last Blog Entry About Arrays, I Promise

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/5/2003 1:12:00 PM

-----

 

 

 

Recall that I defined a *type* as consisting of two things: a **set** of values, and a **rule** for associating values outside of that set with values inside the set.  In JScript .NET, assigning a value outside of a type to a variable annotated with that type restriction does that coercion if possible

 

 

var s : String = 123; // Converts 123 to a String

 

 

Similarly, I already discussed what happens when you assign a JScript array to a hard-typed CLR array variable

 

 

var sysarr : int\[\] = \[10, 20, 30\]; // Create new int\[3\] and copy 

 

 

and what happens when you assign a one-dimensional CLR array to a JScript array variable:

 

 

var jsarr : Array = sysarr; // Wrap sysarr 

 

 

But what happens when you assign a **hard-typed CLR array** to a variable annotated with a **different** CLR array type?

 

 

var intarr : int\[\] = \[10, 20, 30\];

var strarr : String\[\] = intarr;

 

 

You might think that this does the string coercion on every element, but in fact this is simply not legal. Rather than creating a copy with every element coerced to the proper type, the compiler simply gives up and says that these are not type compatible. If you find yourself in this situation then you will simply have to write the code to do the copy for you.  Something like this would work:

 

 

function copyarr(source : System.Array) : String\[\]

{

  var dest : String\[\] = new String\[source.Length\];

  for(var index : int in source)

    dest\[index\] = source.GetValue(index);

  return dest;

}

 

 

There are a few notable things about this example. First, notice that this copies a rank-one array of any element type to an array of strings. This is one of the times when it comes in handy to have the System.Array "any hard-typed array" type\!

 

 

Second, notice that you can use the for-in loop with hard-typed CLR arrays. The for-in loop **enumerates all the indices of an array rather than the contents of the array.** Since CLR arrays are always indexed by integers the index can be annotated as an int. The loop above is effectively the same as

 

 

for (var index : int = 0 ; index \< source.Length ; ++index)

 

 

but the for-in syntax is less verbose and possibly more clear.

 

 

Third, you might recall that GetValue (and SetValue) take an **array** of indices because the array might be multidimensional. But we're not passing in an array here.  Fortunately, you can also pass only the index if it is a single-dimensional array.

 

 

Generally speaking, **hard-typed array types are incompatible with each other.** There is an exception to this rule, which I'll discuss later when I talk about what exactly "subclassing" means in JScript .NET.

