# Continuation Passing Style Revisited, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/21/2010 6:38:00 AM

-----

Good morning fabulous readers, let me just start by saying that this is going to get *really long* and *really complicated* but it will all pay off in the end. I’m also going to be posting on an accelerated schedule, more than my usual two posts per week. (It’ll eventually become clear why I'm doing all of this, he said mysteriously. Remember, *suspense is a sign of a quality blog*.)

I want to talk quite a bit about a subject that I first discussed briefly a few years back, namely, **Continuation Passing Style** (henceforth abbreviated **CPS**), a topic which many people (\*) find both confusing and fascinating.

Before going on to read the rest of this series you might want to quickly refresh your memory of my brief introduction to CPS in JScript.

<http://blogs.msdn.com/b/ericlippert/archive/tags/continuation+passing+style/>

Welcome back. I hope that made sense. The JScript syntax for nested anonymous functions is pretty straightforward and similar to that of C\# anonymous methods, so hopefully it was clear even if you don’t normally read JScript. In case you didn’t make it all the way through there, let me sum up:

The traditional style of programming with subroutines provides a programming model that goes like this:

  - make a note of what you were doing and what values your local variables had in some sort of temporary storage, aka "the stack"
  - transfer control to a subroutine until it returns
  - consult your notes; pick up where you left off, now knowing the result of the subroutine if there was one.

CPS is a style of programming in which there are no “subroutines” per se and no “returns”. Instead, the **last** thing that the current function does is call the next function, passing the result of the current function to the next function. Since no function ever "returns" or does work after it calls the next function, there’s no need to keep track of where you’ve been. It doesn’t matter where you were because you’re never coming back there. In order to ensure that things happen in the desired order, when calling the new function you typically pass a “continuation”, which is itself a function that executes “everything that comes next”.

I showed a way to hack this up in JScript. You make every function take a continuation. New continuations can be built as needed out of nested anonymous functions. Any time that you would have called a subroutine, instead you make a continuation that represents the work you still have yet to do, and logically pass that to the subroutine, so that it can execute it for you. In order to do this without consuming any stack in JScript, you can do this by passing the continuation to some sort of "orchestration" code that keeps track of what has to happen next. The orchestrator just sits there in a loop, dispatching the current continuation with the last-computed result. In languages that do not support CPS natively that’s pretty much the best you can do if you want to do full-on CPS programming without the consumption of stack.

CPS is interesting and useful because it has a *lot* of nice properties, many of which I did not explain in my first series of articles. I presented CPS solely as a way to deal with the problem of deep recursions; since there are no subroutine calls that ever return, there is no need to consume call stack. But CPS is about much more than just that. In particular, one of the things CPS lets us do is **build new control flow primitives** into a language by implementing control flow as **methods**. That might sound crazy, but let’s take a look at a very simple example of how we might build a control flow out of continuations.

Consider for example the ?: conditional operator. It makes a decision about what happens next and therefore is a form of control flow. Suppose we have methods string M(int x), bool B(), int C(), and int D(). We might have this fragment somewhere in our program:

 

M(B() ? C() : D())

Suppose now that the C\# language did not have the ?: operator and you wanted to implement it as a library method call. You can’t just go:

 

T Conditional\<T\>(bool b, T consequence, T alternative)  
{  
  if (b) return consequence; else return alternative;  
}

because now the consequence and alternative are evaluated eagerly, instead of lazily. But we could do this:

 

T Conditional\<T\>(bool b, Func\<T\> consequence, Func\<T\> alternative)  
{  
  if (b) return consequence(); else return alternative();  
}

And now call

 

M(Conditional(B(), ()=\>C(), ()=\>D()))

We have our conditional operator implemented as a library method.

Now suppose we wanted to do the conditional operator in CPS because... because we're gluttons for punishment I guess. In CPS we have some continuation; something is going to happen after the call to M. Let’s call that the "current continuation", whatever it is. How we obtain it is not important, just suppose we have it in hand.

We need to rewrite B so that it takes a continuation that accepts a bool. “A continuation that takes a bool” sounds an awful lot like an Action\<bool\>, so let’s assume that we can rewrite bool B() to be void B(Action\<bool\>).

What is the continuation of B, the "thing that happens after"? Let’s take it one step at a time.

 

B(b=\>M(Conditional(b, ()=\>C(), ()=\>D)))

We have B in CPS, but the lambda passed to B is not in CPS because it does two things: calls Conditional, and calls M. To be in CPS it has to call something as the last thing it does. Let’s repeat the analysis we just did for B. M needs to take an int and an Action\<string\>. C and D need to take Action\<int\>. What about Conditional? It still needs to lazily evaluate its arguments, but calling those lambdas cannot return a value either; rather, they have to take continuations too:

 

B(b=\>Conditional(b, c=\>C(c), c=\>D(c), t=\>M(t, currentContinuation)))

Conditional now looks like this:

 

void Conditional\<T\> (bool condition, Action\<Action\<T\>\> consequence, Action\<Action\<T\>\> alternative, Action\<T\> continuation)  
{  
  if (condition) consequence(continuation) ; else alternative(continuation);  
}

Summing up: B executes and passes its bool result to a lambda. That lambda passes b and three different lambdas to Conditional. Conditional decides which of the first two delegates to pass the third delegate - the continuation - to. Suppose it chooses the alternative. The alternative, D, runs and passes its result to the continuation, which is t=\>M(currentContinuation, t). M then does its thing with integer t, and invokes whatever the current continuation of the original call was, and we’re done.

There are no more returns. Every method is void. Every delegate is Action. The last thing any method does is invokes another method. We no longer need a call stack because we never need to come back. If we could write a C\# compiler that optimized code for this style of programming then none of these methods would consume any call stack beyond their immediate requirements to store locals and temporaries.

And, holy goodness, what a mess we turned that simple operation into. But you see what we did there? I defined a CPS version of the ?: operator as a library method; the fact that the consequence and alternative are lazily evaluated is accomplished by rewriting them as lambdas. The control flow is then accomplished by passing the right continuation to the right method.

But so what? We’ve done nothing here that couldn’t have been done much more easily without continuations. Here’s the interesting bit: **continuations are the [reification](http://blogs.msdn.com/b/ericlippert/archive/2009/04/17/five-dollar-words-for-programmers-part-five-reification.aspx) of control flow.** So far we’ve only been talking about systems where there is a single continuation being passed around. Since continuations are effectively just delegates, there’s no reason why you can’t be passing around multiple continuations, or stashing them away for later use. By doing so we can construct arbitrarily complex control flows *as library methods* by keeping track of multiple continuations and deciding which one gets to go next.

Next time: some musings on more complicated control flows.

(\*) Including myself.

