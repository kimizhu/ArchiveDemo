# How Bad Is Good Enough?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/17/2003 2:05:00 PM

-----

I keep talking about script performance without ever actually giving my rant about why most of the questions I get about performance are pointless at best, and usually downright harmful.

 

 

Let me give you an example of the kind of question I've gotten dozens of times over the last seven years.  Here's one from the late 1990s:

We have some VBScript code that DIMs a number of variables in a well-used function.  The code never actually uses those variables and they go out of scope without ever being touched.  Are we paying a hidden price with each call?

What an interesting performance question\!  In a language like C, declaring n bytes total of local variables just results in the compiler generating an instruction that moves the stack pointer n bytes.  Making n a little larger or smaller doesn't change the cost of that instruction.  Is VBScript the same way?  Surprisingly, no\!  Here's my analysis:

 

 

Bad Analysis \#1

You dim it, you get it.  VBScript has no idea whether you’re going to do this or not: 

function foo()  
  Dim bar  
  Execute(“bar = 123”) 

In order to enable this scenario the script engine must** at runtime** bind all of the** names** of the local variables into a local binder. ** That causes an added per-variable-per-call expense.** 

(Note that JScript .NET **does** attempt to detect this scenario and optimize it, but that's another post.)

Anyway, what is the added expense?  I happened to have my machine set up for perf measuring that day, so I measured it:

On my machine,** every** additional variable which is dimensioned but not used adds a **50 nanosecond** **penalty** to** every** call of the function.  The effect appears to scale linearly with the number of unused dimensioned variables; I did not test scenarios with extremely large numbers of unused variables, as these are not realistic scenarios.  Note also that I did not test very *long* variable names; though VBScript limits variable names to 256 characters, there may well be an additional cost imposed by long variable names. 

My machine is a 927 MHz Pentium III, so that’s somewhere around fifty processor cycles each.  I do not have VTUNE installed right now, so I can’t give you an exact processor cycle count. 

That means that if your heavily used function has, say, five unused variables then** every four million calls to your function will slow your program down by an entire second**, assuming of course that the target machine is my high-end dev machine.  Obviously a slower machine may exhibit considerably worse performance.

However, you do not mention whether you are doing this on a server or a client.  That is extremely important when doing performance analysis\! 

Since the penalty is imposed due to a heap allocation, the penalty on the server may scale differently based on the heap usage of other threads running in the server.  There may be contention issues - my measurements measured only “straight” processor cost; a full analysis of the cost for, say, an 8 proc heavily loaded server doing lots of small-string allocations may well give completely different results.  

Now let me take this opportunity to tell you that **all the analysis I just described is almost completely worthless because it obscures a larger problem**.  There's an elephant in the room that we're ignoring.  The fact that a user is asking me about performance of VBScript tells me that either 

(a) this user is a hard-core language wonk who wants to talk shop, or, more likely, 

(b) **the user has a program written in script which he would like to be running faster.  The user cares deeply about the performance of his program.**

Whoa\!  Now we see why this perf analysis is worthless.  **If the user cares so much about performance then why is he using a late-bound, unoptimized, bytecode-interpreted, weakly-typed, extremely dynamic language specifically designed for rapid development at the expense of runtime performance?** 

Bad Analysis \#2

If you want a script to be faster then there are way more important things to be optimizing away than the 50-nanosecond items.  **The key to effective performance tuning is finding the most expensive thing and starting with that.**  A **single** call that uses an undimensioned variable, for example, is hundreds of times more expensive than that dimensioned-but-unused variable.  A single call to a host object model method is thousands of times more expensive. Optimizing a script by trimming the 50 ns costs is like weeding your lawn by cutting the already-short grass with nail scissors and ignoring the weeds**.  It takes a long time, and it makes no noticeable impact on the appearance of your lawn.**  It epitomizes the difference between "active" and "productive". Don't do that\!   

But even better advice that that would be to **throw away the entire script and start over in C if performance is so important.** 

Now, let me just take this opportunity to interrupt myself and say that yes, **script performance is important**.  We spent a lot of time optimizing the script engines to be pretty darn fast for late-bound, unoptimzed, bytecode-interpreted, weakly-typed dynamic language engines. Eventually you come up against the fact that you have to pick the right tool for the job -- VBScript is as fast as its going to get without turning it into a very different language or reimplementing it completely.

Unfortunately, this second analysis is hardly better than the first, because again, there is an elephant in the room.  There's a vital piece of data which has not been supplied, and that is the key to all perf analysis:

How Bad Is Good Enough?

 

I was going easy on myself -- I actually consider this sort of "armchair" perf analysis to be not worthless,  I consider it to be actively harmful.  

 

 

I've read articles about the script engines that say things like "you should use And 1 to determine whether a number is even rather than Mod 2 because the chip executes the And instruction faster", as though VBScript compiled down to tightly optimized machine code.  People who base their choice of operator on utterly nonsensical rationales **are not going to write code that is** **maintainable or robust.  Those programs end up broken, and "broken" is the ultimate in bad performance, no matter how fast the incorrect program is.** 

 

 

If you want to write fast code -- in script or not -- then ignore every article you ever see on "tips and tricks" that tell you which operators are faster and what the cost of dimensioning a variable is.  Writing fast code does not require a collection of cheap tricks, it requires analysis of user scenarios to set goals, followed by a rigorous program of careful measurements and small changes until the goals are reached.

 

1\) Have a user-focussed plan.  Know what your performance goals are.

2\) Know what to measure to test whether those goals are met.  Are you worried about throughput?  Time to first byte?  Time to last byte?  Scalability?

3\) Measure the whole system, not just isolated parts.

4\) Measure carefully and measure often.

 

That's what the MSN people do, and they know about scalable web sites.

  

I know that's not what people want to hear.  People have these ideas about performance analysis which as far as I can tell, last applied to PDP-11's.  Script running on web servers cannot be optimized through micro-optimization of individual lines of code -- it's not C, where you can know the exact cost of every statement.  With script you've got to look at the whole thing and attack the most expensive things.  Otherwise you end up doing a huge amount of work for zero noticable gain.

 

 

But you've got to know what your goals are.  Figure out what is important to your users.  Applications with user interfaces have to be snappy -- the core processing can take five minutes or an hour, but a button press must result in a UI change in under .2 seconds to not feel broken.  Scalable web applications have to be blindingly fast -- the difference between 25 ms and 50 ms is 20 pages a second.  But what's the user's bandwidth?  Getting the 10kb page generated 25 ms faster will make little difference to the guy with the 14000 bps modem.

 

 

Once you know what your goals are, measure where you're at.  You'd be amazed at the number of people who come to me asking for help in making there things faster who **cannot tell me how they'll know when they're done**.  If you don't know what's fast enough, you could work it forever.

 

 

And if it does turn out that you need to stick with a scripting solution, and the script is the right thing to make faster, look for the big stuff.  Remember, script is glue.  The vast majority of the time spent in a typical page is in either the objects called by the script, or in the Invoke code setting up the call to the objects.  If you had to have one rule of scripting performance, it's this:  **manipulating data is really bad, and code is even worse. Don't worry about the Dims, worry about the calls.  Every call to a COM object that you eliminate is worth tens of thousands of micro-optimizations.**

 

And don't forget also that RIGHT is better than FAST.  Write the code to be extremely straightforward. Code that makes sense is code which can be analyzed and maintained, and that makes it performant.  Consider our "unused Dim" example -- the fact that an unused Dim has a 50 ns cost is irrelevant.  It's an unused variable.  It's worthless code.  It's a distraction to maintenance programmers.  That's the **real** performance cost -- it makes it harder for the devs doing the perf analysis to do their jobs well\!

