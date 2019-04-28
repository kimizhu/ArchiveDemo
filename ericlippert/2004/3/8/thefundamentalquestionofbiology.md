# The Fundamental Question of Biology

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/8/2004 2:04:00 PM

-----

 

I think it was Gary Larson who said that the fundamental question of biology is "*If I put these two life forms in a jar, which one wins?"*  The very nature of the question presupposes a bunch of stuff -- that there are winners and losers, that any two things are naturally opposed, and so on.  

I'm sure that you have seen this attitude plenty.  It comes up all the time in the "holy wars" of information technology -- vi versus emacs, functional versus object oriented, MacOS versus Windows, ad nauseum. Each side ends up with partisans who seem more interested in making their side win than solving an actual problem. 

One of the questions I've gotten many times over the years is "I have a certain problem to solve and I don't know whether to use a script language or C++ or C\# or what.  Can you help?"  Usually the questioner is in a tizzy because of myths and misinformation about the script languages.  Now, obviously I worked on the script engines and so I'm a candidate for script partisanhood.  But I worked on the script engines because I wanted to come up with tools that solve **some** problems well.  If script doesn't solve **your** problem well, DON'T USE IT -- but please make that decision based on a dispassionate evaluation of the pros and cons, not misinformation and myths. 

What are some of the axes on which pros and cons can be evaluated?  Here are just a few that come to mind: 

  - **Initial dev cost**.  Script developers come cheaper than C++ developers, and script was designed to facilitate rapid development of *simple* scripts.  The lack of a class system, etc, can make development of *large* scripts more difficult and expensive. 

  - **Ongoing dev cost**.  Similarly, maintaining simple scripts can be much cheaper than maintaining C++ programs, but due to weak typing, etc, maintaining large scripts can be expensive. 

  - **Testing cost** -- scripts are inherently amenable to testing via scripting.  Testers like that. 

  - **Run time performance** -- the script engines are bytecode-interpreted languages, and there is a perf cost associated with that.  However, script is glue, and the compiler was built for speed.  This means that often the compilation cost is trivial compared to the cost of calling the object model, making scripts not appreciably slower than the equivalent C++ program.  More on this below. 

  - **Download performance** -- scripts are often used in mobile code solutions.  The download cost of a highly compressible short text file is often much cheaper than the equivalent executable.  

  - **Security** -- the script engines have a security model designed for safely running partially trusted code. 

  - **Object models**: Calling any IDispatch object is a model of clarity in script but a lot of code in C++.  On the other hand, in C++ you can call on the vtable interfaces, and also call Win32 APIs directly. 

  - **Modifiability and customizability**: All programs, independent of language, are dynamically updatable and end-user-modifiable, but script makes it easy and C++ makes it hard.  A best-of-both-worlds approach often works well - it is very easy to have a core program written in C++ that downloads customizations written in script -- IE is a good example of such a program\! 

Etc.  These are just a few of the factors that come to play when making this important decision.  We could take each of these points apart into many sub-points and discuss them in detail.  Heck, I could talk about this stuff all day, but I actually have some work to do. 

The cost-benefit discussion is a complicated discussion to have, but at least it's fundamentally about *facts*.  When I have the conversation with people about what languages to use, I hear a lot of *myths* about scripting.  Let me list a few of them: 

**Myth**: *Script programmers are less experienced than C++ programmers, and therefore will write buggier programs.  Some of those bugs might have security impacts. *

**Fact**: Languages with a built-in security model, automatic storage reclamation and no pointers afforded creation of **far fewer security bugs**.  When was the last time you saw someone write a script that was susceptible to a stack buffer overrun? 

Furthermore, if the problem is "*ensure that this program is bug free*" then the argument "*VBScript programmers are **on average** less experienced that C++ programmers*" is a specious argument.  The question is not **how experienced the *average* developers** are, the question is **are there developers experienced *enough* with their language of choice that they can write bug-free programs**? 

Writing bug-free programs in script is much, much easier than writing them in C++; there's a reason why C++ devs are more experienced, and that's because it's **harder** to do good work\! 

If you really want a solid program then one sensible thing to do would be to hire very experienced VBScript devs to write the code in really solid VBScript, rather than average C++ devs who write bug-prone C++. Does your average C++ programmer have more experience than an old hand at VBScript?  Maybe, but that's hardly relevant\!  I've met highly experienced C++ developers who couldn't identify a buffer overrun if their lives depended on it.  (None of them work here.) 

What's more, whole *taxonomies* of bugs have dropped out of my code since I started using C\# a whole lot more than C++. Memory-managed languages rock. 

**Myth**: *Writing solid error handling code in script is hard, and the error handling is inefficient*. 

**Fact**: First off, efficiency is irrelevant.  Exceptions are, by definition, exceptional.  Most people do not do optimization to ensure that their code that cleans up after errors puts up that “something is broken” dialog box blindingly fast\! 

I will grant that it is difficult to write really solid error handling in VBScript, due to the lack of structured exception handling or an On Error Goto mechanism like VB6.  That's one of my great regrets about VBScript, that we never got that in.  

JScript, however, has **a very robust and carefully implemented exception handling mechanism**, which personally I find slightly easier to use than C++'s exception handling mechanism. If implementing correct error handling was the **sole** criterion of what language to use, I'd use JScript myself.  But surely the choice of what language to use does not depend upon trivial semantic differences between the C++ catch() and the JScript catch()\!  Pick a more important criterion. 

**Myth**: *An attacker could replace the script host with a hostile script host. *

**Fact**:  As [Peter points out today](/ptorr/archive/2004/03/06/85266.aspx "http://weblogs.asp.net/ptorr/archive/2004/03/06/85266.aspx"), if a malicious attacker can run unmanaged code, install any executable software on the machine, change a registry key or rename a file then **the malicious attacker is already an administrator on your machine**.   An attacker who can do this *will not mess around with subverting your program, the script host, etc. * -- they already own the box.  If the attacker ownzors the box, [the box is ownzored, dudes](http://blogs.msdn.com/ericlippert/archive/2003/10/18/53241.aspx "http://blogs.msdn.com/ericlippert/archive/2003/10/18/53241.aspx").  What language your program uses is entirely irrelevant. 

**Myth**:  Script is slower, because it is bytecode interpreted, unoptimized, late bound… 

**Fact**:  This is correct, but probably specious. 

First, and most important, performance is always about **is it fast enough**? and seldom about **what is faster**?  If script is fast enough, who cares if something else is faster?\!  If you are arguing against using script then you need to demonstrate that **it is too slow**, not that **something else is faster**.  

Second, scripts are often *effectively* just as fast as executables because the performance is **gated on the object being scripted**, not the **script overhead**.  If you have a C++ program that spends 5000 ms reading a disk file and a VBScript program that spends 5000 ms reading a disk file and 50 ms compiling the script bytecode, I guarantee you that the user will neither notice nor care about the additional 0.05 seconds spent compiling the script.  In such a scenario, performance ceases to be relevant in making the choice you face. 

[As I've blogged before](http://blogs.msdn.com/ericlippert/archive/2003/10/17/53237.aspx "http://blogs.msdn.com/ericlippert/archive/2003/10/17/53237.aspx"), ***armchair performance analysis like this is worse than worthless, it is actively harmful*.**  There can be no performance analysis without **goals, scenarios and measurements**.  When you have goals, scenarios and measurements then you can talk about whether the implementation even needs to be optimized, and if so, how to do so.  Just saying that "script is slower" is pretty much content-free. 

While we're on the subject, a brief aside.  *Is JIT-compiled code necessarily slower than native code*?  Not necessarily -- it could be faster.  How is that possible?  Surely jitting takes extra time? 

IL is often smaller than native code, which means that the number of disk hits required to get the IL into memory is smaller than the number of hits required to load the equivalent native code.  **Processors** are dramatically faster than they were 20 years ago, but **disks** are nowhere near keeping pace with the speed improvement, and **program sizes are going up.**  

What can we conclude from this?  

**The startup cost of a program is gated by its size on disk -- doing the all-in-memory compilation by moving electrons is trivial compared to the cost of moving (comparatively) immense iron disks.  If jitting IL makes the program smaller on disk, it sometimes gets faster.  **Furthermore, **the performance penalty of .NET code, particularly for startup scenarios, is often overwhelmingly gated by the cost of starting up the runtime.**  For small programs, running the jitter is trivial compared to getting the jitter into memory in the first place.  Also, the jitter can consume at-runtime information about the state of the processor and tweak the compiled code accordingly.  This can in theory lead to perf improvements of jitted code over non-jitted code.  However, I do not know the details of any dynamic jitting algorithm that the .NET jitter may be using -- this may be a difference in principle only. 

However, note that this analysis is an off-the-cuff theoretical analysis, and therefore by my own statement, worthless.  If you're curious, **try it and see which is faster. **

****

**Myth: ***Script languages are less powerful than C++. *

One wonders why anyone uses any language other than C++ then, if it's the be-all and end-all of languages\!  Why did I spend five years of my life implementing less powerful languages, when we already had C++, the world's most powerful language? 

This is a linguistic problem.  We haven't defined what the word "powerful" means, so any comparison is meaningless.  **The "power" of a language cannot be discussed indepently of the scenario for which it is being used.  **Saying that one language is "more powerful" than another is a non-statement because it contains insufficient information to evaluate the claim.  ** **

Think back to high school physics; what is power?  *The ability to do some amount of work in some amount of time*.  The question "which is more powerful?" therefore depends on the answer to "*what work do you want to do*?"  The fact that C++ allows access to every byte in the process makes it way, way more powerful than JScript if you are writing a device driver.  The fact that JScript supports closures, anonymous function declarations and prototype inheritance makes it way more powerful than C++ if you are modeling a flexible hierarchy and like writing programs in functional style. 

The **question** you should be considering is *what features of the C++, VBScript and JScript languages work well to accurately capture the semantics you wish to model?  *Not making a blanket **assertion** as to the "power" of a particular grammar.  Once you know what you're modeling, you can see whether any of these languages work particularly well or particularly poorly, which will inform a cost analysis, which will inform a choice. 

Then again, plenty of people who are way smarter than me know what the best language is -- [Common Lisp](http://lambda.weblogs.com/discuss/msgReader%248778 "http://lambda.weblogs.com/discuss/msgReader$8778")\!  

**Myth**: *Script is the language of malicious hackers -- that's why they call them script kiddies -- and therefore script is bad.* 

**Fact**: "Script kiddies" are the legions of usually young, malicious wannabe hackers who download scripts written by experienced attackers and use them to launch scripted attacks against vulnerable sites. Saying that script is bad because spotty teenage vandals use it is obviously specious.  The last time my car was broken into, someone put a brick through the window.  Clearly bricks are bad.  Don't use bricks for anything\! 

In fact, inexperienced malicious hackers have flocked to script for the same reason that benign programmers have flocked to script: it is a powerful solution for writing administrative scripts.  

**Myth**: *Script can't call Win32 APIs directly, is single threaded, etc. *

**Fact**: Actually, that's factual, but it's not necessarily an argument against using script.   Does the problem you're trying to solve require low-level manipulation of the WIN32 API, threads, or processes?  If so then either **write an object model which exposes those things as IDispatch objects**, or use C++, whichever makes more sense based on other criteria.  The two things in the jar don't have to eat each other, they can work together. 

Note that it may make more sense to write the IDispatch object for the low-level stuff and do the high-level stuff in script.  That's a pretty common design pattern in modern applications, as you get the best of both worlds at some added developer cost. 

Because after all, you're going to have to write the low-level code in C++ anyway, right?  If you wrap an object around it, then you get to (a) reuse that code, and (b) develop the less low-level portions of your solution in what may be a more suitable high level language. 

**Myth**: *Script isn't object oriented. *

**Fact**: JScript *is* an object oriented language.  It supports prototype inheritance.  VBScript is not.  But more important is to realize that OOP is not an end in itself. 

OOP was invented to solve a particular set of problems associated with the development of large-scale software projects.  OOP emphasizes clean messaging interfaces, modularized, encapsulated design, and reuse through inheritance.  Why?  Because large groups of people working on large software projects must manage complexity while reducing costs, and those are good ways to do that. 

The script languages were specifically designed to enable rapid development of [short, simple programs](http://blogs.msdn.com/ericlippert/archive/2003/11/18/53388.aspx "http://blogs.msdn.com/ericlippert/archive/2003/11/18/53388.aspx").  C++ emphatically was not\!  If you have a **really big** project, I'd advise against using script languages, particularly VBScript, as the primary language.  Does your project involve multiple developers working on many different subsystems that need to seamlessly integrate?  If so, then I'd advise considering an OOP language. 

If you think that small, simple programs are best implemented in OOP style then you may be suffering from a disease I call Object Happiness -- the crazy belief that all problems are best solved by OOP, from Hello World to nuclear power plant operation software.  There's a lot of Object Happiness going around\! 

**Myth***: I know a guy who went from using his own custom-built script based install package to buying an install solution written in C++, and the number of bugs in his installer went way down.  Script is bad. *

**Fact**:   This is an argument for buying off-the-shelf solutions versus rolling your own, not an argument for using C++ versus VBScript.   If there an off-the-shelf solution that does what you want for a reasonable price, go buy it\! 

More generally, "I know a guy who wrote a buggy script" is not an argument against scripting.  I know plenty of people who write buggy C++ code… 

And finally, let me just sum up by saying: **vi roxors, emacs suxors**, and if you disagree with me, that's because you're WRONG WRONG WRONG.

