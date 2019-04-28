# Iterator Blocks, Part Three: Why no yield in finally?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/16/2009 9:18:00 AM

-----

There are three scenarios in which code could be executing in a finally in an iterator block. In none of them is it a good idea to yield a value from inside the finally, so this is illegal across the board. The three scenarios are (1) normal cleanup, (2) exception cleanup, and (3) iterator disposal.

For the first scenario, suppose we have something like

 

try  
{  
  Setup();  
  yield return M();  
}  
finally  
{  
  yield return N();  
  Cleanup();  
}

How should we transform this into an iterator state machine? Naively, we want to do something like:

 

switch (this.state)  
{  
  case 0: goto LABEL0;  
  case 1: goto LABEL1:  
  case 2: goto LABEL2:  
  case 3: goto LABEL3:  
}  
LABEL0:  
try  
{  
  Setup();  
  this.current = M();  
  this.state = 1;  
  return true; // BUT DON'T RUN THE FINALLY\!  
LABEL1:  
}  
finally  
{  
  this.current = N();  
  this.state = 2;  
  return true;  
LABEL2:  
  Cleanup();  
}  
LABEL3:  
this.state = 3;  
return false;  

There's an immediate problem with this: we both "goto" into a finally block and "return" out of one. Neither is legal.

Leaving a finally block via "return" is illegal because it is weird. Suppose the try block returns, causing the finally to execute. What happens when the finally block returns? The original return value would be lost and some other return value would be substituted for it. This seems bizarre and bug-prone, so its not legal; you cannot exit a finally via return. And of course, you don't want the finally to run after the return in the try in this case\!

Furthermore, in the CLR model of exception handling it is illegal to branch via a goto into a try block or its "handler" (that is, the catch or finally clause). Nor can you branch out of the handler. These special regions have special code that needs to be run when the region is entered and exited; you cannot skip either with a goto.

So we have no immediately obvious way to generate verifiable code for this scenario. Right off the bat, we have a huge number of points against this feature; we'd have to either convince the CLR guys to allow spaghetti code involving protected regions, or come up with some clever technique for generating this code.

And that's just the scenario where nothing is going wrong yet\! Suppose a miracle happens and we manage to successfully generate code for scenario one. Now consider our second scenario, where M throws exception X. Remember, the finally block "catches" the exception and processes the cleanup code. If the cleanup code throws, then the original exception is discarded and the new processing of the new exception takes over. If the cleanup code succeeds, then the original exception continues to be "thrown" up the call stack, looking for more finally blocks or catch blocks.

Suppose the cleanup code does not throw. What should the control flow look like? The caller calls MoveNext for the first time. M() throws X. The finally block takes over. The finally block calls N(), and returns control to the caller with the results of N() \!? What happened to X, the exception? Is it just waiting there, in limbo? When the second call to MoveNext happens, should the cleanup code get run then, and X suddenly pops back into existence and continues to be thrown up the stack? That makes no sense at all; **the two calls to MoveNext could have had completely different stacks**\! This is crazy, plus we have no mechanism at all in the CLR for these kind of shenanigans.

Third, suppose we manage to solve all these problems. Now consider what happens when the caller calls MoveNext, M() succeeds, control returns to the caller, and the caller calls an early Dispose() on the enumerator. (I guess they wanted only one item.) We generate a Dispose method that checks the current state and executes any pending finally blocks. What the heck is the Dispose method supposed to do when it encounters the yield return in the pending finally block? We're not even in a call to MoveNext anymore\! Should we call N() and ignore the result? Should we return from the Dispose() after the call to N(), or do the cleanup and then return? What exactly should the control flow do here? We're in a context where *we might not even be iterating anymore, and yet we're still yielding.*

It really doesn't make any logical sense to do a yield return in this scenario; we're possibly not in a position where the caller is expecting things to continue to be yielded to it.

So in short, we'd have to do at least two impossible things in order to enable a scenario that makes no sense in the first place. If ever a feature called out to be cut at the design phase, this is it. Therefore: no yields inside finally blocks. Thank goodness for that.

Next time: now that you know all that, figuring out "why no yields inside catch blocks" is pretty straightforward.

