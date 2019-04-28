# Cargo Cultists, Part Three: Is Mort A Cargo Cultist?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/2/2004 12:14:00 PM

-----

My colleague [Mike](http://www.mikepope.com/blog "http://www.mikepope.com/blog"), in a comment in [yesterday's entry](http://blogs.msdn.com/ericlippert/archive/2004/03/01/82168.aspx "http://blogs.msdn.com/ericlippert/archive/2004/03/01/82168.aspx"), mentions "Mort".  Who is this Mort guy? 

At Microsoft, we do a lot of market research, and frequently discover that large segments of our customers have characteristics in common.  It's a lot easier to talk about these customer segments if we take one prototypical example of that segment and give them a name.  It's a lot easier to say "*Mort likes Intellisense*" than "the *professional line-of-business programmer who lacks a degree in computer science but has a great deal of familiarity with Office and VBA, and who typically writes productivity applications shared amongst his coworkers, likes Intellisense*." 

There are many of these prototypical customers but the three we talk about most often on the Visual Studio team are **Mort** (the line-of-business developer), **Elvis** (the professional application developer) and **Einstein** (the expert on both low level bit-twiddling and high-level object oriented architectures.)  Elvis and Einstein got their jobs by studying computer science and going into development as a career.  Mort comes to a development position via his line of business -- he's an expert on frobnicating widgets, and one day realizes that his widget tracking spreadsheets could benefit from a little VBA magic, so he picks up enough VBA to get by. 

Clearly Elvis and Einstein are not Cargo Cult Programmers -- Elvis wouldn't last a month and Einstein's entire reason for continued breathing is directly opposed to Cargo Cult Programming.  Is Mort a Cargo Cult Programmer? 

Though Mort is more *likely* to be a Cargo Cult Programmer, I don't think that he necessarily *has* to be, and I certainly think that he does his job better if he is not\!  Yes, Mort doesn't understand OOP, but *for the kinds of problems Mort solves*, he doesn't need to know what inheritance is or how polymorphism works.  Mort's programs tend to be straightforward procedural manipulation of object models and strings, with a few loops and subroutines thrown in here and there.  Mort *does* understand that stuff, and often has a quite sophisticated understanding of the practical uses of the object model. 

What makes Mort cargo-cultish is that he tends to proceed from a not-quite-working solution (often obtained off the web, or by macro recording) to a working solution by piecemeal local changes.  Mort is a very *local* programmer -- he wants to make a few changes to one subroutine and be done.  Mort does not want to understand how an entire system works in order to tinker with it.  And my goodness, Mort *hates* reading documentation -- more on that tomorrow.  Intellisense is Mort's favourite feature in the whole world because it lets him look stuff up by hitting the period key.  Mort's primary job is to frobnicate widgets -- code is just a means to that end -- so every second spent making the code more elegant takes him away from his primary job. 

But what makes Mort not cargo-cultish is that Mort actually does enjoy writing code, Mort is interested in writing code, *Mort wants to do the right thing to write more effective code*.  But Mort, like all of us, sometimes fails to put in the necessary investment in time now that will pay off later. That's not because Mort is a lazy idiot; Mort's a very smart, dedicated guy, but you know that he's got this huge pile of widgets to frobnicate by Wednesday… 

In a sense, my advice for novice developers doesn't apply so much to Mort, because *that advice was directed towards people who want to be Elvis*.  Mort does his job *better* if he avoids cargo cultism.  Elvis *can't do his job at all* if he falls into cargo cultism.  In the future I'll try to be more clear about what developer segment I'm blogging about. 

Creating developer tools that work for Elvis and Einstein is relatively easy.  Creating dev tools that make Mort productive and effective is hard.  Creating tools that work effectively given the widely varying goals, learning styles and knowledge levels of those three is the hardest and most interesting thing about this job.  I'll certainly talk a lot more about Mort, Elvis and Einstein (as well as Matt, Thomas and Erin) as I start blogging more and more about VSTO.

