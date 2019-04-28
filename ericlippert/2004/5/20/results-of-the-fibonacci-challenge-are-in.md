<div id="page">

# Results Of The Fibonacci Challenge Are In

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/20/2004 5:11:00 PM

-----

<div id="content">

<span>Another bunch of good replies to my challenge of yesterday.  And again, a bunch of answers that I didn't expect, and some of the points that I was thinking of weren't mentioned. </span>

<span>People seem to like this more conversational format; I'll probably use it more in the future. </span>

<span>Three readers had conjectures as to the running time of </span><span>fib\_1</span><span>, the recursive algorithm.  Deriving the asymptotic order is pretty straightforward.  There must be some function </span><span>f(n)</span><span> such that </span><span>fib\_1</span><span> is </span><span>O(f(n))</span><span>.  Well, what are the costs?  It's trivially true that </span><span>fib\_1(1)</span><span> is </span><span>O(1)</span><span>, and so is </span><span>fib\_1(2)</span><span>.  Let's therefore say that </span><span>f(1) = 1</span><span> and </span><span>f(2) = 1</span><span>. </span>

<span>What's the cost of the algorithm for large </span><span>n</span><span>?  Well, there's a constant bit at the beginning with the comparison, and a constant cost in the addition, but we can neglect those.  The real work is done in the recursion.  Clearly </span><span>O(f(n)) = O(f(n-2)) + O(f(n-1)) </span>

<span>That looks familiar\! Apparently </span><span>f(n) = fib(n)</span><span>, so </span><span>fib\_1</span><span> is </span><span>O(fib(n))</span><span>. </span>

<span>We’ve established the truth of Frederik Slijkerman's comment that **recursive fib has a running time proportional to the size of its output**.  Steven Bone conjectured that recursive fib was </span><span>O(2<sup>n</sup>)</span><span>.  He was also right -- almost.  How is that possible? </span>

<span>The definition of the Fibonacci numbers that I gave is called a "recurrence relation".  Let me define two more recurrences: </span>

<span>s(1) = s(2) = 1, </span><span>s(n) = s(n-2) + s(n-2)  
</span><span>f(1) = f(2) = 1, </span><span>f(n) = f(n-2) + f(n-1)  
</span><span>b(1) = b(2) = 1, </span><span>b(n) = b(n-1) + b(n-1) </span>

<span>You agree that it is clear that </span><span>s(n) \<= f(n) \<= b(n)</span><span>, right?  The first sequence is </span><span>1, 1, 2, 2, 4, 4, 8, 8, 16, 16...</span><span>  which grows at </span><span>O(SQRT(2)<sup>n</sup>)</span><span>.  The second sequence is </span><span>1, 1, 2, 4, 8, 16, 32, ...</span><span>  which clearly grows at </span><span>O(2<sup>n</sup>)</span><span>.  </span><span>f</span><span> is **<span>sandwiched </span>**between two functions that grow at </span><span>O(something<sup>n</sup>)</span><span> </span><span>so it must also be </span><span>O(something<sup>n</sup>)</span><span>\!  </span>

<span>The exact constant doesn't really matter for our purposes. It's actually about </span><span>O(1.6<sup>n</sup>)</span><span>.  Yes, I'm handwaving here.  A more precise definition of order notation would make it clear when two functions are asymptotically equivalent, but that's a subject for another day.  There are different notations for functions with asymptotic upper, lower, and strict approximations, but I don't want to get into that now. The point is that this thing grows way faster than any polynomial in the long run and, more important, grows **extremely quickly** even in the short run. </span>

<span>Clearly </span><span>fib\_1</span><span> consumes </span><span>O(n)</span><span> memory off the stack.  Equally clearly,  </span><span>fib\_2</span><span> is </span><span>O(n)</span><span> and consumes </span><span>O(1)</span><span> memory.  I hinted that there was an </span><span>O(1)</span><span> solution.  Indeed there is, provided that we more carefully define the problem.  Before I get to it, I mentioned that I often ask about robustness in interviews.  </span>

<span>Clearly it's a good idea to make sure that </span><span>n</span><span> is one or larger; </span><span>fib\_2</span><span> goes into an infinite loop if it isn't.  But that's certainly not all\!  What's the actual vs. desired output of </span>

<span>fib\_2();  
</span><span>fib\_2(1, 2, 3)  
</span><span>fib\_2("2B||\!2B");  
</span><span>fib\_2("hello, world\!");  
</span><span>fib\_2(null);  
</span><span>fib\_2(3.14159);  
</span><span>fib\_2(new Date()); </span>

<span>and so on?  None of those make much sense and some of them go into infinite loops. Remember, JScript isn't C++ in fancy dress.  It's not a statically typed language, so if you have type restrictions, you're going to have to write a bunch of runtime code, or use a different language. </span>

<span>Furthermore, clearly there is a sensible lower bound of one.  Is there a sensible upper bound?  I note that </span>

<span>fib\_2(77) --\>   5527939700884757  
fib\_2(78) --\>   8944394323791464  
fib\_2(79) --\>  14472334024676220 </span>

<span>How do we add together a number ending in 7 and a number ending in 4 and end up with an even number?  **We've run out of precision**.  A floating point number only has 53 bits of precision; as soon as the value exceeds 2<sup>53</sup>, we start dropping bits off the end and therefore start returning incorrect results.  I asked for a function that produced the nth Fibonacci number, not one that produced a **<span>close approximation</span>** of it\!  That's a point which a good interview candidate would clarify.  </span>

<span>Assuming that we really do need exact numbers, and assuming further that the caller is willing to accept the limitations on precision that an IEEE float imposes, this method only has 78 sensible inputs -- the first 78 positive integers.  Therefore it only has 78 possible outputs.  When I hear that a method only has 78 outputs and the inputs are integers, I like table driven solutions.  (If we were using some other language that had 64 bit or 32 bit integers, the limits would be different, but they'd still be small enough to make a table-driven solution feasible.) </span>

<span>var fibarr = new Array(null, 1, 1, 2, 3, 5, /\*\[...\]\*/, 8944394323791464);  
</span><span>function fib\_3(n)  
</span><span>{  
</span><span>  </span><span>// \[appropriate checking here.\]  
</span><span>  </span><span>return fibarr\[n\];  
</span><span>} </span>

<span>That's certainly </span><span>O(1).</span><span> </span>

<span>If those aren't sensible restrictions, then we have waaaaay more work to do.  We need to figure out what the restrictions are, then write a numeric computation library that handles numbers up to the precision we need, and so on.  That introduces a whole other set of costs which we'd have to analyze on a case by case basis.  If we really did have to handle arbitrarily big numbers then a well-implemented algorithm would probably end up being <span>O(log n)</span>. Several people mentioned Binet's Formula, which we could use. However, the mathematics of all that is more than I want to tackle in today's blog entry; I have work to do.</span> <span>If the caller really, really wanted this to be as fast as possible in raw speed terms, not algorithmic complexity terms, then we have to start talking about what we can change.  Language?  Seems to me that a C++ or assembly language implementation is going to be faster all around.  What hardware are we stuck with?  How big is the processor cache?  Will the table fit into it?  Can we hint to the compiler that the table is read-only after it is constructed and eke out more speed somehow?  What are these numbers being used for anyway, and how do we know that this method is even the bottleneck? </span>

<span>Finally, Dan Shappir mentioned tail recursion and lazy evaluation using a zipper function on an infinite list.  That's quite coincidental; a couple days ago coworker of mine showed me a problem from an old ACM contest he'd been working on for fun. The problem was to implement a particular subset of Haskell's infinite list and zipper function capabilities. Which got me thinking that there are lots of interesting programming ideas that I could blog about.  Perhaps I'll take up some of that stuff in the coming weeks. </span>

<span></span> 

</div>

</div>

