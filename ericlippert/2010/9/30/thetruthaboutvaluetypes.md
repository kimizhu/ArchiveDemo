# The Truth About Value Types

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/30/2010 6:33:00 AM

-----

As you know if you've [read this blog for a while](http://blogs.msdn.com/b/ericlippert/archive/2009/04/27/the-stack-is-an-implementation-detail.aspx), I'm disturbed by the myth that "value types go on the stack". Unfortunately, there are plenty of examples in our own documentation and in many books that reinforce this myth, either subtly or overtly. I'm opposed to it because:

1.  It is usually stated *incorrectly:* the statement should be "value types *can* be stored on the stack", instead of the more common "value types are always stored on the stack".
2.  It is almost always *irrelevant*. We've worked hard to make a managed environment where the distinctions between different kinds of storage are hidden from the user. Unlike some languages, in which you must know whether a particular storage is on the stack or the heap for correctness reasons.
3.  It is incomplete. What about *references*? References are neither *value types* nor *instances of reference types*, but they are values. They've got to be stored somewhere. Do they go on the stack or the heap? Why does no one ever talk about them? Just because they don't have a type in the C\# type system is no reason to ignore them.

The way in the past I've usually pushed back on this myth is to say that the real statement should be **"in the Microsoft implementation of C\# on the desktop CLR, value types are stored on the stack when the value is a local variable or temporary that is not a closed-over local variable of a lambda or anonymous method, and the method body is not an iterator block, and the jitter chooses to not enregister the value."**

The sheer number of weasel words in there is astounding, but they're all necessary:

  - Versions of C\# provided by other vendors may choose other allocation strategies for their temporary variables; there is no language requirement that a data structure called "the stack" be used to store locals of value type.
  - We have many versions of the CLI that run on embedded systems, in web browsers, and so on. Some may run on exotic hardware. I have no idea what the memory allocation strategies of those versions of the CLI are. The hardware might not even have the concept of "the stack" for all I know. Or there could be multiple stacks per thread. Or everything could go on the heap.
  - Lambdas and anonymous methods hoist local variables to become heap-allocated fields; those are not on the stack anymore.
  - Iterator blocks in today's implementation of C\# on the desktop CLR also hoist locals to become heap-allocated fields. They do not have to\! We could have chosen to implement iterator blocks as coroutines running on a fiber with a dedicated stack. In that case, the locals of value type could go on the stack of the fiber.
  - People always seem to forget that there is more to memory management than "the stack" and "the heap". Registers are neither on the stack or the heap, and it is perfectly legal for a value type to go in a register if there is one of the right size. If if is important to know when something goes on the stack, then why isn't it important to know when it goes in a register? Conversely, if the register scheduling algorithm of the jit compiler is unimportant for most users to understand, then why isn't the stack allocation strategy also unimportant?

Having made these points many times in the last few years, I've realized that the fundamental problem is in *the mistaken belief that the type system has anything whatsoever to do with the storage allocation strategy.* It is simply false that the choice of whether to use the stack or the heap has anything fundamentally to do with the *type* of the thing being stored. The truth is: **the choice of allocation mechanism has to do *only* with the *known* *required* *lifetime* of the *storage*.**

Once you look at it that way then everything suddenly starts making much more sense. Let's break it down into some simple declarative sentences.

  - There are three kinds of values: (1) instances of value types, (2) instances of reference types, and (3) references. (Code in C\# cannot manipulate instances of reference types directly; it always does so via a *reference*. In unsafe code, pointer types are treated like value types for the purposes of determining the storage requirements of their values.)
  - There exist "storage locations" which can store values.
  - Every value manipulated by a program is stored in some storage location.
  - Every reference (except the null reference) refers to a storage location.
  - Every storage location has a "lifetime". That is, a period of time in which the storage location's contents are valid.
  - The time between a start of execution of a particular method and the method returning normally or throwing an exception is the "activation period" of that method execution.
  - Code in a method can require the use of a storage location. If the required lifetime of the storage location is longer than the activation period of the current method execution then the storage is said to be "long lived". Otherwise it is "short lived". (Note that when method M calls method N, the use of the storage locations for the parameters passed to N and the value returned by N is required by M.)

Now we come to implementation details. In the Microsoft implementation of C\# on the CLR:

  - There are three kinds of storage locations: stack locations, heap locations, and registers.
  - Long-lived storage locations are always heap locations.
  - Short-lived storage locations are always stack locations or registers.
  - There are some situations in which it is difficult for the compiler or runtime to determine whether a particular storage location is short-lived or long-lived. In those cases, the prudent decision is to treat them as long-lived. In particular, the storage locations of instances of reference types are always treated as though they are long-lived, even if they are provably short-lived. Therefore they always go on the heap.

And now things follow very naturally:

  - We see that references and instances of value types are essentially the same thing as far as their storage is concerned; they go on either the stack, in registers, or the heap depending on whether the storage of the value needs to be short-lived or long-lived.
  - It is frequently the case that array elements, fields of reference types, locals in an iterator block and closed-over locals of a lambda or anonymous method must live longer than the activation period of the method that first required the use of their storage. And even in the rare cases where their lifetimes are shorter than that of the activation of the method, it is difficult or impossible to write a compiler that knows that. Therefore we must be conservative: all of these storage locations go on the heap.
  - It is frequently the case that local variables and temporary values can be shown via compile-time analysis to be unused after the activation period ends, and therefore can be treated short-lived, and therefore can go onto the stack or put into registers.

Once you abandon entirely the crazy idea that the *type* of a value has *anything whatsoever* to do with the *storage*, it becomes much easier to reason about it. Of course, my point above stands: you don't *need* to reason about it unless you are writing unsafe code or doing some sort of heavy interoperating with unmanaged code. Let the compiler and the runtime manage the lifetime of your storage locations; that's what its good at.

