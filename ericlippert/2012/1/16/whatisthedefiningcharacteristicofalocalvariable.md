# What is the defining characteristic of a local variable?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/16/2012 6:48:00 AM

-----

If you ask a dozen C\# developers what a "local variable" is, you might get a dozen different answers. A common answer is of course that a local is "a storage location on the stack". But that is describing a local in terms of its implementation details; there is nothing in the C\# language that requires that locals be stored on a data structure called "the stack", or that there be one stack per thread. (And of course, locals are often stored in registers, and registers are not the stack.)

A less implementation-detail-laden answer might be that a local variable is a variable whose storage location is "allocated from the temporary store". That is, a local variable is a variable whose lifetime is known to be short; the local's lifetime ends when control leaves the code associated with the local's declaration space.

That too, however, is a lie. The C\# specification is surprisingly vague about the lifetime of an "ordinary" local variable, noting that its lifetime is only kinda-sorta that length. The jitter's optimizer is permitted broad latitude in its determination of local lifetime; a local can be cleaned up early or late. The specification also notes that the lifetimes of some local variables are necessarily extended beyond the point where control leaves the method body containing the local declaration. Locals declared in an iterator block, for instance, live on even after control has left the iterator block; they might die only when the iterator is itself collected. Locals that are closed-over outer variables of a lambda are the same way; they live at least as long as the delegate that closes over them. And in the upcoming version of C\#, locals declared in async blocks will also have extended lifetimes; when the async method returns to its caller upon encountering an "await", the locals live on and are still there when the method resumes. (And since it might not resume on the same thread, in some bizarre situations, the locals had better not be stored on the stack\!)

So if locals are not "variables on the stack" and locals are not "short lifetime variables" then what are locals?

The answer is of course staring you in the face. The **defining** characteristic of a local is that **it can only be accessed by name in the block which declares it**; it is **local** to a block. What makes a local truly unique is that it can *only* be a private implementation detail of a method body. The name of that local is never of any use to code lexically outside of the method body.

