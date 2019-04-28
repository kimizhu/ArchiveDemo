# Too much reuse

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/4/2010 6:33:00 AM

-----

A recent user question:

> I have code that maintains a queue of pending work items waiting to be completed on various different worker threads. In certain unfortunate fatal error situations I complete each of these by throwing an exception. Can I create just one exception object? Are there any issues throwing the same exception object multiple times on multiple threads?

Anyone who has ever seen this in a code review knows the answer:  

catch(Exception ex)  
{  
   Logger.Log(ex);  
   throw ex;  
}

This is a classic “gotcha”; almost always the right thing to do is to say “throw;” rather than “throw ex;” – the reason being that **exceptions are not completely immutable in .NET.** The exception object’s stack trace is set at the point where the exception is thrown, every time it is thrown, not at the point where it is created. The “throw;” does not reset the stack trace, “throw ex;” does.

Don’t reuse a single exception object. Every time gets thrown the stack trace will be reset, which means that any code up the stack which catches the exception and logs the trace for later analysis will almost certainly be logging someone else’s trace. Making exceptions is cheap, and you’re already in a fatal error situation; it doesn’t matter if the app crashes a few microseconds slower. Take the time to allocate as many exceptions as you plan on throwing.

