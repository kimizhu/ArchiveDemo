# Never Say Never, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/24/2011 6:31:00 AM

-----

Whether we have a "never" return type or not, we need to be able to determine when the end point of a method is unreachable for error reporting in methods that have non-void return type. The compiler is pretty clever about working that out; it can handle situations like

 

int M()  
{  
  try  
  {  
    while(true) N();  
  }  
  catch(Exception ex)  
  {  
    throw new WrappingException(ex);  
  }  
}

The compiler knows that N either throws or it doesn't, and that if it doesn't, then the try never exits, and if it does, then either the construction throws, or the construction succeeds and the catch throws. No matter what, the end point of M is never reached.

However, the compiler is not infinitely clever. It is easy to fool it:

 

int M()  
{  
  int x = 123;  
  try  
  {  
    while(x \>= x / 2) x = x / 2;  
  }  
  catch(Exception ex)  
  {  
    throw new WrappingException(ex);  
  }  
}

Here x is always going to be a small, non-negative integer, and a small, non-negative integer is always greater than or equal to half of it. You know that, I know that, but the compiler doesn't know that. The compiler complains that this method has a reachable end point, even though clearly the end point will never be reached. We could put that logic into the compiler if we really wanted to; we could be more clever.

How much more clever could we be? Is there any limit to our cleverness? Could we in theory produce a compiler that **perfectly** determined whether the end point of a method was reachable?

Turns out, no. A proof of that fact is a bit tricky, but we're tough, we can do it.

First off, let's restrict the problem to a particular pattern. Consider programs of this form:

 

class Program  
{  
  private static string data = @"some data";  
  public static int DoIt()  
  {  
     some code  
  }  
}

The question is *"given values for 'some data' and 'some code', does DoIt have a reachable end point?"* If we can't solve the problem for programs with this very restrictive format then we can't solve it in general, so let's explore whether we can solve problems in this limited format.

Let's suppose that we have a class in our compiler library with a public method that answers that question. We assume that "code" is a legal body of a C\# program and "data" is the legal contents of a string literal.

 

public class Compiler  
{  
  public static bool HasReachableEndPoint(string code, string data)  
  {  
    // \[Omitted: work out the result and return it\]  
  }  
}

And now things get really weird. What if we call:

 

string weird = @"  
if (Compiler.HasReachableEndPoint(data, data))  
  return 123;  
";  
bool result = Compiler.HasReachableEndPoint(weird, weird);

What does that do? It attempts to work out whether DoIt has a reachable end point in this program:

 

class Program  
{  
  private static string data = @"  
if (Compiler.HasReachableEndPoint(data, data))  
  return 123;  
";  
  public static int DoIt()  
  {  
     if (Compiler.HasReachableEndPoint(data, data))  
       return 123;  
  }  
}

What is "result"? Suppose it is true. Then that means that DoIt has a reachable end point. Which means that when we run DoIt, the call in the conditional statement returns false. But "data" and "weird" are *the same string*, so why is "result" true if the result of the same call is going to be false? That's logically inconsistent, so result cannot be true. [Clearly I cannot drink from the cup closest to me](https://www.youtube.com/watch?v=eQNHBUqfLnM).

Suppose "result" is false. That means that DoIt does not have a reachable end point. Which means that Compiler.HasReachableEndPoint(data, data) always either returns true, or throws an exception, or runs forever. But again, "data" and "weird" are the same string, so why would it return true, throw, or run forever here, when by assumption it returns false normally when given that input? Clearly result is not "false" either, since that leads to a contradiction. Clearly I cannot drink from the cup closest to you.

Since result can logically be neither true nor false, HasReachableEndPoint must either throw or go into an infinite loop when given these inputs. But if it does that then **there are some inputs for our reachability tester which cause the compiler to fail or run forever**, which seems bad. The compiler needs to be able to compile all legal programs and error on all illegal programs; ideally we don't want to crash, run forever, or give a wrong answer for any possible input.

What we've shown here is, first, that you should never go up against a Sicilian when death is on the line, and second, that an endpoint reachability tester logically must either (1) sometimes give the wrong answer, or (2) sometimes fail to give an answer. **The reachable endpoint problem is *in general* not reliably solvable by compilers no matter how clever the compiler writers are.**

Now, you might reasonably push back on this proof, noting that the proof relies upon an absolutely crazy situation, namely, a program that *contains part of its own code as data* that itself calls the reachability analyzer in what Douglas Hofstadter calls a "strange loop". Even if you don't like the proof though, we have other evidence that a "perfect" reachable end point detector that always returns a correct result in finite time is probably impossible. Consider for example this totally trivial little method, with just five loops and some calculations on an arbitrary-precision integer type:

 

public static int DoIt()  
{  
  Integer max = 0;  
  while(true)  
  {  
    max += 1;  
    for (Integer x = 1; x \<= max; ++x)  
      for (Integer y = 1; y \<= max; ++y)  
        for (Integer z = 1; z \<= max; ++z)  
          for (Integer n = 1; n \<= max; ++n)   
            if (x.Power(n+2) + y.Power(n+2) == z.Power(n+2)) goto done;  
  }  
  done: ;  
  // Uh oh, we "forgot" to return an int.  
  // But that's only a problem if the label is reachable.  
}

Want to know if Fermat's Last Theorem is true? Just run your magical perfect C\# compiler on a class containing that method and see if it compiles. If it fails with a "reachable end point in non-void-returning method" error then the goto must have been reachable, which means that there is a non-trivial solution to the equation, and Fermat's Last Theorem is false. If compilation succeeds (with a warning that there's an unreachable label\!) then the goto was unreachable and the theorem is true. To answer the question you don't even need to run *the program*, you just have to run the compiler\! The fact that this program is insanely inefficient in the amount of recalculation it performs is irrelevant, since we're not going to even run it.

**Such a compiler would enable us to answer any open question in finite mathematics** simply by writing a program that terminates if the hypothesis is true and runs forever if it is not. It seems [inconceivable](https://www.youtube.com/watch?v=D58LpHBnvsI) that we could even in theory construct a compiler that had the ability to answer all unsolved problems in the entire history of finite mathematics.

This famous problem is called the "Halting Problem" because it is usually posed for computational systems that do not have "throw an exception" as an option. The only alternatives are "halt with a result", or "go into an infinite loop".

The Halting Problem is of course solvable by heuristics *provided that you can tolerate false positives*. The C\# compiler will cheerfully tell you if the end point of a method is absolutely *guaranteed* to be unreachable, but as we've seen, you can fool it into telling you that some unreachable end points are reachable. As long as it does not have false negatives (that is, sometimes tells you that the reachable end point of a method is unreachable) everything is fine.

