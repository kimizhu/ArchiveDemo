# Fun with Floating Point Arithmetic, Part Five

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/20/2005 10:56:00 AM

-----

I went to [Joel Spolsky's](http://www.joelonsoftware.com)geek dinner at Crossroads the other night, which was a lot of fun. I didn't get much of a chance to chat with Joel, as he was surrounded by a cadre of adoring fans three deep the whole time.  I mostly hung out with [KC](http://blogs.msdn.com/kclemson/)and [Larry](http://blogs.msdn.com/larryosterman/)and some other attendees and had an interesting talk about digital rights management, corporate blogging, and the difficulties of finding good quality Jackie Chan movies in the original Chinese. I ran into [Wesner Moise](http://wesnerm.blogs.com/net_undocumented/), who I first met long ago when he was working on Access or Excel or one of those Office kind of products but haven't really run into since. He was rather surprised that I remembered his name, but hey, how many Wesner Moises do you think I meet?  Anyway, coincidentally, Wesner has also been running a series on the perils of floating point and integer mathematics on his blog recently. Check it out\!  \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* I said a while back that floating point math is nothing like the math we're used to. Consider some of the properties that define real number addition. For instance: **closure**: x + y is a number  
**commutative**: x + y = y + x  
**unique zero**: a + b = a if and only if b = 0  
**associative**: (x + y ) + z = x + (y + z) and so on, and similarly for multiplication. Commutivity still holds, but many of these rules do not work in floating point arithmetic.  I'll dig into a few of them here -- for more math rules that don't work in floating point, see Wesner's blog articles on the subject.  He lists dozens of them\! Consider the closure property, for example. It's not true in VBScript: print 10^308 + 10^308 ' Overflow error It is true in JScript, if you consider Infinity to be a number. The commutative property is true in both VBScript and JScript, but the unique zero, and hence the associative property, are true in neither: print 10^20 = 10^20 + 5000 prints True -- So clearly, zero is not a unique number which, when added, results in the same value. (Zero is unique in that it is the only number when added to EVERY number, results in no change. But almost every number has multiple values which result in no change when added.) That means that the associative property goes out the window: print 10^20 + (5000 + 5000) = (10^20 + 5000) + 5000 prints False The fact that the order in which you make the additions can affect the result makes a difference if you are designing algorithms that must add up lots of little things to one big thing. In those cases, **you should try to add together all the little things first, and then add the total to the big thing.** That way, the small additions are done with the most precision possible. There's a related error due to rounding off. 1/100 cannot be represented perfectly accurately in binary any more than 1/3 can be represented perfectly in decimal: var steps = 100;  
var start = 10;  
var stop = 11;  
var current = start;  
do  
{  
  print(current);  
  current = current + (stop-start)/steps;  
} while (current \< stop) Which ends up with ...  
10.96999999999998  
10.979999999999979  
10.989999999999979  
10.999999999999978 This actually takes 101 steps, because the last one through the loop gets to 10.999999999999978, which is less than 11.0.  This tiny accumulated error results in the algorithm running for 1% too many steps. A better algorithm is to do the looping in integers and compute the current value anew every time: var steps = 100;  
var start = 10;  
var current;  
for (var step = 0; step \< steps ; ++step)  
{  
  current = start + step/steps;  
  print(current);  
} That actually runs the right number of steps. A corollary of this is that you should almost never compare two floating point numbers for equality, because you never know when some rounding error might have crept in. Rather, subtract them and look at the absolute difference.  For instance, if you know that x and y are positive floats that are likely to be close to each other, don't say if (x==y), say if (Math.abs(x-y) \< 0.0001) or if (Math.abs(x-y) \< 0.0000001 \* x) or whatever makes sense in your application.  (If you need to deal with NaNs and infinities, simple subtraction is anything but\!) We saw above that addition of a small number to a large number can lead to an erroneous result, but the error was extremely small.  An error of 5000 in a number as big as 10<sup>20</sup> is 0.00000000000002%, which ain't bad.  We'd expect that subtraction of two numbers very close to each other would also produce an erroneous result, but with a similar error percentage. We'd be wrong. Here's an example: Suppose you have to solve an arbitrary quadratic equation: A x<sup>2</sup> + B x + C = 0 The solutions are well known, so we can write a little subroutine: Sub SolveQuadratic(A, B, C)  
  Discriminant = B\*B-4\*A\*C  
  If Discriminant \< 0 Then  
    Print "No real solutions"  
  Else  
    Print (-B + Sqr(Discriminant)) / (2 \* A)  
    Print (-B - Sqr(Discriminant)) / (2 \* A)  
  End If  
End Sub  
SolveQuadratic 2, 5, -12 And sure enough, it prints out -4 and 1.5, done. What about SolveQuadratic 1, -10000000.0000001, 1 The correct solutions are 10000000 and 0.0000001, but this prints out 10000000 and 9.96515154838562E-08, yielding an error of nearly 3.5%\! We've got about **a hundred trillion times as much error** here as we did in the addition. Why? Because B and the root of the discriminant are *very, very* close to each other.  You only get 15 decimal places of accuracy, and we've used them all up. Therefore, the difference is going to be *very* inaccurate. Their sum, however, is going to be quite accurate, since they're of similar size and precision. Fortunately, in this example there's a trick. The product of the two solutions is always C / A, so we can use this fact to write a better algorithm: Sub SolveQuadratic(A, B, C)  
  Discriminant = B\*B-4\*A\*C  
  If Discriminant \< 0 Then  
    Print "No real solutions"  
    Exit Sub  
  End If  
  Soln1 = (-B - Sqr(Discriminant)) / (2 \* A)  
  Soln2 = (-B + Sqr(Discriminant)) / (2 \* A)  
  If Abs(Soln1) \< Abs(Soln2) Then Soln1 = Soln2  
  Soln2 = C / (A \* Soln1)  
  Print Soln1  
  Print Soln2  
End Sub Which produces a much more accurate result. But wait a minute -- addition is order-dependent, as we've seen. so are multiplication and division. Should that be ( C / A ) / Soln1 or C / (A \* Soln1) ? Or does it even matter? Figuring that out is left as an exercise for the reader\!

