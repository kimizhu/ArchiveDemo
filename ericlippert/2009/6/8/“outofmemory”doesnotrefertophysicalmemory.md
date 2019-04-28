# “Out Of Memory” Does Not Refer to Physical Memory

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/8/2009 10:04:00 AM

-----

[![Inside Eric's head](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/OutOfMemoryDoesNotRefertoPhysicalMemory_9674/ExtendedMemory_thumb.jpg "Inside Eric's head")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/OutOfMemoryDoesNotRefertoPhysicalMemory_9674/ExtendedMemory_2.jpg) I started programming on x86 machines during a period of large and rapid change in the memory management strategies enabled by the Intel processors. The pain of having to know the difference between “extended memory” and “expanded memory” has faded with time, fortunately, along with my memory of the exact difference.

As a result of that early experience, I am occasionally surprised by the fact that many professional programmers seem to have ideas about memory management that haven’t been true since before the “80286 protected mode” days.

For example, I occasionally get the question **“I got an ‘out of memory’ error but I checked and the machine has plenty of RAM, what’s up with that?”**

Imagine, thinking that the amount of memory you have in your machine is relevant when you run out of it\! How charming\! :-)

The problem, I think, with most approaches to describing modern virtual memory management is that they start with assuming the DOS world – that “memory” equals RAM, aka “physical memory”, and that “virtual memory” is just a clever trick to make the physical memory *seem* bigger. Though historically that is how virtual memory evolved on Windows, and is a reasonable approach, that’s not how I *personally* conceptualize virtual memory management.

So, a quick sketch of my somewhat backwards conceptualization of virtual memory. But first a caveat. The modern Windows memory management system is far more complex and interesting than this brief sketch, which is intended to give the flavour of virtual memory management systems in general and some mental tools for thinking clearly about what the relationship between storage and addressing is. **It is not by any means a tutorial on the real memory manager.** (For more details on how it actually works, [try this MSDN article](http://msdn.microsoft.com/en-us/library/ms810616.aspx).)

I’m going to start by assuming that you understand two concepts that need no additional explanation: the operating system manages **processes**, and the operating system manages **files on disk**.

Each process can have as much data storage as it wants. It asks the operating system to create for it a certain amount of data storage, and the operating system does so.

Now, already I am sure that myths and preconceptions are starting to crowd in. Surely the process cannot ask for “as much as it wants”. Surely the 32 bit process can only ask for 2 GB, tops. Or surely the 32 bit process can only ask for as much data storage as there is RAM. Neither of those assumptions are true. The **amount of data storage reserved for a process is only limited by the amount of space that the operating system can get on the disk**. (\*)

This is the key point: the data storage that we call “process memory” is in my opinion best visualized as a **massive file on disk**.

So, suppose the 32 bit process requires huge amounts of storage, and it asks for storage many times. Perhaps it requires a total of 5 GB of storage. The operating system finds enough disk space for 5GB in files and tells the process that sure, the storage is available. *How does the process then write to that storage?* The process only has 32 bit pointers, but uniquely identifying every byte in 5GB worth of storage would require at least 33 bits.

Solving that problem is where things start to get a bit tricky.

The 5GB of storage is split up into chunks, typically 4KB each, called “pages”. The operating system gives the process a 4GB “virtual address space” – over a million pages - which *can* be addressed by a 32 bit pointer. The process then tells the operating system which pages from the 5GB of on-disk storage should be “mapped” into the 32 bit address space. (How? Here’s a page where [Raymond Chen gives an example of how to allocate a 4GB chunk and map a portion of it.](http://blogs.msdn.com/oldnewthing/archive/2004/08/10/211890.aspx))

Once the mapping is done then the operating system knows that when the process \#98 attempts to use pointer 0x12340000 in its address space, that this corresponds to, say, the byte at the beginning of page \#2477, and the operating system knows where that page is stored on disk. When that pointer is read from or written to, the operating system can figure out what byte of the disk storage is referred to, and do the appropriate read or write operation.

An “out of memory” error almost never happens because there’s not enough *storage* available; as we’ve seen, storage is disk space, and disks are huge these days. Rather, an “out of memory” error happens because the process is unable to find a large enough section of *contiguous unused pages in its virtual address space to do the requested mapping.*

Half ([or, in some cases, a quarter](http://blogs.msdn.com/oldnewthing/archive/2004/08/22/218527.aspx)) of the 4GB address space is reserved for the operating system to store it’s process-specific data. Of the remaining “user” half of the address space, significant amounts of it are taken up by the EXE and DLL files that make up the application’s code. Even if there is enough space in total, there might not be an unmapped “hole” in the address space large enough to meet the process’s needs.

The process can deal with this situation by attempting to identify portions of the virtual address space that no longer need to be mapped, “unmap” them, and then map them to some other pages in the storage file. If the 32 bit process is *designed* to handle massive multi-GB data storages, obviously that’s what its *got* to do. Typically such programs are doing video processing or some such thing, and can safely and easily re-map big chunks of the address space to some other part of the “memory file”.

But what if it isn’t? What if the process is a much more normal, well-behaved process that just wants a few hundred million bytes of storage? If such a process is just ticking along normally, and it then tries to allocate some massive string, the operating system will almost certainly be able to provide the disk space. But how will the process map the massive string’s pages into address space?

If by chance there isn’t enough contiguous address space then the process will be unable to obtain a pointer to that data, and it is effectively useless. In that case the process issues an “out of memory” error. Which is a misnomer, these days. It really should be an “unable to find enough contiguous address space” error; there’s plenty of *memory* because memory equals disk space.

I haven’t yet mentioned RAM. **RAM can be seen as merely a performance optimization.** Accessing data in RAM, where the information is stored in electric fields that propagate **at close to the speed of light** is *much* faster than accessing data on disk, where information is stored in enormous, heavy ferrous metal molecules that move at **close to the speed of my Miata**. (\*\*)

The operating system keeps track of what pages of storage from which processes are being accessed most frequently, and makes a *copy* of them in RAM, to get the speed increase. When a process accesses a pointer corresponding to a page that is not currently cached in RAM, the operating system does a “page fault”, goes out to the disk, and makes a copy of the page from disk to RAM, making the reasonable assumption that it’s about to be accessed again some time soon.

The operating system is also very smart about sharing read-only resources. If two processes both load the same page of code from the same DLL, then the operating system can share the RAM cache between the two processes. Since the code is presumably not going to be changed by either process, it's perfectly sensible to save the duplicate page of RAM by sharing it.

But even with clever sharing, eventually this caching system is going to run out of RAM. When that happens, the operating system makes a guess about which pages are *least* likely to be accessed again soon, writes them out to disk if they’ve changed, and frees up that RAM to read in something that is more likely to be accessed again soon.

When the operating system guesses incorrectly, or, more likely, when there simply is not enough RAM to store all the frequently-accessed pages in all the running processes, then the machine starts “thrashing”. The operating system spends all of its time writing and reading the expensive disk storage, the disk runs constantly, and you don’t get any work done.

This also means that "running out of RAM" seldom(\*\*\*) results in an “out of memory” error. Instead of an error, it results in bad performance because the full cost of the fact that storage is actually on disk suddenly becomes relevant.

Another way of looking at this is that **the total amount of virtual memory your program consumes is really not hugely relevant to its performance**. What is relevant is not the total amount of virtual memory consumed, but rather, (1) how much of that memory is not shared with other processes, (2) how big the "working set" of commonly-used pages is, and (3) whether the working sets of all active processes are larger than available RAM.

By now it should be clear why “out of memory” errors usually have nothing to do with how much *physical* memory you have, or how even how much *storage* is available. It’s almost always about the *address space*, which on 32 bit Windows is relatively small and easily fragmented.

And of course, many of these problems effectively go away on 64 bit Windows, where the address space is billions of times larger and therefore much harder to fragment. (The problem of thrashing of course still occurs if physical memory is smaller than total working set, no matter how big the address space gets.)

This way of conceptualizing virtual memory is completely backwards from how it is usually conceived. Usually it’s conceived as storage being **a chunk of physical memory**, and that the **contents of physical memory are swapped out to disk when physical memory gets too full**. But I much prefer to think of storage as being **a chunk of disk storage**, and physical memory being **a smart caching mechanism that makes the disk look faster**. Maybe I’m crazy, but that helps me understand it better.

\*\*\*\*\*\*\*\*\*\*\*\*\*

(\*) OK, I lied. 32 bit Windows limits the total amount of process storage on disk to 16 TB, and 64 bit Windows limits it to 256 TB. But there is no reason why a single process could not allocate multiple GB of that if there’s enough disk space.

(\*\*) Numerous electrical engineers pointed out to me that of course the individual *electrons* do not move fast at all; it's the *field* that moves so fast. I've updated the text; I hope you're all happy with it now.

(\*\*\*) It is possible in some virtual memory systems to mark a page as “the performance of this page is so crucial that it must always remain in RAM”. If there are more such pages than there are pages of RAM available, then you could get an “out of memory” error from not having enough RAM. But this is a much more rare occurrence than running out of address space.

