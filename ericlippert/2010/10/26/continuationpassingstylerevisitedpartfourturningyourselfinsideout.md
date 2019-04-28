# Continuation Passing Style Revisited Part Four: Turning yourself inside out

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/26/2010 6:15:00 AM

-----

The obvious question at this point is: *if CPS is so awesome then why don’t we use it all the time?* Why have most professional developers never heard of it, or, those who have, think of it as something only those crazy Scheme programmers do?

First of all, it is simply **hard** for most people who are used to thinking about subroutines, loops, try-catch-finally and so on to reason about delegates being used for control flow in this way. I am reviewing my notes on CPS from CS442 right now and I see that in 1995 I wrote down a [*prof*QUOTE](http://www.mathnews.uwaterloo.ca/Issues/mn11402/pQ.php): *“With continuations you have to sort of stand on your head and pull yourself inside out”*. Professor Duggan (\*) was absolutely correct in saying that. Recall from a couple days ago that our tiny little example of CPS transformation of the C\# expression M(B()?C():D()) involved *four* lambdas. Not everyone is good at reading code that uses higher-order functions. It imposes a large cognitive burden.

Moreover: one of the nice things about having specific control flow statements baked in to a language is that they let your code *clearly* express the meaning of the control flow while *hiding the mechanisms* – the call stacks and return addresses and exception handler lists and protected regions and so on. **Continuations make the mechanisms of control flow explicit in the structure of the code**. All that emphasis on *mechanism* can overwhelm the *meaning* of the code.

Also: generation of four lambdas in what was 14 characters of code means that the number of objects you end up creating for all those closures is potentially huge for non-trivial programs. Lots of the research on CPS has gone into determining how to make a CPS transformation without creating a bazillion closures.

So if ordinary humans have difficulty reading and writing CPS programs, why is it interesting? Is this just an intellectual exercise?

The reason it's interesting to *me* is because I'm on the *implementation* side of languages. If you can write a compiler that turns your favourite language into CPS, and you have an orchestrator that knows how to dispatch the next continuation without consuming stack, then as we've seen, you have the all the building blocks you need to add any control flow that you like to that language as library calls. Have the compiler do the heavy lifting of translating the program into CPS, and then let the runtime deal with it; the users can write programs using the normal imperative style. If they want to add a new kind of control flow, they can write library methods for exotic control flows. Of course we don't actually *do* that, but it is character-building to understand the technique.

But that's still pretty computer-sciency. On the *usage* side of languages, there are increasingly many non-academic programmers who use call-with-current-continuation facilities in various languages to build real stuff that solves business problems, (these days, particularly in JScript) but this is still a fairly exotic technique by the standards of most mainstream line-of-business programmers.

The reason that CPS is relevant to C\# programmers is because you probably already know how to write CPS programs, even if you didn't know that was what they were called. You’ve probably done it at some point. And you probably found it confusing, error-prone, and difficult. **You’ve probably had to write a program that used asynchrony.**

**Next time**: What do asynchrony and CPS have to do with each other? (Hint: pretty much everything.)

(\*) Professor Duggan, if you're reading this, hey, it's me, the guy with the hat from your CS 442 class in 1995. It only took me ten years after I graduated - ten years of *designing programming language features and implementing their compilers* - to understand your explanation of continuation passing style. This is not a reflection of any failure in your teaching ability, which was *great*. It's just hard to get your head around CPS, you know?

