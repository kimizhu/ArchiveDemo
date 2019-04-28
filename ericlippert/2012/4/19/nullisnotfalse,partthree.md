# null is not false, part three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/19/2012 7:03:00 AM

-----

Returning now to the subject at hand: we would like to allow user-defined "overloads" of the & and | operators in C\#, and if we are going to have & and | be overloadable, it seems desirable to have && and || be overloadable too.

But now we have a big design problem. We typically overload operators by making a **method**:

class C  
{  
  string s;  
  public C(string s) { this.s = s; }  
  public override string ToString() { return s; }  
  public static C operator +(C x, C y) { return new C(x.s + "+" + y.s); }  
}  
...  
Console.WriteLine(new C("123") + new C("456")); // "123+456"

But method arguments are eagerly evaluated in C\#. We can't very well say:

public static C operator &&(C x, C y)  { ... whatever ... } 

because when you cay

C c = GetFirstC() && GetSecondC();

that is going to be rewritten as something like:

C c = C.op\_ShortCircuitAnd(GetFirstC(), GetSecondC());

which obviously evaluates both operands regardless of whether the left hand is "true" or "false".

Of course in modern-day C\# we have a type which represents "perform this calculation that produces a result in the future, on demand"; that type is Func\<T\>. We could implement this as:

public static C operator &&(C x, Func\<C\> fy)  { ... whatever ... }

and now the rewrite becomes

C c = C.op\_ShortCircuitAnd(GetFirstC(), ()=\>GetSecondC());

The method can then invoke the delegate only if it decides that it needs to evaluate the right hand side.

That would totally work, though it has a couple of problems. First, it is potentially expensive; even if we never use it, we go to all the trouble of allocating a delegate. Second, C\# 1.0 did not have either lambdas or generic delegate types, so the whole thing would have been a non-starter back then.

What we settled on instead was to say that really there are two things going on here. First we must decide whether to evaluate the right hand side or not. If we do not evaluate the right hand side then the result can be the left hand side. If we do evaluate the right hand side then we combine the two evaluated sides using the non-short-circuiting operator.

It is that first operation -- decide whether or not to evaluate the right hand side -- that requires operator true and operator false. These are the "is this operand one that requires the right hand side to be implemented or not?" operators.

But now we face another problem. I said last time that we have a problem with the short-circuiting && and || operators when considering values other than true or false: namely, does x && y mean "evaluate y if and only if x is true", or "evaluate y if and only if x is not false"? Obviously for straight-up Boolean x and y, those are the same, but for nullable Booleans they are not. And for our user-defined operator they are not the same either. After all, if you already had an unambiguous way to convert your type to true or false, you would simply implement an implicit conversion to bool and use the built-in && and || operators.

The C\# 1.0 design team decided that the rule is "evaluate y if and only if x is not false". We implement that rule by having an "operator false" that returns true if x is to be treated as false, and false if it is not to be treated as false. That's a bit confusing, I know. Maybe an example will help:

class C  
{  
  string s;  
  public C(string s) { this.s = s; }  
  public override string ToString() { return s; }  
  public static C operator &(C x, C y) { return new C(x.s + "&" + y.s); }  
  public static C operator |(C x, C y) { return new C(x.s + "|" + y.s); }  
  public static bool operator true(C x) { return x.s == "true"; }  
  public static bool operator false(C x) { return x.s == "false"; }  
}  
...  
C ctrue = new C("true");  
C cfalse = new C("false");  
C cfrob = new C("frob");

Console.WriteLine(ctrue && cfrob); // true\&frob  
Console.WriteLine(cfalse && cfrob); // false  
Console.WriteLine(cfrob && cfrob); // frob\&frob

That is to say, x && y here is implemented as:

C temp = x;  
C result = C.op\_false(temp) ? temp : temp & y;

And similarly, x || y uses the "operator true" in the analogous manner.

A little known fact is that if you can use && and || with a user-defined type, then you can also use them in control flows that take a bool, like if and while. This means that you can in fact say:

if (ctrue && cfrob) ...

because that becomes the moral equivalent of:

C temp = ctrue;  
C result = C.op\_false(temp) ? temp : temp & cfrb;  
bool b = C.op\_true(result);  
if (b) ...

Pretty neat, eh?

