# I Take Exception To That

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/15/2003 1:22:00 PM

-----

 

 

Joel and Ned are having a spirited debate over the merits of exception handling.  Oddly enough, I agree with both of them.

 

 

The only time I do programming in C++ is to write COM code, and COM and exceptions do not mix.  The only way to make COM and C++ exceptions mix is to use smart pointers, and as I've already discussed, that only makes the situation worse by introducing subtle, impossible-to-find-and-debug bugs in the error handling.  When I write COM code every function call returns an HRESULT and I carefully ensure that if there is an error, then everything gets cleaned up properly.  COM is fundamentally an environment which is based on error return values for error handling.  Any additional error state can be provided by stateful objects.   

 

 

But the .NET CLR is a completely different environment, one with an exception model built deep into the runtime itself and into the framework.  You'd be *crazy* to not use the exception model in a fully garbage-collected environment like the CLR.  I write a lot of C++ and C\# code that has to work together via Interop; ALL of my C++ functions returns an error code, and NONE of my C\# functions do.  That is not to say that there is no error handling in my C\# code -- far from it\!  My code is chock full of try-catch-finally blocks to deal with errors.

 

 

Programming languages and object frameworks are tools **that afford different styles of programmin**g -- attempting to mix the style appropriate to one in another is a recipe for unmaintainable code.

