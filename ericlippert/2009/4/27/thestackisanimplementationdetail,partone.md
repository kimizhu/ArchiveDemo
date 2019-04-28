# The Stack Is An Implementation Detail, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/27/2009 9:15:00 AM

-----

[![Stack\<Stone\>](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/TheStackIsAnImplementationDetail_C978/Stack_thumb_1.jpg "Stack\<Stone\>")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/TheStackIsAnImplementationDetail_C978/Stack_4.jpg)I blogged a while back about how [“references” are often described as “addresses”](http://blogs.msdn.com/ericlippert/archive/2009/02/17/references-are-not-addresses.aspx) when describing the semantics of the C\# memory model. Though that’s arguably *correct*, it’s also arguably an *implementation detail* rather than an important eternal truth. Another memory-model implementation detail I often see presented as a fact is “value types are allocated on the stack”. I often see it because of course, [that’s what our documentation says](http://msdn.microsoft.com/en-us/library/system.valuetype.aspx).

Almost every article I see that describes the difference between value types and reference types explains in (frequently incorrect) detail about what “the stack” is and how the major difference between value types and reference types is that value types go on the stack. I’m sure you can find dozens of examples by searching the web.

I find this characterization of a value type based on its *implementation details* rather than its *observable characteristics* to be both confusing and unfortunate. Surely the most relevant fact about value types is not the implementation detail of *how they are allocated*, but rather the *by-design semantic meaning* of “value type”, namely *that they are always copied “by value”*. If the relevant thing was their allocation details then we’d have called them “heap types” and “stack types”. But that’s not relevant most of the time. Most of the time the relevant thing is their copying and identity semantics.

I regret that the documentation does not focus on what is most relevant; by focusing on a largely irrelevant implementation detail, we enlarge the importance of that implementation detail and obscure the importance of what makes a value type semantically useful. I dearly wish that all those articles explaining what “the stack” is would instead spend time explaining what exactly “copied by value” means and how misunderstanding or misusing “copy by value” can cause bugs.

Of course, the simplistic statement I described is not even *true*. As the MSDN documentation correctly notes, value types are allocated on the stack *sometimes*. For example, the memory for an integer field in a class type is part of the class instance’s memory, which is allocated on the heap. A local variable is hoisted to be implemented as a field of a hidden class if the local is an outer variable used by an anonymous method(\*) so again, the storage associated with that local variable will be on the heap if it is of value type.

But more generally, again we have **an explanation that doesn’t actually explain anything**. Leaving performance considerations aside, what possible difference does it make *to the developer* whether the CLR’s jitter happens to allocate memory for a particular local variable by adding some integer to the pointer that we call “the stack pointer” or adding the same integer to the pointer that we call “the top of the GC heap”? As long as the implementation maintains the semantics guaranteed by the specification, it can choose any strategy it likes for generating efficient code.

Heck, there’s no requirement that the *operating system* that the CLI is implemented on top of provide a per-thread one-meg array called “the stack”. That Windows typically does so, and that this one-meg array is an efficient place to store small amounts of short-lived data is great, but it’s not a requirement that an operating system provide such a structure, or that the jitter use it. The jitter could choose to put every local “on the heap” and live with the performance cost of doing so, as long as the value type semantics were maintained.

Even worse though is the frequently-seen characterization that value types are “small and fast” and reference types are “big and slow”. Indeed, value types that can be jitted to code that allocates off the stack are extremely fast to both allocate and deallocate. Large structures heap-allocated structures like arrays of value type are also pretty fast, particularly if you need them initialized to the default state of the value type. And there is some memory overhead to ref types. And there are some high-profile cases where value types give a big perf win. **But in the vast majority of programs out there, local variable allocations and deallocations are not going to be the performance bottleneck.**

Making the nano-optimization of making a type that really should be a ref type into a value type for a few nanoseconds of perf gain is probably not worth it. I would only be making that choice if profiling data showed that there was a large, real-world-customer-impacting performance problem directly mitigated by using value types. Absent such data, I’d always make the choice of value type vs reference type based on whether the type is *semantically* representing a value or *semantically* a reference to something.

UPDATE: [Part two is here](http://blogs.msdn.com/ericlippert/archive/2009/05/04/the-stack-is-an-implementation-detail-part-two.aspx)

\*\*\*\*\*\*\*

(\*) Or in an iterator block.

