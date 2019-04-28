# Continuation Passing Style Revisited Part Two: Handwaving about control flow

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/22/2010 6:29:00 AM

-----

Last time on Fabulous Adventures: “*But we can construct arbitrarily complex control flows by keeping track of multiple continuations and deciding which one gets to go next.*”

Let’s look at an example of something more complex than a conditional. Consider a simplified version of “try-catch”, where there is no expression to the throw. A throw is simply a non-local goto that goes to the nearest enclosing catch. The traditional way to think about  

void Q()  
{  
  try  
  {  
    B(A());  
  }  
  catch  
  {  
    C();  
  }  
  D();  
}  
  
int A()  
{  
    throw;  
    return 0; // unreachable, but let's ignore that  
}  
void B(int x) { whatever }  
void C() { whatever }  
void D() { whatever }

is “Remember that there is a catch block somewhere near here. Remember what you were doing. Call A(). If it returns normally, remember what you were doing and then call B(), passing the result from A(). If B() returns normally, call D(). If A() or B() do not return normally then look for that catch block you remembered earlier. Call C() and then D().

In the real CLR [the code to implement throw is insanely complicated](http://blogs.msdn.com/b/cbrumme/archive/2003/10/01/51524.aspx); it has to run around the call stack looking for filters and handlers, and so on. Suppose a language did not have try-catch built in. There’s no way that we could implement all that insane complication as a *library method*, even if we wanted to. Or is there? Can we make Try() and Throw() methods in a language that supports CPS?

We could start by passing *two* continuations around to every function that might throw: one "normal" continuation, and one "error" continuation. Let's suppose that Q can throw. We'll translate everything into "two continuation" CPS. So we now have:

 

void A(Action\<int\> normal, Action error)  
{  
    Throw(()=\>normal(0), error);  
}

That should make sense. What does A() do? It calls Throw, and then it returns zero by passing zero to its normal continuation. So the normal continuation of Throw is to pass zero to the normal continuation of A. (As we'll shortly see, this is irrelevant because Throw by definition does not complete normally. But it's just another function call, so let's continue to treat it as one.)

 

void B(int x, Action normal, Action error) { whatever }  
void C(Action normal, Action error) { whatever }  
void D(Action normal, Action error) { whatever }

What's the implementation of Throw? That's easy\! Throw invokes the error continuation and abandons the normal continuation.

 

void Throw(Action normal, Action error)  
{  
    error();  
}

What's the implementation of Try? That's harder. What does try-catch do? It establishes a new error continuation for the try body, but not for the catch body. How are we going to do that? Let me pull this out of thin air, and we'll see if it works:

 

void Try(Action\<Action, Action\> tryBody,  
         Action\<Action, Action\> catchBody,  
         Action outerNormal,  
         Action outerError)  
{  
    tryBody(outerNormal, ()=\>catchBody(outerNormal, outerError));  
}

void Q(Action qNormal, Action qError)  
{  
    Try (  
       /\* tryBody      \*/ (bodyNormal, bodyError)=\>A(  
       /\* normal for A \*/   x=\>B(x, bodyNormal, bodyError),  
       /\* error for A  \*/   bodyError),  
       /\* catchBody    \*/ C,  
       /\* outerNormal  \*/ ()=\>D(qNormal, qError),  
       /\* outerError   \*/ qError );  
}

OK, first off, is this in CPS? Yes. Every method is void returning, every delegate is void returning, and every method or lambda has as its last act a call to some other method.

Is it correct? Let's walk through it. We call Try and pass a try body, a catch body, a normal continuation and an error continuation.

The Try invokes the try body, which takes two continuations. Try passes outerNormal, which was ()=\>D(qNormal, qError), as the normal continuation of the body.

It passes ()=\>catchBody(outerNormal, outerError) as the error continuation of the body. So that we're sure about what that does, let's expand that out. The catch body was C. Therefore the error continuation of the body will be evaluated as ()=\>C(()=\>D(qNormal, qError), qError). (Make sure this step makes sense to you.)

What was the try body? It was (bodyNormal, bodyError)=\>A(x=\>B(x, bodyNormal, bodyError), bodyError). We now know what bodyNormal and bodyError are, so let's expand those out too. When we expand all this out, the execution of the body will do:

 

A(  
  x=\>B(               // A's normal continuation  
    x,                // B's argument  
    ()=\>D(            // B's normal continuation  
      qNormal,        // D's normal continuation  
      qError),        // D's error continuation  
    ()=\>C(            // B's error continuation  
      ()=\>D(          // C's normal continuation  
        qNormal,      // D's normal continuation  
        qError),      // D's error continuation  
      qError)),       // C's error continuation  
  ()=\>C(              // A's error continuation  
    ()=\>D(            // C's normal continuation  
      qNormal,        // D's normal continuation  
      qError),        // D's error continuation  
    qError))          // C's error continuation

So, the body invokes A. It immediately invokes Throw, passing some complicated thing as the normal continuation and ()=\>C(()=\>D(qNormal, qError), qError) as the error continuation.

Throw ignores its normal continuation, abandoning it, and invokes ()=\>C(()=\>D(qNormal, qError), qError).

What if C (or something it calls) throws? if C throws then control immediately transfers to qError - whatever error handler is in place during the call to Q. What if C completes normally? Its normal continuation is ()=\>D(qNormal, qError), so if C completes normally then it invokes D.

What if D then throws? Then it calls qError. If D does not throw then eventually (we hope\!) it calls qNormal.

Take a step back. What if we remove the Throw so that A doesn't throw? What if it just passes zero to its normal continuation? Well, the normal continuation of A listed above is a call to B, so this passes zero to B. And as you can see from that, if B throws then it transfers control to C, and if B does not throw then it transfers control to D.

And there you go. **We just implemented try-catch and throw as methods.** Moreover, we implemented them both as *single line methods*. Apparently try-catch-throw is not very hard after all\!

You see what I mean about CPS being the reification of control flow? A continuation is an object that represents what happens next, and that’s what control flow is all about: determining what happens next. *Any conceivable control flow can be represented in CPS.*

Any conceivable control flow is rather a lot. This works because all possible control flows are simply fancy wrappers around a conditional “goto”. “if” statements are gotos, loops are gotos, subroutine calls are gotos, returns are gotos, exceptions are gotos, they’re all gotos. With continuations the non-local branching nature of any particular flavor of goto is completely irrelevant. You can implement some quite exotic control flows with CPS. Resumable exceptions, conditionals that evaluate both branches in parallel, yield return, coroutines. Heck, you can write programs that run forwards and then run themselves again *backwards* if you really want to. If it's a kind of control flow, then you can do it with CPS.

**Next time:** handwaving about coroutines.

