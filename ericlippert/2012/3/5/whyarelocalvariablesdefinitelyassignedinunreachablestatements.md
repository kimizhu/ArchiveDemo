# Why are local variables definitely assigned in unreachable statements?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/5/2012 2:05:18 PM

-----

You're probably all familiar with the feature of C\# which disallows reading from a local variable before it has been "definitely assigned":

void M()  
{  
  int x;  
  if (Q())  
    x = 123;  
  if (R())  
    Console.WriteLine(x); // illegal\!  
}

This is illegal because there is a path through the code which, if taken, results in the local variable being read from before it has been assigned; in this case, if Q() returns false and R() returns true.

The reason why we want to make this illegal is not, as many people believe, because the local variable is going to be initialized to garbage and we want to protect you from garbage. We do in fact automatically initialize locals to their default values. (Though the C and C++ programming languages do not, and will cheerfully allow you to read garbage from an uninitialized local.) Rather, it is because the existence of such a code path is probably a bug, and we want to throw you in the pit of quality; you should have to work hard to write that bug.

The way in which the compiler determines if there is any path through the code which causes x to be read before it is written is quite interesting, but that's a subject for another day. The question I want to consider today is: **why are local variables considered to be definitely assigned inside unreachable statements?**

void M()  
{  
  int x;  
  if (Q())  
    x = 123;  
  if (false)  
    Console.WriteLine(x); // legal\!  
}

First off, obviously the way I've described the feature immediately gives the intuition that this ought to be legal. Clearly there is no path through the code which results in the local variable being read before it is assigned. In fact, there is no path through the code that results in the local variable being read, period\!

On the other hand: **that code looks wrong**. We do not allow syntax errors, or overload resolution errors, or convertibility errors, or any other kind of error, in an unreachable statement, so why should we allow definite assignment errors?

It's a subtle point, I admit. Here's the thing. You have to ask yourself "why is there unreachable code in the method in the first place?" Either that unreachable code is deliberate, or it is an error.

If it is an error, then something is deeply messed up here. The programmer did not intend the written control flow in the first place. It seems premature to guess at what the definite assignment errors are in the unreachable code, since the control flow that would be used to determine definite assignment state is wrong. We are going to give a warning about the unreachable code; the user can then notice the warning and fix the control flow. Once it is fixed, then we can consider whether there are definite assignment problems with the fixed control flow.

Now, why on earth would someone deliberately make unreachable code? It does in fact happen; actually it happens quite frequently when dealing with libraries made by another team that are not quite done yet:

// If we can resrov the frob into a glob, do that and then blorg the result.  
// Even if the frob is not a glob, we know it is definitely a resrovable blob,  
// so resrov it as a blob and then blorg the result. Finally, fribble  
// the blorgable result, regardless of whether it was a glob or a blob.  
void BlorgFrob(Frob frob)  
{  
  IBlorgable blorgable;  
  // TODO: Glob.TryResrov has not been ported to C\# yet.  
  if (false /\* Glob.TryResrov(out blorgable, frob) \*/)  
  {  
    BlorgGlob(blorgable);  
  }  
  else  
  {  
    blorgable = Blob.Resrov(frob)  
    BlorgBlob(blorgable);  
  }  
  blorgable.Fribble(frob);  
}

Should BlorgGlob(blorgable) be an error? It seems plausible that it should not be an error; after all, it's never going to read the local. But it is still nice that we get overload resolution errors reported inside the unreachable code, just in case there is something wrong there.

