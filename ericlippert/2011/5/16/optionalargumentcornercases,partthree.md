# Optional argument corner cases, part three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/16/2011 7:50:00 AM

-----

(This is part three of a series on the corner cases of optional arguments in C\# 4; part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/05/12/optional-argument-corner-cases-part-two.aspx). Part four is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/05/19/optional-argument-corner-cases-part-four.aspx).)

A lot of people seem to think that this:

 

void M(string x, bool y = false) { ... whatever ... }

is actually a syntactic sugar for the way you used to have to write this in C\#, which is:

 

void M(string x) { M(x, false); }  
void M(string x, bool y) { ... whatever ... }

But it is not. The syntactic sugar here is not on the declaration side, but rather on the call side. There is only one method; when you make a call that omits optional arguments, the compiler simply inserts the arguments at the call site. That is:

 

M("hello");

is rewritten as

 

M("hello", false);

It would actually be pretty weird if we did it on the declaration side, because what would you do about this?

 

void N(bool a1 = false, bool a2 = false) { ... whatever ... }

Clearly we can't generate

 

void N() { N(false, false); }  
void N(bool a1) { N(a1, false); }  
void N(bool a2) { N(false, a2); }  
void N(bool a1, bool a2) { ... whatever ... }

because now we have two methods with the same signature. But why would we need to generate the "a2" method in the first place? Because we added **named arguments** as well as optional arguments. Someone could call:

 

N(a2: true);

Clearly we have to rewrite the caller side, not the called side.

An argument with a default value does not change the signature at all, so anything that relies upon signature matching still needs to have an exact match. That is, even though you can say:

 

M("hello");

and you can say

 

Action\<string\> action = (string s)=\>{M(s);};

You cannot say

 

Action\<string\> action = M;

Because M does not have a signature match with that delegate type; the delegate is expecting a method that takes a string, but you're giving it a method that takes a string and a bool. Where is the code generated that fills in the bool? There is no call site here, and the default value has to be filled in at the call site.

Similarly, you can't do something like this:

 

class B  
{  
  public virtual void M(string x, bool y = false) {}  
}  
class D : B  
{  
  public override void M(string x) {}  
}

Or this:

 

class D : B  
{  
  public override void M(string x, bool y = false, int z = 123) {}  
}

The signatures have to match to do an override, and default values for missing arguments are not part of the signature.

**Next time:** more consequences of call-site rewriting.

(This is part three of a series on the corner cases of optional arguments in C\# 4; part two is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/05/12/optional-argument-corner-cases-part-two.aspx). Part four is [here](http://blogs.msdn.com/b/ericlippert/archive/2011/05/19/optional-argument-corner-cases-part-four.aspx).)

