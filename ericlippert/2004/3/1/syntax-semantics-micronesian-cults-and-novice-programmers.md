<div id="page">

# Syntax, Semantics, Micronesian cults and Novice Programmers

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/1/2004 11:18:00 AM

-----

<div id="content">

<span>I've had this idea in me for a long time now that I've been struggling with getting out into the blog space.  It has to do with the future of programming, declarative languages, Microsoft's language and tools strategy, pedagogic factors for novice and experienced programmers, and a bunch of other stuff.  All these things are interrelated in some fairly complex ways.  I've come to the realization that I simply do not have time to organize these thoughts into one enormous essay that all hangs together and makes sense.  I'm going to do what blogs do best -- write a bunch of (comparatively\!) short articles each exploring one aspect of this idea.  If I'm redundant and prolix, so be it. </span>

<span></span>

<span>Today I want to blog a bit about novice programmers.  In future essays, I'll try to tie that into some ideas about the future of pedagogic languages and languages in general.  </span>

<span></span>

**<span>Novice programmers reading this</span>**<span>: I'd appreciate your feedback on whether this makes sense or it's a bunch of useless theoretical posturing. </span>

<span></span>

**<span>Experienced programmers reading this</span>**<span>:  I'd appreciate your feedback on what you think are the vital concepts that you had to grasp when you were learning to program, and what you stress when you mentor new programmers. </span>

<span></span>

<span>An intern at another company wrote me recently to say "*<span>I am working on a project for an internship that has lead me to some scripting in vbscript.  Basically I don't know what I am doing and I was hoping you could help.</span>*"  The writer then included a chunk of script and a feature request.  I've gotten requests like this many times over the years; there are a lot of novice programmers who use script, for the obvious reason that we designed it to be appealing to novices. </span>

<span></span>

<span>Well, as I wrote last Thursday, there are times when you want to teach an intern to fish, and times when you want to give them a fish.  I could give you the line of code that implements the feature you want.  And then I could become the feature request server for every intern who doesn't know what they're doing…  nope.  Not going to happen.  Sorry.  Down that road lies cargo cult programming, and believe me, you want to avoid that road. </span>

<span></span>

<span>What's cargo cult programming?  Let me digress for a moment.  The idea comes from a [true story,](http://en.wikipedia.org/wiki/Cargo_cult) which I will briefly summarize: </span>

<span></span>

<span>During the Second World War, the Americans set up airstrips on various tiny islands in the Pacific.  After the war was over and the Americans went home, the natives did a perfectly sensible thing -- they dressed themselves up as ground traffic controllers and waved those sticks around.  They mistook cause and effect -- they assumed that the guys waving the sticks were the ones making the planes full of supplies appear, and that if only they could get it right, they could pull the same trick.  From our perspective, we know that it's the other way around -- the guys with the sticks are there **because** the planes need them to land.  No planes, no guys.  </span>

<span></span>

<span>The cargo cultists had the unimportant surface elements right, but did not see enough of the whole picture to succeed. They understood the **<span>form</span>** but not the **<span>content</span>**.  There are lots of cargo cult programmers -- **<span>programmers who understand what the code does, but not how it does it</span>**.  Therefore, they cannot make meaningful changes to the program.  They tend to proceed by making random changes, testing, and changing again until they manage to come up with something that works.  </span>

<span></span>

<span>(Incidentally, Richard Feynman wrote a great essay on cargo cult science.  Do a web search, you'll find it.) </span>

<span></span>

<span>Beginner programmers: do not go there\! Programming courses for beginners often concentrate heavily on getting the syntax right.  By "syntax" I mean the actual letters and numbers that make up the program, as opposed to "semantics", which is the meaning of the program.  As an analogy, "syntax" is the set of grammar and spelling rules of English, "semantics" is what the sentences mean.  Now, obviously, you have to learn the syntax of the language -- unsyntactic programs simply do not run. But what they don't stress in these courses is that **<span>the syntax is the easy part.</span>**  The cargo cultists had the syntax -- the formal outward appearance -- of an airstrip down cold, but they sure got the semantics wrong. </span>

<span></span>

<span>To make some more analogies, it's like playing chess.  Anyone can learn **<span>how the pieces legally move</span>**.  Playing a game where the strategy makes sense is the hard (and interesting) part.  **<span>You need to have a very clear idea of the semantics of the problem you're trying to solve, then carefully implement those semantics.</span>** </span>

<span></span>

<span>Every VBScript statement has a meaning.  **<span>Understand what the meaning is.</span>**  Passing the right arguments in the right order will come with practice, but getting the meaning right requires thought.  You will eventually find that some programming languages have nice syntax and some have irritating syntax, but that it is largely irrelevant.  It doesn't matter whether I'm writing a program in VBScript, C, Modula3 or Algol68 -- all these languages have different syntaxes, but very similar semantics.  **<span>The semantics *<span>are</span>* the program.</span>** </span>

<span></span>

<span>You also need to understand and use **<span>abstraction</span>**.  High-level languages like VBScript already give you a huge amount of abstraction away from the underlying hardware and make it easy to do even more abstract things. </span>

<span></span>

<span>Beginner programmers often do not understand what abstraction is.  Here's a silly example.  Suppose you needed for some reason to compute 1 + 2 + 3 + .. + n for some integer n.  You could write a program like this: </span>

<span></span>

<span>n = InputBox("Enter an integer")  
</span><span>  
Sum = 0  
</span><span>For i = 1 To n  
</span><span>      Sum = Sum + i  
</span><span>Next  
  
</span><span>MsgBox Sum</span>

<span></span>

<span>Now suppose you wanted to do this calculation many times.  You could replicate the middle four lines over and over again in your program, or you could **<span>abstract the lines into a named routine</span>**: </span>

<span></span>

<span>Function Sum(n)  
</span><span>      Sum = 0  
</span><span>      For i = 1 To n  
</span><span>            Sum = Sum + i  
</span><span>      Next  
</span><span>End Function  
</span><span>  
</span><span>n = InputBox("Enter an integer")  
</span><span>MsgBox Sum(n)</span>

<span></span>

<span>That is **<span>convenient</span>** -- you can write up routines that make your code look cleaner because you have less duplication.  But **<span>convenience is not the real power of abstraction</span>**.  The power of abstraction is that **<span>the implementation is now irrelevant to the caller</span>**.  One day you realize that your sum function is inefficient, and you can use Gauss's formula instead.  You throw away your old implementation and replace it with the much faster: </span>

<span></span>

<span>Function Sum(n)  
</span><span>      Sum = n \* (n + 1) / 2  
</span><span>End Function </span>

<span></span>

<span>The code which calls the function doesn't need to be changed.  If you had not abstracted this operation away, you'd have to change all the places in your code that used the old algorithm. </span>

<span></span>

<span>A study of the history of programming languages reveals that we've been moving steadily towards languages which support more and more powerful abstractions.  Machine language abstracts the **<span>electrical signals</span>** in the machine, allowing you to program with **<span>numbers</span>**.  Assembly language abstracts the **<span>numbers</span>** into **<span>instructions</span>**.  C abstracts the **<span>instructions</span>** into higher concepts like **<span>variables, functions and loops</span>**.  C++ abstracts even farther by allowing variables to refer to **<span>classes</span>** which contain both **<span>data and functions that act on the data</span>**.  XAML abstracts away the notion of a class by providing a **<span>declarative syntax</span>** for object relationships. </span>

<span></span>

<span>To sum up, Eric's advice for novice programmers is: </span>

<span></span>

  - **<span>Don't be a cargo cultist -- understand the meaning and purpose of every line of code before you try to change it.</span>**
  - **<span></span><span>Understand abstraction, and use it appropriately.</span>**

<span></span>

<span>The rest is just practice.</span>

</div>

</div>

