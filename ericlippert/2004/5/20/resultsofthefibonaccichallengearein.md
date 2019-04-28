# Results Of The Fibonacci Challenge Are In

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/20/2004 5:11:00 PM

-----

Another bunch of good replies to my challenge of yesterday.  And again, a bunch of answers that I didn't expect, and some of the points that I was thinking of weren't mentioned. 

People seem to like this more conversational format; I'll probably use it more in the future. 

Three readers had conjectures as to the running time of fib\_1, the recursive algorithm.  Deriving the asymptotic order is pretty straightforward.  There must be some function f(n) such that fib\_1 is O(f(n)).  Well, what are the costs?  It's trivially true that fib\_1(1) is O(1), and so is fib\_1(2).  Let's therefore say that f(1) = 1 and f(2) = 1. 

What's the cost of the algorithm for large n?  Well, there's a constant bit at the beginning with the comparison, and a constant cost in the addition, but we can neglect those.  The real work is done in the recursion.  Clearly O(f(n)) = O(f(n-2)) + O(f(n-1)) 

That looks familiar\! Apparently f(n) = fib(n), so fib\_1 is O(fib(n)). 

We’ve established the truth of Frederik Slijkerman's comment that **recursive fib has a running time proportional to the size of its output**.  Steven Bone conjectured that recursive fib was O(2<sup>n</sup>).  He was also right -- almost.  How is that possible? 

The definition of the Fibonacci numbers that I gave is called a "recurrence relation".  Let me define two more recurrences: 

s(1) = s(2) = 1, s(n) = s(n-2) + s(n-2)  
f(1) = f(2) = 1, f(n) = f(n-2) + f(n-1)  
b(1) = b(2) = 1, b(n) = b(n-1) + b(n-1) 

You agree that it is clear that s(n) \<= f(n) \<= b(n), right?  The first sequence is 1, 1, 2, 2, 4, 4, 8, 8, 16, 16...  which grows at O(SQRT(2)<sup>n</sup>).  The second sequence is 1, 1, 2, 4, 8, 16, 32, ...  which clearly grows at O(2<sup>n</sup>).  f is **sandwiched **between two functions that grow at O(something<sup>n</sup>) so it must also be O(something<sup>n</sup>)\!  

The exact constant doesn't really matter for our purposes. It's actually about O(1.6<sup>n</sup>).  Yes, I'm handwaving here.  A more precise definition of order notation would make it clear when two functions are asymptotically equivalent, but that's a subject for another day.  There are different notations for functions with asymptotic upper, lower, and strict approximations, but I don't want to get into that now. The point is that this thing grows way faster than any polynomial in the long run and, more important, grows **extremely quickly** even in the short run. 

Clearly fib\_1 consumes O(n) memory off the stack.  Equally clearly,  fib\_2 is O(n) and consumes O(1) memory.  I hinted that there was an O(1) solution.  Indeed there is, provided that we more carefully define the problem.  Before I get to it, I mentioned that I often ask about robustness in interviews.  

Clearly it's a good idea to make sure that n is one or larger; fib\_2 goes into an infinite loop if it isn't.  But that's certainly not all\!  What's the actual vs. desired output of 

fib\_2();  
fib\_2(1, 2, 3)  
fib\_2("2B||\!2B");  
fib\_2("hello, world\!");  
fib\_2(null);  
fib\_2(3.14159);  
fib\_2(new Date()); 

and so on?  None of those make much sense and some of them go into infinite loops. Remember, JScript isn't C++ in fancy dress.  It's not a statically typed language, so if you have type restrictions, you're going to have to write a bunch of runtime code, or use a different language. 

Furthermore, clearly there is a sensible lower bound of one.  Is there a sensible upper bound?  I note that 

fib\_2(77) --\>   5527939700884757  
fib\_2(78) --\>   8944394323791464  
fib\_2(79) --\>  14472334024676220 

How do we add together a number ending in 7 and a number ending in 4 and end up with an even number?  **We've run out of precision**.  A floating point number only has 53 bits of precision; as soon as the value exceeds 2<sup>53</sup>, we start dropping bits off the end and therefore start returning incorrect results.  I asked for a function that produced the nth Fibonacci number, not one that produced a **close approximation** of it\!  That's a point which a good interview candidate would clarify.  

Assuming that we really do need exact numbers, and assuming further that the caller is willing to accept the limitations on precision that an IEEE float imposes, this method only has 78 sensible inputs -- the first 78 positive integers.  Therefore it only has 78 possible outputs.  When I hear that a method only has 78 outputs and the inputs are integers, I like table driven solutions.  (If we were using some other language that had 64 bit or 32 bit integers, the limits would be different, but they'd still be small enough to make a table-driven solution feasible.) 

var fibarr = new Array(null, 1, 1, 2, 3, 5, /\*\[...\]\*/, 8944394323791464);  
function fib\_3(n)  
{  
  // \[appropriate checking here.\]  
  return fibarr\[n\];  
} 

That's certainly O(1). 

If those aren't sensible restrictions, then we have waaaaay more work to do.  We need to figure out what the restrictions are, then write a numeric computation library that handles numbers up to the precision we need, and so on.  That introduces a whole other set of costs which we'd have to analyze on a case by case basis.  If we really did have to handle arbitrarily big numbers then a well-implemented algorithm would probably end up being O(log n). Several people mentioned Binet's Formula, which we could use. However, the mathematics of all that is more than I want to tackle in today's blog entry; I have work to do. If the caller really, really wanted this to be as fast as possible in raw speed terms, not algorithmic complexity terms, then we have to start talking about what we can change.  Language?  Seems to me that a C++ or assembly language implementation is going to be faster all around.  What hardware are we stuck with?  How big is the processor cache?  Will the table fit into it?  Can we hint to the compiler that the table is read-only after it is constructed and eke out more speed somehow?  What are these numbers being used for anyway, and how do we know that this method is even the bottleneck? 

Finally, Dan Shappir mentioned tail recursion and lazy evaluation using a zipper function on an infinite list.  That's quite coincidental; a couple days ago coworker of mine showed me a problem from an old ACM contest he'd been working on for fun. The problem was to implement a particular subset of Haskell's infinite list and zipper function capabilities. Which got me thinking that there are lots of interesting programming ideas that I could blog about.  Perhaps I'll take up some of that stuff in the coming weeks.

