# The JScript Type System Part Seven: Yeah, you've probably guessed that I wrote the array stuff

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/21/2003 1:23:00 PM

-----

A reader asked me to clarify a point made in an earlier entry:

 

 

Note that JScript .NET arrays do not have any of the methods or properties of a CLR array. (Strings, by contract, can be used implicitly as either JScript .NET strings or as System.String values, which I'll talk more about later.) But JScript .NET arrays do not have CLR array fields like Rank, SetValue, and so on.

 

 

It's actually pretty straightforward.  When you have a string in a JScript .NET program, we allow you to treat it as both a System.String and as a JScript object with the String prototype.  For example:

 

 

var s = "   hello   ";

print(s.toUpperCase()); // calls JScript string prototype's toUpperCase

print(s.Trim()); // calls System.String.Trim

 

 

Which is it?  From a theoretical standpoint, it doesn't really matter -- you can use it as either.  From an implementation standpoint, of course we use System.String internally and magic up the prototype instance when we need one -- just as in JScript classic all strings are VT\_BSTR variants internally and we magic up a wrapper when we need one.  So JScript strings and CLR strings really are totally interoperable.

 

 

Arrays aren't quite so seamless.  As I mentioned earlier, when you try to use a JScript .NET array when a CLR array is expected, we create a copy.  But when you go the other way, things are a little different. Rather than producing a copy, using a CLR array as a JScript .NET array "wraps it up". No copy is made. The operation is therefore efficient **and preserves identity**. Changes made to a wrapped array are preserved:

 

 

function ChangeArray(arr : Array) : void

{

  print(arr\[0\]); // 10

  arr\[0\] += 100;

  // JScript .NET methods work just fine 

  print(arr.join(":")); // 10:20:30 

}

var arr : int\[\] = \[10, 20, 30\];

ChangeArray(arr);

print(arr\[0\]); // 110 

 

 

The principle rule for treating a hard-typed array as a JScript .NET array is that **it must be single-dimensional**. Since all JScript .NET arrays are single-dimensional it makes no sense to wrap up a high-rank CLR array.

 

 

Once the array is wrapped up it still has all the restrictions that a normal hard-typed array has. It may not change size, for instance. This means that an attempt to call certain members of the JScript .NET Array prototype on a wrapped array will fail. All calls to push, pop, shift, unshift and concat as well as some calls to splice will change the length of the array and are therefore illegal on wrapped CLR arrays.

 

 

Note that you may use the other JScript .NET array prototype methods on any hard-typed array (but not vice versa). You can think of this as implicitly creating a wrapper around the CLR array, much as a wrapper is implicitly created when calling methods on numbers, Booleans or strings:

 

 

var arr : int\[\] = \[10, 20, 30\];

arr.reverse();    // You may call JScript .NET methods on hard-typed arrays

print(arr.join(":"));   // 30:20:10

 

 

There may be a situation where you *do* want to make a copy of a CLR array rather than wrapping it. JScript .NET has syntax for this, namely:

 

 

var sysarr : int\[\] = \[10, 20, 30\];

var jsarr1 : Array = sysarr; // create wrapper without copy

var jsarr2 : Array = Array(sysarr); // create wrapper without copy

var jsarr3 : Array = new Array(sysarr); // not a wrapper; copies contents

 

 

In the last case jsarr3 is not a wrapper. It is a real JScript .NET array and may be resized.

