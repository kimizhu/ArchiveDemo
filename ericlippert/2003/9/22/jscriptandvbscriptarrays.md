# JScript and VBScript Arrays

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/22/2003 1:55:00 PM

-----

 

 

 

 

Earlier I alluded to the fact that JScript arrays are objects but VBScript arrays are not.  What's up with that?

 

 

It's kind of strange.  Consider the properties of a JScript array.  A Jscript array is

 

 

\* one dimensional.

\* associative; JScript arrays are indexed by strings.  Numeric indices are actually converted to strings internally.

\* sparse: arr\[1\] = 123; arr\[1000000\] = 456; gives you a two-member array, not a million-member array.

\* an object with properties and methods.

 

 

Whereas a VBScript array is

 

 

\* multi-dimensional

\* indexed by integer tuples

\* dense

\* not an object

 

 

**It is hard to come up with two things that could be more different and yet both called "array".**

 

 

As you might expect, JScript and VBScript arrays have completely different implementations behind the scenes.  A JScript array is basically just a simple extension of the existing JScript expando object infrastructure, which is implemented as a (rather complicated) hash table.  A VBScript array is implemented using the SAFEARRAY data structure, which is pretty much just a structure wrapped around a standard "chunk of memory" C-style array.

 

 

Since all the COM objects in the world expect VBScript-style arrays and not JScript-style arrays, making JScript interoperate with COM is not always easy.  I wrote an object called VBArray into the JScript runtime to translate VBScript-style arrays into JScript arrays, but it is pretty kludgy.  And though I wrote some code to go the other way -- to turn JScript arrays into VBScript arrays -- there were just too many thorny issues involving object identity and preservation of information for us to actually turn it on.  There weren't exactly a whole lot of users demanding the feature either, so that part of the feature got cut.  (If I recall correctly, my colleague Peter Torr wrote some COM code to do that before he came to work at Microsoft.  He might still have it lying around.)

 

 

Things got even weirder when I wrote the code to interoperate between CLR arrays and JScript .NET arrays, but that's another story.

