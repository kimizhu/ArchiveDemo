# How Do The Script Garbage Collectors Work?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/17/2003 8:23:00 PM

-----

UPDATE: **This article was written in 2003**. Since that time the JScript garbage collector has been completely rewritten so as to be more performant in general, to handle the larger working sets entailed by modern web applications that we had absolutely no idea were coming when we designed the JScript GC back in 1995, to be better at predicting when there is garbage that needs collecting, and to be better at handling circular references involving browser objects. I did not do any of that work; I haven't worked on the script team for almost a decade now. I do not know how the modern JScript GC works; I've had the architect describe the basics to me but I am not an expert on it. **This article should be considered "for historical purposes only"; it does not reflect how JScript works today.**

-----

JScript and VBScript both are automatic storage languages. Unlike, say, C++, the script developer does not have to worry about explicitly allocating and freeing each chunk of memory used by the program. The internal device in the engine which takes care of this task for the developer is called the **garbage collector**.

Interestingly enough though, JScript and VBScript have completely different garbage collectors. Occasionally people ask me how the garbage collectors work and what the differences are.

JScript uses a **nongenerational mark-and-sweep garbage collector**. It works like this:

  - Every variable which is "in scope" is called a "scavenger". A scavenger may refer to a number, an object, a string, whatever. We maintain a list of scavengers -- variables are moved on to the scav list when they come into scope and off the scav list when they go out of scope.
  - Every now and then the garbage collector runs. First it puts a "mark" on every object, variable, string, etc – all the memory tracked by the GC. (JScript uses the VARIANT data structure internally and there are plenty of extra unused bits in that structure, so we just set one of them.)
  - Second, it clears the mark on the scavengers and the transitive closure of scavenger references. So if a scavenger object references a nonscavenger object then we clear the bits on the nonscavenger, and on everything that it refers to. (I am using the word "closure" in a different sense than in my earlier post.)
  - At this point we know that all the memory still marked is allocated memory which cannot be reached by any path from any in-scope variable. All of those objects are instructed to tear themselves down, which destroys any circular references.

Actually it is a little more complex than that, as we must worry about details like "what if freeing an item causes a message loop to run, which handles an event, which calls back into the script, which runs code, which triggers another garbage collection?" But those are just implementation details. (Incidentally, every JScript engine running on the same thread shares a GC, which complicates the story even further.)

You'll note that I hand-waved a bit there when I said "every now and then..." Actually what we do is keep track of the number of strings, objects and array slots allocated. We check the current tallies at the beginning of each statement, and when the numbers exceed certain thresholds we trigger a collection.

The benefits of this approach are numerous, but the principle benefit is that circular references are not leaked unless the circular reference involves an object not owned by JScript.

However, there are some down sides as well. Performance is potentially not good on large-working-set applications -- if you have an app where there are lots of long-term things in memory and lots of short-term objects being created and destroyed then the GC will run often and will have to walk the same network of long-term objects over and over again. That's not fast.

The opposite problem is that perhaps a GC will **not** run when you want one to. If you say "blah = null" then the memory owned by blah will not be released until the GC releases it. If blah is the sole remaining reference to a huge array or network of objects, you might want it to go away as soon as possible. Now, you can force the JScript garbage collector to run with the *CollectGarbage()* method, but I don't recommend it. The whole point of JScript having a GC is that you don't need to worry about object lifetime. If you do worry about it then you're probably using the wrong tool for the job.

VBScript, on the other hand, has a much simpler stack-based garbage collector. Scavengers are added to a stack when they come into scope, removed when they go out of scope, and any time an object is discarded it is *immediately* freed.

You might wonder why we didn't put a mark-and-sweep GC into VBScript. There are two reasons. First, VBScript did not have classes until version 5, but JScript had objects from day one; VBScript did not need a complex GC because there was no way to get circular references in the first place\! Second, VBScript is supposed to be like VB6 where possible, and VB6 does not have a mark-n-sweep collector either.

The VBScript approach pretty much has the opposite pros and cons. It is fast, simple and predictable, but circular references of VBScript objects are not broken until the engine itself is shut down.

The CLR GC is also mark-n-sweep but it is **generational** – the more collections an object survives, the less often it is checked for life.  This dramatically improves performance for large-working-set applications. Of course, the CLR GC was designed for industrial-grade applications, the JScript GC was designed for simple little web pages.

What happens when you have a web page, ASP page or WSH script with both VBScript and JScript? **JScript and VBScript know nothing about each others garbage collection semantics.** A VBScript program which gets a reference to a JScript object just sees another COM object. The same for a VBScript object passed to JScript. A circular reference between VBScript and JScript objects would not be broken and the memory would leak (until the engines were shut down). A noncircular reference will be freed when the object in question goes out of scope in both language (and the JS GC runs.)

