# Chained user-defined explicit conversions in C\#, Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/20/2007 10:00:00 AM

-----

Jeroen Frijters knew the answer to my challenge of last time: how is it that Foo foo = new Foo(); can cause a runtime conversion failure? And how is it that Bar bar = (Bar)(new Baz()); can succeed even if there is no user-defined conversion or built-in implicit conversion between Baz and Bar?

The answer is that it is, as Stuart Ballard editorialized, “a goofy COM interop thing that the language should never have supported directly”.

In COM programming you never talk to a object directly, as a class. Rather, the object has a number of public interfaces; the client of a particular object asks the object for the interface it wants, and it talks to the object on that interface. COM provides a special function called CoCreateInstance which gets the ball rolling; it knows how to create an instance of a specific class and provides a pointer to a given interface on that object.

Suppose you need to interoperate with an existing COM object in C\#. We could have made you do all the work you’d do in a C++ program: call CoCreateInstance, get the interface, blah blah blah. But that’s kind of gross and ugly. Instead, we take advantage of the fact that the vast majority of the time that you are talking to a COM object is via an interop library created with the type library importer. The type library importer knows from the information in the type library which interfaces are associated with which coclasses. The typical pattern of interop classes created by the type library importer would look something like this if we wrote out the code in C\#:

 

using System.Runtime.InteropServices;  
\[ComImport\]  
\[Guid("12345678-1234-1234-1234-123412341234")\]  
\[CoClass(typeof(InteropClass))\]  
public interface IMyInterface  
{ /\* whatever \*/}  
public class InteropClass : IMyInterface {/\*whatever\*/}

So the C\# compiler knows that IMyInterface is associated with the coclass InteropClass. We therefore allow this:

 

IMyInterface x = new IMyInterface();

as is a syntactic sugar for

 

IMyInterface x = (IMyInterface)(new InteropClass());

So this is yet another place where we introduce an explicit cast on your behalf.

This means that it is legal to say

 

InteropClass x = (InteropClass)(new IMyInterface());

And it will not fail at run time, even though there is no user-defined conversion or built-in implicit conversion from IMyInterface to InteropClass\!

Unfortunately, there’s a bit of a bug in the C\# compiler. We assume two things. First, that the type library importer always generates this pattern correctly, and second, that no one other than the type library importer pulls shenanigans like this. The first assumption is probably good, but there’s nothing stopping anyone from pulling shens like:

 

using System.Runtime.InteropServices;  
\[ComImport\]  
\[Guid("12345678-1234-1234-1234-123412341234")\]  
\[CoClass(typeof(InteropClass))\]  
public interface IMyInterface  
{ /\* whatever \*/}  
public class InteropClass {/\*whatever\*/} // oops, no interface

The compiler does not check to see if InteropClass actually implements IMyInterface\! Therefore this guaranteed-to-fail explicit cast gets through at compile time and fails at run time.

Obviously this is really gross. I don’t think we’ll get to fixing this in C\# 3.0, but perhaps in a later version of the language we will tighten this up and make it illegal to have a coclass which does not actually implement the given interface.

Next time, one more place where poorly specified compiler behaviour causes us to insert an explicit cast.

