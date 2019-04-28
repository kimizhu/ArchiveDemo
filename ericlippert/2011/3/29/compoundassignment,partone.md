# Compound Assignment, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/29/2011 6:24:00 AM

-----

When people try to explain the compound assignment operators += –= \*= /= %= \<\<= \>\>= &= |= ^= to new C\# programmers they usually say something like “x += 10; is just a short way of writing x = x + 10;”. Now, though that is undoubtedly true for a local variable x of type int, that’s not the whole story, not by far. There are actually many subtle details to the compound assignment operators that you might not appreciate at first glance.

First off, suppose the expression on the left hand side has a side effect or is expensive to call. You only want it to happen once:

 

class C  
{  
  private int f;  
  private int P { get; set; }  
  private static C s = new C();  
  private static C M()  
  {   
    Console.WriteLine("Hello");  
    return s;  
  }  
  private struct Evil  
  {  
      public int f; // Mutable value type with a public field, evil\!  
      public int P { get; set; }  
  }  
  private static Evil\[\] evil = new Evil\[2000\];  
  private static Evil\[\] N()  
  {  
    Console.WriteLine("Badness");  
    return evil;  
  }  

If somewhere inside C you have M().f += 10; then you only want M’s side effect to happen once. This is not the same as M().f = M().f + 10;

What is it the same as then? How about this:

 

C receiver = M();  
receiver.f = receiver.f + 10;

Is that right? It seems to be, but suppose we make it a bit more complicated. Suppose we have N()\[123\].f += 10; . Is this then

Evil receiver = N()\[123\];  
receiver.f = receiver.f + 10;

Clearly not.  We've made a **copy** of the contents of variable N()\[123\] and **we are now mutating the variable containing the copy but we need to be mutating the original**.

Once more we see how much pure concentrated evil mutable value types are\!

To express the real semantics **concisely** we need a feature that C\# does not have, namely, “ref locals”. C\# has ref-typed **parameters**, but not ref-typed **locals**. When you make a ref-typed parameter essentially you are saying “this parameter is an alias for this variable”:

 

    void N(ref int x) { x = 10; }  
    …  
    N(ref M().f);

That says “Evaluate the expression as a variable and then make the variable x refer to the same storage location as the variable”. Suppose we had the ability to do that with locals instead of just parameters. That is, **we can make a local variable that is an alias for a (possibly non-local) variable.** Then M().f += 10 would be equivalent to:

 

ref int variable = ref M().f;  
variable = variable + 10;

And thus the side effect of M only happens once. Similarly  N()\[123\].f += 10; where the array is of mutable value type becomes

ref int variable = ref N()\[123\].f;  
variable = variable + 10;

and the side effect of N only happens once, and we mutate the field of the correct variable.

C\# does not have the “ref local” feature though we could implement it if we wanted to; the CLR supports it. I think we have higher priorities though.

What if instead of a **variable** we modified a **property**?

 

M().P += 10;

You might again think that this is a a syntactic sugar for

 

C receiver = M();  
receiver.P = receiver.P + 10;

which is of course a syntactic sugar for:

 

C receiver = M();  
receiver.set\_P(receiver.get\_P() + 10);

Again, we only want the side effect exercised once, though of course we have to call two different methods for the getter and the setter; that’s unavoidable.

But again, we have a problem if the receiver is a variable of value type. If we have  N()\[123\].P += 10; then we have to generate

ref Evil receiver = ref N()\[123\];  
receiver.set\_P(receiver.get\_P() + 10);

So that we make sure that the mutable value type property we're invoking is on the right variable.

Similarly if we had an **indexer** defined on C:

 

M()\[X()\] += 10;

Now we have to keep track of both the receiver and the index to make sure they are not evaluated twice. That’s the same as:

 

C receiver = M();  
int index = X();  
receiver\[index\] = receiver\[index\] + 10;

and of course just as with properties, **those too are just syntactic sugars for calls**, and again, **we need to make sure we get the refness right** if the receiver is a variable of a mutable value type.

And similarly with += –= on events, though of course those are different because they are syntactic sugars for event add and remove methods.

Anyway, I don’t think I need to further belabour the point that **side effects are only computed once** and that **determining the correct location to mutate is not as easy as you might think**.

Another interesting aspect of the **predefined** compound operators is that if necessary, a cast – an allegedly “explicit” conversion – is inserted *implicitly* on your behalf. If you say

 

short s = 123;  
s += 10;

then that is not analyzed as s = s + 10 because short plus int is int, so the assignment is bad. This is actually analyzed as

 

s = (short)(s + 10);

so that if the result overflows a short, it is automatically cut back down to size for you.

A final subtle point is that **for the predefined operators** if the assignment without the compounded operation would not have been legal, then the compound assignment is not legal either. If you say

 

int i = 10;  
short s = 123;  
s += i;

then that’s not legal because s = i is not legal.

Those design details are interesting in of themselves; next time we’ll see how some of these subtleties affect some proposed extensions to the language.

