<div id="page">

# The Tragedy of Thread Happiness Disease

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/15/2004 11:46:00 AM

-----

<div id="content">

<span>A [JOS](http://www.joelonsoftware.com/ "http://www.joelonsoftware.com/") reader interested in developing server software asked recently </span>

<span></span>

**<span>Is it possible to determine the number of concurrent threads a server can support from the server's specification? </span>**

<span></span>

<span>Now, as I've said [before](http://blogs.msdn.com/ericlippert/archive/2003/12/01/53411.aspx "http://blogs.msdn.com/ericlippert/archive/2003/12/01/53411.aspx"), I'm no expert on performance tuning multi-threaded applications, but I have picked up a thing or two hanging around the real experts on the IIS team.  In fact, the perf teams get questions in this vein all the time from customers, both internal and external. </span>

<span></span>

<span>The interesting thing about the question is that it is symptomatic of **<span>Thread Happiness</span>**, a peculiar disease which usually strikes programmers who are relatively new to large-scale multi-threaded software development.   
  
How do I know that the questioner is getting Thread Happy?  Because if they were writing an application where there were two threads or five threads or ten threads then they wouldn't be asking *<span>that</span>* question.  They'd be asking "**<span>I want to write a server app with two/five/ten/whatever threads -- how buff does the server have to be?" </span>**</span>

<span> </span>

But they're asking the question **<span>"*<span>how many</span>* threads can I create on a given server?"</span>** so I can only assume that they are thinking of writing some application that is going to create a **<span>large</span>** and **<span>variable</span>** number of threads, **as many as possible**.

That's Thread Happiness right there.  Don't do that\!  It's almost certainly a bad design.  Massively multi-threaded applications cause more problems than they solve; they are very difficult to get correct, and even harder to get performant.

<span></span>

<span>To actually answer the question, no, there's no way of telling that from the server spec. Why?  Because there is nowhere near enough information in the "static" facts about the hardware.  Server application threading performance depends on "dynamic" facts like:  
  
</span>

  - <span>What other processes are going to be running?  </span>
  - <span></span><span>What are those threads going to be doing?  </span>
  - <span></span><span>How much memory do you have free?  </span>
  - <span></span><span>How much contention, locking, busy waiting, context switching, blah blah blah, is there going to be?</span>
  - <span></span><span>What are the performance metrics that will determine when things have gotten too bad.</span>

<span>So, in short, don't even go there.  Come up with a design that uses a small number of threads and tune that.</span>

<span></span>

<span>Now, you might be wondering "What about real-world server applications which do spin up a variable number of threads then?  How do they decide how many is too many?"  
  
It's quite hard.  These things used to be configurable in IIS, but a few years back the IIS team had the realization that (a) you can't expect a human being to figure out what the ideal thread limit is, and (b) the ideal thread limit changes dynamically because of all the factors I mentioned above.  
  
IIS therefore keeps a relatively small thread pool, and continually monitors its own performance, tweaking the thread pool count, thread priorities, etc, as conditions change.  It tries to keep the server well-tuned dynamically.   
  
This was non-trivial code to write.  Though I've done quite a bit of optimization of software that runs in-process with IIS, the details of the IIS thread timing algorithms are way, way beyond my skills.  I wouldn't recommend such an approach unless you've got a lot of experience in the field.  A huge performance lab with a wide range of server hardware and a dedicated staff of experienced performance testers doesn't hurt either\!</span>

</div>

</div>

