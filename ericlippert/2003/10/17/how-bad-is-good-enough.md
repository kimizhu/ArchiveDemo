<div id="page">

# How Bad Is Good Enough?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/17/2003 2:05:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">I keep talking about script performance without ever actually giving my rant about why most of the questions I get about performance are pointless at best, and usually downright harmful.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Let me give you an example of the kind of question I've gotten dozens of times over the last seven years.<span style="mso-spacerun: yes">  </span>Here's one from the late 1990s:</span>

<span style="FONT-SIZE: 10pt; FONT-FAMILY: Arial">We have some VBScript code that DIMs a number of variables in a well-used function.  The code never actually uses those variables and they go out of scope without ever being touched.  Are we paying a hidden price with each call?</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">What an interesting performance question\!<span style="mso-spacerun: yes">  </span>In a language like C, declaring n bytes total of local variables just results in the compiler generating an instruction that moves the stack pointer n bytes.<span style="mso-spacerun: yes">  </span>Making n a little larger or smaller doesn't change the cost of that instruction.<span style="mso-spacerun: yes">  </span>Is VBScript the same way?<span style="mso-spacerun: yes">  </span>Surprisingly, no\!<span style="mso-spacerun: yes">  </span>Here's my analysis:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="COLOR: #333399; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Bad Analysis \#1</span>

<span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">You dim it, you get it.  VBScript has no idea whether you’re going to do this or not:</span><span style="COLOR: #3366ff"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">function foo()</span><span style="COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">  
<span style="mso-spacerun: yes">  </span></span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Dim bar</span><span style="COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">  
  </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Execute(“bar = 123”)</span><span style="COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">In order to enable this scenario the script engine must</span>**<span style="COLOR: #3366ff"> </span><span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">at runtime</span>**<span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> bind all of the</span>**<span style="COLOR: #3366ff"> </span><span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">names</span>**<span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> of the local variables into a local binder. </span>**<span style="COLOR: #3366ff"> </span><span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">That causes an added per-variable-per-call expense.</span>**<span style="COLOR: #3366ff"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">(Note that JScript .NET **does** attempt to detect this scenario and optimize it, but that's another post.)</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Anyway, what is the added expense?<span style="mso-spacerun: yes">  </span>I happened to have my machine set up for perf measuring that day, so I measured it:</span>

<span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">On my machine,</span>**<span style="COLOR: #3366ff"> </span><span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">every</span>**<span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> additional variable which is dimensioned but not used adds a **50 nanosecond** **penalty** to</span>**<span style="COLOR: #3366ff"> </span><span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">every</span>**<span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> call of the function.  The effect appears to scale linearly with the number of unused dimensioned variables; I did not test scenarios with extremely large numbers of unused variables, as these are not realistic scenarios.  Note also that I did not test very *long* variable names; though VBScript limits variable names to 256 characters, there may well be an additional cost imposed by long variable names.</span><span style="COLOR: #3366ff"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">My machine is a 927 MHz Pentium III, so that’s somewhere around fifty processor cycles each.  I do not have VTUNE installed right now, so I can’t give you an exact processor cycle count.</span><span style="COLOR: #3366ff"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">That means that if your heavily used function has, say, five unused variables then</span>**<span style="COLOR: #3366ff"> </span><span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">every four million calls to your function will slow your program down by an entire second</span>**<span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, assuming of course that the target machine is my high-end dev machine.  Obviously a slower machine may exhibit considerably worse performance.</span>

<span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">However, you do not mention whether you are doing this on a server or a client.  That is extremely important when doing performance analysis\!</span><span style="COLOR: #3366ff"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #3366ff; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Since the penalty is imposed due to a heap allocation, the penalty on the server may scale differently based on the heap usage of other threads running in the server.  There may be contention issues - my measurements measured only “straight” processor cost; a full analysis of the cost for, say, an 8 proc heavily loaded server doing lots of small-string allocations may well give completely different results.  </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Now let me take this opportunity to tell you that **all the analysis I just described is almost completely worthless because it obscures a larger problem**.<span style="mso-spacerun: yes">  </span>There's an elephant in the room that we're ignoring.<span style="mso-spacerun: yes">  </span>The fact that a user is asking me about performance of VBScript tells me that either </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">(a) this user is a hard-core language wonk who wants to talk shop, or, more likely, </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">(b) **the user has a program written in script which he would like to be running faster.<span style="mso-spacerun: yes">  </span>The user cares deeply about the performance of his program.**</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Whoa\!<span style="mso-spacerun: yes">  </span>Now we see why this perf analysis is worthless.<span style="mso-spacerun: yes">  </span>**If the user cares so much about performance then why is he using a late-bound, unoptimized, bytecode-interpreted, weakly-typed, extremely dynamic language specifically designed for rapid development at the expense of runtime performance?** </span>

<span style="COLOR: #333399; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Bad Analysis \#2</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">If you want a script to be faster then there are way more important things to be optimizing away than the 50-nanosecond items.<span style="mso-spacerun: yes">  </span>**The key to effective performance tuning is finding the most expensive thing and starting with that.**  A **single** call that uses an undimensioned variable, for example, is hundreds of times more expensive than that dimensioned-but-unused variable.  A single call to a host object model method is thousands of times more expensive. Optimizing a script by trimming the 50 ns costs is like weeding your lawn by cutting the already-short grass with nail scissors and ignoring the weeds**.<span style="mso-spacerun: yes">  </span>It takes a long time, and it makes no noticeable impact on the appearance of your lawn.** <span style="mso-spacerun: yes"> </span>It epitomizes the difference between "active" and "productive". Don't do that\!<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">But even better advice that that would be to **throw away the entire script and start over in C if performance is so important.** </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Now, let me just take this opportunity to interrupt myself and say that yes, **script performance is important**.<span style="mso-spacerun: yes">  </span>We spent a lot of time optimizing the script engines to be pretty darn fast for late-bound, unoptimzed, bytecode-interpreted, weakly-typed dynamic language engines. Eventually you come up against the fact that you have to pick the right tool for the job -- VBScript is as fast as its going to get without turning it into a very different language or reimplementing it completely.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Unfortunately, this second analysis is hardly better than the first, because again, there is an elephant in the room.<span style="mso-spacerun: yes">  </span>There's a vital piece of data which has not been supplied, and that is the key to all perf analysis:</span>

<span style="COLOR: #333399; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">How Bad Is Good Enough?</span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">I was going easy on myself -- I actually consider this sort of "armchair" perf analysis to be not worthless, <span style="mso-spacerun: yes"> </span>I consider it to be actively harmful.  </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">I've read articles about the script engines that say things like "you should use </span><span class="NormalWebChar"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-size: 12.0pt">And 1</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> to determine whether a number is even rather than </span><span class="NormalWebChar"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-size: 12.0pt">Mod 2</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> because the chip executes the </span><span class="NormalWebChar"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-size: 12.0pt">And</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> instruction faster", as though VBScript compiled down to tightly optimized machine code.  People who base their choice of operator on utterly nonsensical rationales **are not going to write code that is** **maintainable or robust.<span style="mso-spacerun: yes">  </span>Those programs end up broken, and "broken" is the ultimate in bad performance, no matter how fast the incorrect program is.** </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">If you want to write fast code -- in script or not -- then ignore every article you ever see on "tips and tricks" that tell you which operators are faster and what the cost of dimensioning a variable is.<span style="mso-spacerun: yes">  </span>Writing fast code does not require a collection of cheap tricks, it requires analysis of user scenarios to set goals, followed by a rigorous program of careful measurements and small changes until the goals are reached.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">1) Have a user-focussed plan.  Know what your performance goals are.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">2) Know what to measure to test whether those goals are met.  Are you worried about throughput?  Time to first byte?  Time to last byte?  Scalability?</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">3) Measure the whole system, not just isolated parts.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">4) Measure carefully and measure often.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">That's what the MSN people do, and they know about scalable web sites.</span>

 <span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">I know that's not what people want to hear.  People have these ideas about performance analysis which as far as I can tell, last applied to PDP-11's.  Script running on web servers cannot be optimized through micro-optimization of individual lines of code -- it's not C, where you can know the exact cost of every statement.  With script you've got to look at the whole thing and attack the most expensive things.  Otherwise you end up doing a huge amount of work for zero noticable gain.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">But you've got to know what your goals are.<span style="mso-spacerun: yes">  </span>Figure out what is important to your users.<span style="mso-spacerun: yes">  </span>Applications with user interfaces have to be snappy -- the core processing can take five minutes or an hour, but a button press must result in a UI change in under .2 seconds to not feel broken.<span style="mso-spacerun: yes">  </span>Scalable web applications have to be blindingly fast -- the difference between 25 ms and 50 ms is 20 pages a second.<span style="mso-spacerun: yes">  </span>But what's the user's bandwidth?<span style="mso-spacerun: yes">  </span>Getting the 10kb page generated 25 ms faster will make little difference to the guy with the 14000 bps modem.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Once you know what your goals are, measure where you're at.<span style="mso-spacerun: yes">  </span>You'd be amazed at the number of people who come to me asking for help in making there things faster who **cannot tell me how they'll know when they're done**.<span style="mso-spacerun: yes">  </span>If you don't know what's fast enough, you could work it forever.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">And if it does turn out that you need to stick with a scripting solution, and the script is the right thing to make faster, look for the big stuff.<span style="mso-spacerun: yes">  </span>Remember, script is glue.  The vast majority of the time spent in a typical page is in either the objects called by the script, or in the Invoke code setting up the call to the objects.  If you had to have one rule of scripting performance, it's this:  **manipulating data is really bad, and code is even worse. Don't worry about the Dims, worry about the calls.  Every call to a COM object that you eliminate is worth tens of thousands of micro-optimizations.**</span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">And don't forget also that RIGHT is better than FAST.<span style="mso-spacerun: yes">  </span>Write the code to be extremely straightforward. Code that makes sense is code which can be analyzed and maintained, and that makes it performant.<span style="mso-spacerun: yes">  </span>Consider our "unused </span><span class="NormalWebChar"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-size: 12.0pt">Dim</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">" example -- the fact that an unused </span><span class="NormalWebChar"><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-size: 12.0pt">Dim</span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> has a 50 ns cost is irrelevant.<span style="mso-spacerun: yes">  </span>It's an unused variable.<span style="mso-spacerun: yes">  </span>It's worthless code.<span style="mso-spacerun: yes">  </span>It's a distraction to maintenance programmers.<span style="mso-spacerun: yes">  </span>That's the **real** performance cost -- it makes it harder for the devs doing the perf analysis to do their jobs well\!</span>

</div>

</div>

