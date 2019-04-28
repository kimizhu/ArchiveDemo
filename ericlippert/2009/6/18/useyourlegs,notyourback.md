# Use your legs, not your back

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/18/2009 9:32:00 AM

-----

In C\# you can "lift", "raise" and "hoist", and they all mean different things.

To "lift" an operator is to take an operator that operates on non-nullable value types, and create from it a similar operator that operates on nullable value types. (We are a little bit inconsistent in exactly how we use the word "lifted", which I documented [here](http://blogs.msdn.com/ericlippert/archive/2007/06/27/what-exactly-does-lifted-mean.aspx).)

For example, if you have

 

 public static Complex operator +(Complex x, Complex y) { ... }

 then we automatically generate a lifted operator for you that basically does this:

 

 public static Complex? operator +(Complex? x, Complex? y)   
 {  
   return (x == null || y == null) ?  
    (Complex?) null :   
    (Complex?) (x.Value + y.Value);  
 }

["Raising" by contrast refers to events](http://msdn.microsoft.com/en-us/library/wkzf914z\(VS.71\).aspx) -- not to exceptions, which are of course "thrown". Another common term for raising an event is "firing". Given that it makes sense to standardize on one or the other, the usage committee people felt that between "raising" and "firing", they'd pick the less bellicose-sounding one. Which is maybe a bit silly, but if you've got to pick one, then I suppose that's as good a criterion as any.

Finally, "hoisting" is what we call it when the compiler emits a field for what looks like a local variable, because that local variable is in fact a captured outer variable of an anonymous function (or a local of an iterator block). When you have:

 

class C  
{  
  void M()  
  {  
    int x = 123;  
    Func\<int, int\> f = y=\>x+y;  
...

then we rewrite that as if you'd written something like:

 

class C  
{  
  private class Locals  
  {  
    public int x;  
    public int Method(int y) { return this.x + y }  
  }  
  void M()  
  {  
    Locals locals = new Locals();  
    locals.x = 123;  
    Func\<int, int\> f = locals.Method;  
...

 see, local "x" has been "hoisted" up and out of its declaration space.

Even after a number of years on the compiler team, I still misuse "raise", "lift" and "hoist" in casual conversation; that they have such similar meanings in English and dissimilar meanings as jargon is unfortunate, but usually doesn't result in too much confusion.

