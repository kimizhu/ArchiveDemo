# Iterator Blocks, Part Six: Why no unsafe code?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/27/2009 10:07:00 AM

-----

There are three good reasons to disallow unsafe blocks inside an iterator block.

First, it is an incredibly unlikely scenario. The purpose of iterator blocks is to make it easy to write an iterator that walks over some abstract data type. This is highly likely to be fully managed code; it's simply not a by-design scenario.

Second, the scenario is a violent mixing of "levels." You think of the level of abstraction of a programming language feature as how "far from the machine" the feature is. Unsafe pointer manipulation is as close to the machine as it gets in C\#. You are running without a safety net, poking and peeking at the raw bytes in the virtual memory space of the process. All the protections of verifiable type safety are completely gone and you have total responsibility over every bit in the user-mode address space.

Iterator blocks, by contrast, are the highest level of abstraction in C\# 2.0. There is no immediately obvious mapping from the source code to the generated code; it's a magic box that does the right thing because the compiler is doing some pretty massive transformations on the code, generating classes and implementing interfaces for you.

Mixing those two levels of abstraction in one method seems like a really bad idea.

Third, we would have to do some extra work to make "fixed" blocks work correctly, work that would be directly opposed to our guidelines for the correct and efficient use of the "fixed" block.

I suppose I should briefly explain what the "fixed" block does. When you call unmanaged code that needs to, say, write into the unmanaged address that is the beginning of an array of 16 bit chars, somehow you need to tell the garbage collector "hey, you, GC, running on that other thread over there -- if you get the urge to reorganize memory to be more efficient, I need you to please not move this array for a moment, because unmanaged code on this thread is going to be writing into that address". This operation is called "fixing" or "pinning" the object.

Clearly it is a bad idea to pin lots of stuff and leave it pinned for a long time. The GC works as well as it does because it is able to reorganize memory to be more efficient; pinning blocks of memory across collections impedes the GC's ability to get work done. We therefore recommend that you pin as little as possible for as little time as possible.

The CLR provides us two ways to pin something. The hard, flexible, expensive way is to explicitly get a GCHandle on the object, tell the garbage collector "this object is pinned", take the address of the object, do what you need to do, and then unpin it.

The easy way is to mark a local variable as a "pinned local" using the "fixed" statement. The C\# compiler will generate special code that marks the local as "pinned". When the garbage collector does a collection, clearly the GC needs to look at all the locals on the current stack frame, because those locals are all "alive". When the GC sees that a local is pinned, it not only notes that it is alive, it also makes a note to itself that the referent of the contents of this variable are not to be moved during the collection. This is a cheap and easy alternative to explicitly getting a GC handle, but it depends on the variable in question being a local.

In an iterator block, locals are all realized by hoisting them to fields. Because we cannot pin fields, we'd have to change the code generation for the fixed block so that it took out a handle to the object, pinned it, and ensured that the object was unpinned at some point.

That's bad enough; now consider what happens when you yield while an object is pinned. Arbitrarily much time could pass between the yield and the unpin\! This directly violates our guidance that things should be pinned for as little time as possible.

For all those reasons, no unsafe code is allowed in iterator blocks. In the unlikely event that you need to dereference pointers in an iterator block, you can always extract the unsafe code to its own helper method and call it from the iterator block.

\*\*\*\*\*\*\*\*\*\*\*

I hope you've enjoyed this little trip into the weird corner cases of iterator blocks. This is a complicated feature with lots of interesting design choice points.

I'm flying south to Canada to spend some time with my family on the beaver-shark infested shores of the great inland sea known as Lake Huron, so the next few fabulous adventures will be pre-recorded. See you when I'm back.

