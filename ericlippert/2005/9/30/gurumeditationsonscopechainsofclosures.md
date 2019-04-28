# Guru meditations on scope chains of closures

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/30/2005 3:41:00 PM

-----

I just had a really interesting meeting with some of the scripting community MVPs.  I may write up some notes on that meeting here next week, so watch this space. In other news, someone asked me last night (and I quote)

> 

*Oh ECMA guru, can you provide a succinct explanation of why the following code works, but if I replace x with this it fails?* Number.prototype.times = function(f) {  
  for(var i = 0; i \< this; i++)  
    f();  
}  
Number.prototype.raiseTo = function(n) {  
  var answer = 1;  
  var x = this;  
  n.times(function(){ answer = **x** \* answer;}); // succeeds  
//n.times(function(){ answer = **this** \* answer;}); // fails\!  
  return answer;  
}  
(3).raiseTo(2).raiseTo(5); Well I'd hardly describe myself as a guru, but I take the questioner's point -- surely if you set x to this then x and this should be aliases for each other, right? Often. But not in this case. The inner function's execution context contains a scope chain that includes the variable object of the outer function, so the inner function can see x by looking up the scope chain. **But** **this** **is not a variable**, so it is not member of the outer function's variable object. Rather, it is a member of the outer function's execution context. Therefore the inner function's execution context gets the *global* this, in accordance with the ECMAScript specification, revision 3, section 10.2.3:

> *The caller provides the*

*this value. If the this value provided by the caller is not an object (including the case where it is null), then the this value is the global object.* The global object is not something that can be multiplied, so the alternate version of the program fails. The code above was of course not real production code, it was just code that came up while testing an engine implementation. Obviously JScript already has a built-in power function that works much better than this crazy thing. But I'm in an expansive mood, so next time I'll talk a bit about some of the shortcomings of this programming style, and some ideas for improving it.

