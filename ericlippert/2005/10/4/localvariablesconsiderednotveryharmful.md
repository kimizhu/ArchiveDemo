# Local variables considered not very harmful

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/4/2005 1:22:00 PM

-----

Like I said, that code from [last time](http://blogs.msdn.com/ericlippert/archive/2005/09/30/475826.aspx) was just test code, not real production code. Though clearly it works, I'd never write code like that in a million years. I'm not thrilled with the way it uses the answer variable of the outer scope as an accumulator, and it is profoundly weird that the "do this function n times" function is a member of the Number prototype and not the Function prototype.  Worse still, we have a function that is essentially used as a composition operator that only makes sense to use on functions that have side effects and no arguments or returns. That's a kind of crazy way to do composition operations. I'd probably do something more like this: First, define a self-composition *factory* on the Function prototype, so that it is of general use on all functions of one argument, not just functions of no arguments and no returns that have side effects. What we'll do is make a new member function for every function object that returns a new function which is equivalent to calling the input function n times: Function.prototype.times = function(n) {  
  var f = this;  
  return function (x) {  
    while (n \> 0) {  
      x = f(x);  
      --n;  
    }  
    return x;  
  };  
} Obviously we don't need to compose functions of more than one argument because we want to feed the *single* output back into the input.  Now we compose the function "multiply something by x" with itself n times. That gives us the function "multiply something by x, n times".  Pass 1 to that function and we have a handy raiseTo function: Number.prototype.raiseTo = function(power) {  
  var x = this;  
  return function(y) {  
    return x \* y  
  }.times(power)(1);  
} No fuss, no muss, no accumulators, no side effects, no non-number-related functions on the number prototype. I like it\! Heck, if we wanted to get *really* crazy then we could eliminate the local variables entirely by introducing another function constructor in-line: Function.prototype.times = function(n) {  
  return function(f) {   
    return function (x) {  
      while (n \> 0) {  
        x = f(x);  
        --n;  
      }  
      return x;  
    }  
  }(this);  
} Number.prototype.raiseTo = function(power) {  
  return function(x) {  
    return function(y) {  
      return x \* y  
    }  
  }(this).times(power)(1);  
} Local variables are actually an unnecessary syntactic sugar in Jscript.  We can do everything that local variables do by cleverly nesting function scopes. However, it becomes *somewhat* less easy to read, so I would stick to using local variables if I were you\!

