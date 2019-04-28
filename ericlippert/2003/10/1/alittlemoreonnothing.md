# A Little More on Nothing

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/1/2003 1:13:00 PM

-----

VBScript has Null, Empty and Nothing.  What about JScript?  Unfortunately, JScript is a little screwed up here.

 

Like VBScript, in JScript an uninitialized variable is a VT\_EMPTY variant.  To check to see if a variable is undefined, you can say

 

typeof(x) == "undefined"

 

There is a built-in Empty constant like in VBScript -- in JScript it is undefined.

 

If you say

 

var x = null;

print(typeof(x));

 

then you get "object", so you might expect that null is the JScript equivalent of Nothing -- an object reference that refers to no object.  From a *logical* perspective, you'd be right.  Unfortunately, for reasons which are lost to the mists of time, JScript implements a null variable internally the same way that VBScript implements a Null variable -- VT\_NULL, the "data is missing" value.

 

This poor decision leads to two problems.  First, JScript does not interoperate very well with COM objects which have methods that can legally take a Nothing but not a Null.  (I believe that Peter Torr has some addon code that adds this capability to JScript.) Second, JScript processes Null variants passed into it as though they were object references, not database nulls.  Thus JScript does not implement any of the rules about Null propagation that VBScript implements.

 

We corrected these problems somewhat in JScript .NET.  JScript .NET uses a null object reference to internally represent null, and can manipulate DBNull objects, so the interop problems go away.  However, JScript .NET still does not implement the null propagation semantics for mathematical operations on database nulls.

