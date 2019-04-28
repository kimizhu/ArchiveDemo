# Atomicity, volatility and immutability are different, part three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/16/2011 7:03:00 AM

-----

So what does "volatile" mean, anyway? Misinformation abounds on this subject.

First off, so as to not bury the lead: in C\# the rules have been carefully designed so that **every volatile field read and write is also atomic**. (Of course the converse does not follow; it is perfectly legal for an operation to be atomic without it being "volatile", whatever that means.)

The way this is achieved is simple; the rules of C\# only permit you to annotate fields with "volatile" if the field also has a type that is guaranteed to have atomic reads and writes.

There is no *logical* requirement for this property; logically "volatile" and "atomic" are [orthogonal](http://blogs.msdn.com/b/ericlippert/archive/2005/10/28/483905.aspx). One could have a volatile-but-not-atomic read, for example. The very idea of doing so gives me the heebie-jeebies\! Getting an up-to-date value that has been possibly splinched through the middle seems like a horrid prospect. I am very glad that C\# has ensured that any volatile read or write is also an atomic read or write.

"Volatile" and "immutable" are essentially *opposites*; as we'll see **the whole point of volatility is to impose some kind of safety upon certain dangerous kinds of mutability**.

But what does "volatile" mean anyway? To understand this we must first go back in time to the early days of the C language. Suppose you are writing a device driver in C for a temperature-recording device at a weather station:

 

int\* currentBuf = bufferStart;  
while(currentBuf \< bufferEnd)  
{  
    int temperature = deviceRegister\[0\];  
    \*currentBuf = temperature;     
    currentBuf++;  
}

It is entirely possible that the optimizer reasons as follows: we know that bufferStart, bufferEnd and deviceRegister are initialized at the beginning of the program and never changed afterwards; they can be treated as constants. We know that the memory addresses mapped to deviceRegister do not overlap the buffer; there is no aliasing going on here. We see that **there are never any writes whatsoever** in this program to deviceRegister\[0\]. Therefore the optimizer can pretend that you wrote:

 

int\* currentBuf = bufferStart;  
**int temperature = deviceRegister\[0\];**  
while(currentBuf \< bufferEnd)  
{  
    \*currentBuf = temperature;     
    currentBuf++;  
}

which obviously is completely different in our scenario. The optimizer makes the seemingly-reasonable assumption that **if it can prove that a variable is never written to again then it need only read from it once**. But if the variable in question is marked as "volatile" -- that is, i**t changes on its own, outside of the control of the program** -- then the compiler cannot safely make this optimization.

That's what "volatile" is for in C. (And there are a few other usages as well; it also prevents optimizations that would screw up non-local gotos and some other relatively obscure scenarios.)

Let's make an analogy.

Imagine that there is a three-ring binder with a thousand pages in it, called **The Big Book of Memory**.  Each page has a thousand numbers on it, written in pencil so that they can be changed. You also have a "register page" which only has a dozen numbers on it, each with special meaning. When you need to do some operation on a number, first you flip to the right page, then you look up the right number on that page, then you copy it into the register page. You do your calculations only on the register page. When you are done a calculation you might write a number back somewhere into the book, or you might do another read from the book to get another value to operate on.

Suppose you are doing some operation in a loop -- as with our temperature example above. You might decide as an optimization that you are pretty sure that a particular number location is never going to change. So instead of reading it out of the book every time you need it, you copy it onto your register page, and never read it again. That's the optimization we proposed above. If one of those numbers is changing constantly based on factors outside your control then making this optimization is not valid. You need to go back to the book every time.

You'll note that **nothing in our C-style volatile story so far said anything at all about multithreading**. C-style volatile is about **telling the compiler to turn off optimizations** because the compiler cannot make reasonable assumptions about whether a given variable is changing or not. **It is not about making things threadsafe. Let's see why\! (\*)**

Suppose you have one thread that is writing a variable and another thread that is reading the same variable. You might think that this is exactly the same as our "C-style volatile" scenario. Imagine, for example, that our "deviceRegister\[0\]" expression above was not reading from some hardware register changing based on outside factors, but rather was simply reading from a memory address that was potentially changing based on the operation of another thread that is feeding it temperature data from some other source. Does "C-style volatile" solve our threading problem?

Kinda, sorta... well, no. The assumption we've just made is that a memory address being changed **by the temperature outside** is logically the same as a memory address being changed **by another thread**. **That assumption is not warranted in general**. Let's continue with our analogy to see why that is by first considering a model in which it *is* warranted.

Suppose instead of a number being updated by magic because it is somehow reflecting the temperature, suppose instead we have **two people taking turns using the book**. One is the Reader and one is the Writer. They only have one book and one register page between them, so they have to cooperate.

The Reader again might be reading the same book number slot over and over again. The Reader might decide that they can read the value just once, copy it to the register page, and then keep on using the copy. The Reader does this for a while. When it is time to give the Writer a turn, the Reader writes all of the current register page values to a special page in the book that is reserved for the Reader's use. The Reader then hands the book and the register page to the Writer. 

The Writer fills in the register page from their personal special location in the book, and keeps on doing what they were doing, which is writing new values into the book. When the Writer wants to take a break, again, they write the contents of the register page into the book and hand the register page back to the Reader.

The problem should be obvious. **If the Writer is writing to the location that the Reader previously cached to their register page then the Reader is making decisions based on an out-of-date value.**

If this analogy is a good description of the memory model of the processor then marking the shared location as "C-style volatile" does the right thing. The Reader knows that they should not be caching the value; to get the up-to-date value they have to go back to the book every time because they don't know whether the Writer changed the value when the Writer last had control. (This is particularly true in non-cooperative multithreading; perhaps the Reader and the Writer do not choose for themselves the schedule for taking turns\!)

**Unfortunately, that is not the actual memory model of many modern multi-processor machines**. The actual memory model goes more like this:

Suppose there are two people sharing one book -- again, the Reader and the Writer. Each has **their own register page**, **plus a blank book page**. The Reader goes to read a value from the book. But the book isn't there\! Instead, there is the Librarian. The Reader asks the Librarian for the book and the Librarian says "*you can't have the book itself; it is far to valuable to let you use it. But give me your blank page, and I'll copy the stuff from the book onto it*". The Reader figures that is better than nothing. In fact, it's really good, now that we consider it\! The Librarian hands the Reader back a copy of **the entire page**, not just the single number the Reader wanted. Now the Reader can now go to town and really efficiently do calculations involving **any number in that page** without talking to the Librarian again. Only when the Reader wants something outside of the bounds of the copied page do they have to go back to the Librarian. The Reader's performance just went way up.

Similarly, the Writer goes to write a value in the book at the same time. (Remember, the Reader and Writer no longer have to take turns because they both have their own register page.) But the Librarian does not allow this. The Librarian says "here, let me make you a copy of the page you want to write to. You make your changes to the copy, and when you are done, let me know and I will update the entire page." The Writer thinks this is great\! The Writer can write all kinds of crazy things and never talk to the Librarian again until the Writer needs to write to (or read from) a different page. When that happens the Writer hands the modified copy page to the Librarian, the Librarian copies the Writer's page back into the book, and gives the Writer a copy of the new page that the Writer wants.

Clearly this is awesome if the Reader and the Writer are not both reading and writing the same page of the book. But what if they are? C-style volatile does not help at all in this situation\! Suppose the Reader decides, oh, this memory location is marked as volatile, so I will not cache the read of the value onto my register page. Does that help? Not a bit\! Even if the reader always goes back to the page, **they are going back to their copy of the page**, the copy made for them by the Librarian. Suppose the Reader then says, "OK, this thing is volatile, so when I read it, heck, I'll just go back to the Librarian again and have the Librarian make me a new copy of this page". Does that help? No, because **the Writer might not have submitted the changes to the Librarian yet\!** The Writer has been making changes to their local copy.

In order to solve this problem the Reader could have a way to tell the Librarian "Hey, Librarian\! I need to read the most up-to-date version of this location from the Book". The Librarian then has to go find the Writer and **ask the Writer to stop what they are doing and submit the changes right now**. Both the Reader and the Writer come to a screeching halt and the Librarian then does the laborious work of ensuring that the Book is consistent. (And of course we haven't even considered situations where there are multiple readers and multiple writers all partying on the same page.) Alternatively, the Writer could tell the Librarian "hey, I'm about to update this value; go find anyone who is about to read it and tell them that they need to fetch a new copy of this page when I'm done". Doesn't really matter; the point is that everyone has to somehow cooperate to make sure that a consistent view of all the edits is achieved.

This strategy gives a **massive performance increases** in the common scenario where multiple readers and multiple writers are each working on data that is highly contiguous -- that is, each reader and each writer does almost all of their work on the one page they have copied locally, so that they don't have to go back to the Librarian. It gives **massive performance penalties** in scenarios where readers and writers are working on the same page and cannot tolerate inconsistencies or out-of-date values; the readers and writers are constantly going back to the Librarian, stopping everybody from doing work, and spending all their time copying stuff back into and out of the Book of Memory to ensure that the local caches are consistent.

Clearly we have a problem here. If C-style volatile doesn't solve this problem, what does solve this problem? **C\#-style volatile, that's what.**

Sorta. Kinda. In a pretty bogus way, actually.

In C\#, "volatile" means not only "*make sure that the compiler and the jitter do not perform any code reordering or register caching optimizations on this variable*". It also means "*tell the processors to do whatever it is they need to do to ensure that I am reading the latest value, even if that means halting other processors and making them synchronize main memory with their caches*".

Actually, that last bit is a lie. The true semantics of volatile reads and writes are considerably more complex than I've outlined here; in fact **they do not actually guarantee that every processor stops what it is doing** and updates caches to/from main memory. Rather, **they provide weaker guarantees about how memory accesses before and after reads and writes may be observed to be ordered with respect to each other**. Certain operations such as creating a new thread, entering a lock, or using one of the Interlocked family of methods introduce stronger guarantees about observation of ordering. If you want more details, read sections 3.10 and 10.5.3 of the C\# 4.0 specification.

Frankly, **I discourage you from ever making a volatile field**. Volatile fields are a sign that you are doing something downright *crazy*: you're attempting to read and write the same value on two different threads without putting a lock in place. Locks guarantee that memory read or modified inside the lock is observed to be consistent, locks guarantee that only one thread accesses a given hunk of memory at a time, and so on. The number of situations in which a lock is too slow is very small, and the probability that you are going to get the code wrong because you don't understand the exact memory model is very large. I don't attempt to write any low-lock code except for the most trivial usages of Interlocked operations. I leave the usage of "volatile" to real experts.

For more information on this incredibly complex topic, see:

[Why C-style volatile is almost useless for multi-threaded programming](http://software.intel.com/en-us/blogs/2007/11/30/volatile-almost-useless-for-multi-threaded-programming/)

[Joe Duffy on why attempting to 'fix' volatile in C\# is a waste of time](http://www.bluebytesoftware.com/blog/2010/12/04/SayonaraVolatile.aspx)

[Vance Morrison on incoherent caches and other aspects of modern memory models](http://msdn.microsoft.com/en-us/magazine/cc163715.aspx)

\-----

(\*) Of course we already know one reason: volatile operations are not guaranteed to be atomic, and thread safety requires atomicity. But there is a deeper reason why C-style volatility does not create thread safety.

