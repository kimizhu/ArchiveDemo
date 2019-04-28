# Continuation Passing Style Revisited Part Three: Musings about coroutines

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/25/2010 6:40:00 AM

-----

Last time I sketched briefly how one might implement interesting control flows like try-catch using continuations; as we saw, the actual implementations of Try and Throw are trivial once you have CPS. I'm sure that you could extend that work to implement try-catch-finally. Or, another basic exercise when learning about CPS you might try is to implement **coroutines**. What’s a coroutine? Excellent question\! Cast your mind back to the days of cooperative multitasking in Windows 3. The idea of cooperative multitasking is the operating system would allow a process to run, and the process would decide when to pause itself and allow another process to run. If a process got an infinite loop then it could starve all the other processes of processor time indefinitely. Programs had to be carefully designed so that they did not take up a lot of cycles before yielding control back to the OS, which would then choose the next process to run. Of course, nowadays we have non-cooperative multitasking built into the operating system at both the process and thread level. The operating system, not the process, decides when a particular thread in a particular process needs to pause. This means that programs do not have to be written specially to be nice to other programs; they can just be written the straightforward way without worrying that other processes will starve. However, it means that the operating system has to be able to rapidly grab all the state in the CPU that is relevant to a particular thread, save it for later, and restore it so that the thread picks up where it left off without ever being the wiser. (In fact, the operating system is essentially storing the *continuation* of the thread\!) Coroutines are a programming language concept very similar to cooperative multitasking at the OS level. A coroutine is a method that runs for a little while, and then decides to be nice, pause itself, and allow some other coroutine to run. When some other coroutine returns the favour, the first one picks up exactly where it left off. One imagines any number of systems that could make use of this technique:  

Stack\<Pancakes\> pancakes = new Stack\<Pancakes\>();  
coroutine MakePancakes(Chef chef)  
{  
    while(true)  
    {  
        while (chef.IsAwake)  
            pancakes.Push(chef.MakePancake());  
        yield;  
    }  
}  
  
coroutine EatPancakes(Child child)  
{  
    while(true)  
    {  
        while (child.IsHungry && pancakes.Count \!= 0)  
            child.Eat(pancakes.Pop());  
        yield;  
    }  
}

This is way better than mutual recursion. Clearly you don't want to be calling "MakePancakes()" and "EatPancakes()" in there because (1) that makes for an infinite recursion, and (2) there might be multiple chefs and multiple children who get turns in different orders. Rather, you want to say "*I'm done working on this task for now. Someone else can run, but I need to pick up right here when it is my turn again*". Since this is cooperative single-threaded multitasking, no two coroutines ever run at the same time, or ever are suspended halfway through an operation. There's no "thread safety" problems with two routines sharing the same global pancake stack.

The tricky bit is, of course, how to achieve "pick up right here when it is my turn again." In Windows you can use [fibers](http://blogs.msdn.com/b/ericeil/archive/2008/05/18/when-does-it-make-sense-to-use-win32-fibers.aspx) to implement coroutines, because basically that's all a fiber is: a "lightweight" thread that decides when it yields control to another fiber, instead of allowing the OS to decide. But let’s ignore that. Suppose we didn’t have fibers(\*) but we did have continuations.

From what you know about continuations thus far it should be clear that coroutines can be implemented quite easily in any programming language that supports CPS. It is particularly easy if you can get a little help from whatever code is orchestrating the invocation of the next continuation, but not strictly necessary. When it is time to yield control, you just tell the orchestrator “I am a coroutine who is yielding now. Here is my current continuation. Please put it at the end of a queue. Go run whatever coroutine’s continuation is on the top of the queue.” If everyone cooperates then each coroutine with pending work runs for a little while and then gives the next guy a turn. (And of course, when the coroutine completes, if it does, then it simply never puts a continuation on the queue and it vanishes.) Since "everything I need to do next" is precisely what a continuation is defined as, we've already solved the problem of figuring out how to pick up where we left off. As you’ve no doubt just realized, [if you didn't know it already](http://blogs.msdn.com/b/ericlippert/archive/2009/07/09/iterator-blocks-part-one.aspx), **the “yield return” statement in C\# is a weak form of coroutine**. When you hit a “yield return”, conceptually the enumerator stores away its continuation – that is, enough information to know how to pick up where it left off. It then yields control back to its caller, which decides when to call MoveNext() again and pick up where the iterator block left off. The iterator block and the caller have to cooperate; the iterator block promises to give the next item back in a timely manner, and the caller promises to call either MoveNext() or Dispose() to allow the iterator to either run its next piece of work or clean up after itself. Of course, as I noted last year, iterator blocks are not *really* implemented in "pure" Continuation Passing Style the way I've presented it thus far. We do not actually transform the entire method and every function call in it into CPS and build a lambda for “the stuff that comes next”. Because we are limiting ourselves to implementing coroutine-like data flow **solely for the purpose of building an implementation of IEnumerator**, we don’t have to pull the big hammer that is CPS out of our toolbox. We build a much simpler system, a state machine that keeps track of where it was, plus a closure that keeps track of what all the local variable values were. But in theory we could have written iterator blocks as coroutines, and we could have written coroutines by building them out of continuations. The “yield return” statement gives a fairly complex control flow, but we can build any complex control flow out of continuations. At this time it might be a good idea to read [Raymond’s articles](http://blogs.msdn.com/b/oldnewthing/archive/2008/08/12/8849519.aspx) on how iterators are actually implemented in C\#, and, optionally, the rest of [my articles on some of the consequences of those design decisions](http://blogs.msdn.com/b/ericlippert/archive/tags/iterators/). *Familiarity with that subject will be necessary presently*. (Remember, foreshadowing is a sign of a quality blog.) **Next time**: if Continuation Passing Style is so awesome then why don’t we all use this technique every day?

(\*) Or that we didn’t want to pay the price of a million bytes of stack space reserved by default for each fiber. Fibers aren't actually as lightweight as one might like.

