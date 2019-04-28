# It Never Leaks But It Pours

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/13/2003 2:16:00 PM

-----

 

One of the easiest bugs to write is the dreaded **memory leak**.  You allocate some chunk of memory and never release it.  Those of us who grew up writing application software might sometimes have a cavalier attitude towards memory leaks -- after all, the memory is going to be reclaimed when the process heap is destroyed, right?  But those days are long gone.  Applications now often run for days, weeks or months on end.  Any memory that is leaked will slowly consume all the memory in the system until it dies horribly.  Web servers in particular are highly susceptible to leaks, as they run forever and move a LOT of memory around with each page request.

 

 

The whole point of developing a garbage collected language is to decrease the burden on the developer.  Because the underlying infrastructure manages memory for you, you don't have to worry about introducing leaks.  Of course, that puts the burden squarely upon the developer of the underlying infrastructure, ie, me.  As you might imagine, I've been called in to debug a LOT of memory leaks over the years.  The majority turned out to be in poorly written third-party components, but a few turned out to be in the script engines.   

 

 

I mentioned a while back that ASP uses a technique called "thread pooling" to increase its performance.  The idea is that you maintain a pool of idle threads, and when you need work done, you grab one from the pool.  This saves on the expense of creating a new thread and destroying it when you're done with it.  On a web server where there may be millions of page requests, the expense of creating a few million threads is non-trivial.  (Also, this ensures that you can keep a lid on the number of requests handled by one server -- if the server starts getting overloaded, just stop handing out threads to service requests.)

 

 

I think I also mentioned a while back that JScript has a per-thread garbage collector.  That is, if you create two engines on the same thread, they actually share a garbage collector.  When one of those engines runs a GC, effectively they all get collected.

 

 

What do these things have to do with each other?  Well, as it turns out, there *is* a memory leak that we have just discovered in the JScript engine.  A small data structure associated **with the garbage collector** is never freed when the thread goes away.  What incredible irony\!  The very tool we designed to prevent your memory leaks is leaking memory.

 

 

It gets worse.  As it turns out, this leak has been in the product for *years*.  Why did we never notice it?  Because it is a **per-thread leak**, and ASP uses thread pooling\!  Sure, the memory leaks, but only once per thread, and ASP creates a small number of threads, so they never noticed the leak.   

 

 

So why am I telling you this?  Because for some reason, it never rains but it pours.  We are suddenly getting a considerable number of people reporting this leak to us.  Apparently, all of a sudden there are third parties developing massively multi-threaded applications that continually create and destroy threads with script engines. Bizarre, but true. They are all leaking a few dozen bytes per thread, and a few hundred thousand threads in, that adds up.   

 

 

I have no idea what has led to this sudden uptick in multithreaded script hosts.  But if you're writing one, let me tell you two things:

 

 

1\)      We're aware of the problem.  Top minds on the Sustaining Engineering Team are looking at it, and I hope that we can have a patched version for a future service release.

2\)      Use thread pooling\!  Not only will it effectively eliminate this leak, it will make your lives easier in the long run, believe me.

 

 

This whole thing reminds me that I want to spend some time discussing some of the pitfalls we've discovered in performance tuning multi-threaded applications.  But that will have to wait for another entry.

