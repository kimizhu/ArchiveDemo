# Iterator blocks Part Four: Why no yield in catch?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/20/2009 9:44:00 AM

-----

Now that you know why we disallow yielding in a finally, it’s pretty straightforward to see why we also disallow yielding in a catch.

First off, we still have the problem that it is illegal to “goto” into the middle of the handler of a try-protected region. The **only** way to enter a catch block is via the “non-local goto” that is catching an exception. So once you yielded out of the catch block, the next time MoveNext was called, we’d have no way to get back into the catch block where we left off.

Second, the exception that was thrown and caught is an intrinsic part of the execution of the catch block because it can be re-thrown using the “empty throw” syntax. We have no way of preserving that state across calls to MoveNext.

I suppose that we could in theory allow yields inside catch blocks that do not have an “empty throw”. We could turn this:

 

try  
{  
  M();  
}  
catch(Exception x)  
{  
  yield return x.ToString().Length;  
  yield return 123;  
}

into

 

switch(this.state)  
{  
case 0: goto LABEL0;  
case 1: goto LABEL1;  
case 2: goto LABEL2;  
}  
LABEL0:  
try  
{  
  M();   
  this.state = 2;  
  goto LABEL2;  
}  
catch(Exception x)  
{  
  this.x = x;   
}  
this.current = this.x.ToString().Length;  
this.state = 1;  
return true;  
LABEL1:  
this.current = 123;  
this.state = 2;  
return true;  
LABEL2:  
return false;

But frankly, that would be pretty weird. It would feel like a very odd restriction to disallow a fundamental part of the “catch” syntax.

And really, what’s the usage case that motivates this situation in the first place? Do people really want to try something and then yield a bunch of results if it fails? I suppose you could do something like:

 

IEnumerable\<string\> GetFooStrings(Foo foo)  
{  
  try  
  {  
    yield return Foo.Bar();  
  }  
  catch(InvalidFooException x)  
  {  
    yield return "";  
  }  
… etc.  

but there are lots of ways to easily rewrite this so that you don’t have to yield in a catch:

IEnumerable\<string\> GetFooStrings(Foo foo)  
{  
  string result;  
  try  
  {  
    result = Foo.Bar();  
  }  
  catch(InvalidFooException x)  
  {  
    result = "";  
  }  
  yield return result;  
… etc.  

Since there are usually easy workarounds, and we would have to put crazy-seeming restrictions on the use of re-throws, it’s easier to simply say “no yields inside a catch”.

We still haven’t explained why it is *illegal* to yield inside a try that has a catch, nor why it is *legal* to yield inside a try that has a finally. To understand why, next time I’ll need to muse about the duality between sequences and events.

