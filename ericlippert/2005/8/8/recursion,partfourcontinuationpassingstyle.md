# Recursion, Part Four: Continuation Passing Style

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/8/2005 6:00:00 AM

-----

We're getting hung up on the stack management aspects of recursive programming. **Why do we need a stack at all? What purpose does it serve?** If you're like most developers, you probably learned about subroutines and functions at an early age. The idea is pretty straightforward.  You stop what you're doing right now, go perform some other task, and then pick up where you left off, possibly using data computed by the other task.  But this idea of "pick up where you left off" means that you have to have some mechanism for storing information about what you were doing and where to pick it up. A stack is a natural data structure for that, so we've been thinking a lot about stacks lately when we consider how to operate on recursive data structures. But what if we simply denied the whole premise of functions? What if we said that **no function was ever allowed to return**?  It either terminates the program, or itself calls another function. You might be thinking that clearly the cure is worse than the disease.  I mean, sure, now you don't have to keep track of what you were doing, but that's because you're never going to get back there\!  How is it possible to get any work done at all if every time you try to call a procedure for a little help, it takes over control and never comes back to you? This sounds like crazy talk, but let's think about the consequences of removing all returns from the language. Three things are clear. First, if you yourself write a function then **calling another function has to be the last thing you do**. Control is never coming back, so there's no point in having any code after a function call. Second, suppose you have more work to do after you call a function foo. Foo is in the same position as you -- the last thing it does will either be a function call or terminating the program. Therefore, you need to put the work that needs to be done after foo into function bar, and make for darn sure that foo calls bar rather than terminating\! But if you didn't write foo, how do you know that foo is going to call bar when its done?  You need to somehow tell foo to call bar, and hope that foo cooperates. Third, if someone calls your function then they're in the same boat -- it's the last thing they're going to do, ever, because you're not coming back. You need to find some way to allow your caller to tell you what to do when you're done your work. This idea of functions never returning, and callers passing their functions information about what needs to be done next, is a bizarre but powerful style of programming called **Continuation Passing Style**. The "information about what to do next" is called a "continuation", which is passed from function to function, hence the name. Many languages support CPS natively -- Scheme, Ruby and Rhino, for example, all support CPS.  To get it working in JScript will take some doing, but when we're done there will be no explicit stack at all in our formerly recursive program. In the next episode I'm going to write our tree depth program in a mix of CPS and regular procedural programming. (To be purely CPS we'd write CPS versions of the addition operators, the Math.max function, and so on, but I'm not going to go there  -- that's overkill.)  But before we get into the tree depth program let's get a little more familiar with CPS. Suppose we had this fragment of a JScript program: function foo(x)  
{  
  var y = bar();  
  blah(x,y);  
}  
function bar()  
{  
  return 123;  
}  
function blah(a, b)  
{  
  print(a+b);  
}  
foo(1); I hope you agree that this is a very straightforward program. How would we do this in CPS?  Well, first of all, every function would have to take an extra argument for the continuation. Our program calls foo and then terminates, so the continuation of the call to foo is "terminate the program".  Let's presume that we have a magic function that does that. function foo(x, cfoo)  
{  
  // UNDONE: rewrite foo in CPS  
}  
foo(1, terminate); Let's reason about foo from end to start. foo wants to ensure that three things happen in this order:

  - bar runs
  - blah runs
  - foo's continuation from its caller runs

But blah is never going to return.  **Therefore we'll make it blah's responsibility to call foo's continuation when it is done.** We'll rewrite blah so that it does CPS, so that we can say: function foo(x, cfoo)  
{  
  var y = bar();  
  blah(x,y,cfoo);  
} OK, super, we've taken care of our responsibility to our caller by foisting it off onto blah.  But now we have an additional problem. bar is never going to return, so we're never going to call blah\!  We've both failed in our responsibility to our caller and haven't done the work we need to do. We have an additional problem. bar was going to return a value that we were going to use. But we've just removed returns from the language\! Let's solve both of these problems.  We'll say that bar takes a continuation and **passes the value that it was going to return as an argument to its continuation**. That's functionally the same thing, right? Before, you return the value to "whatever you were going to do next", now you pass the value to "whatever you were going to do next", same thing. But what continuation should foo pass to bar?  Well, what does foo want done when bar calls its continuation with its "return" value?  That's easy. It wants to call blah with that value and foo's original arguments. We can write a helper function that does that. function foo(x, afterfoo)  
{  
  function foocontinuation(y)  
  {  
    blah(x,y,afterfoo);  
  }  
  bar(foocontinuation);  
}  
function bar(afterbar)  
{  
  afterbar(123);  
}  
function blah(a, b, afterblah)  
{  
  print(a+b, afterblah); // print also rewritten in CPS  
} foo(1, terminate); foocontinuation is a [closure](http://blogs.msdn.com/ericlippert/archive/2003/09/17/53028.aspx), so it keeps track of what the values of x and afterfoo were when it was passed to bar.  So we're all set here.  The global code passes 1, terminate to foo   
foo passes foocontinuation to bar  
bar passes 123 to foocontinuation   
foocontinuation passes 1, 123, terminate to blah  
blah adds together its arguments and passes 124, terminate to print  
print presumably prints out 124 and calls terminate, and we're done Not once did any function return in there, and we did everything in the right order. Now of course in reality, JScript does not know that none of these functions are going to return. Nor is JScript smart enough to realize that even if they did return, none of these functions now does anything after the subroutine call, and therefore keeping track of the old frames on the stack is unnecessary, since they're never going to be read from again. If it did, we could totally write programs in this style and never worry about running out of stack space, but unfortunately, it doesn't. But just as we helped JScript along by writing our own explicit frame stack, we can help it along by writing our own CPS system. Stay tuned\! (Eric is on vacation; this message was prerecorded.)

