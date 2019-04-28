# Long jumps considered way more harmful than exceptions

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/16/2003 6:20:00 PM

-----

Bob Congdon's blog (<http://www.bobcongdon.net/blog/>) points out that in the dark days before exception handling you could always use setjmp/longjmp to do non-local gotos.

 

 

In fact, the script engines are compiled in C++ with exception handling turned off (for performance reasons), and the mainline loop of the bytecode interpreter uses setjmp-longjmp exception handling to implement error handling.  When you have a script that calls an object that returns an error, we longjmp back to the start of the interpreter loop and then figure out what to do next.   

 

 

In VBScript of course it depends on whether On Error Resume Next is on or not, and in JScript we construct an exception object and start propagating it back up the stack until we find an interpreter frame that has a catch block.  (If there are multiple script engines on the stack then things get extremely complicated, so I won't even go there.)

 

 

Since a long jump does not call any destructors, it was very important that we design our interpreter loop to not put anything on the system stack that required destructing.  Fortunately, since we were designing the interpreter to be an interpreter for a garbage-collected language, it was pretty easy.  Everything that the interpreter does that requires memory either takes the memory out of the area reserved for the script's stack (which will be cleaned up when the frame goes away) or heap-allocates it and adds the memory to the garbage collector.

 

 

Not everyone has the luxury of having a longjmp-safe garbage collector already implemented, so kids, don't try this at home\!  If you must use exception handling in C++, take my advice and use real C++ exception handling.

