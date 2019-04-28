# Why no var on fields?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/26/2009 12:59:00 PM

-----

In my recent request for things that make you go hmmm, a reader notes that you cannot use "var" on fields. Boy, would I ever like that. I write this code all the time:

 

private static readonly Dictionary\<TokenKind, string\> niceNames =  
  new Dictionary\<TokenKind, string\>()  
  {  
    {TokenKind.Integer, "int"}, ...

Yuck. It would be much nicer to be able to write

 

private static readonly var niceNames =  
  new Dictionary\<TokenKind, string\>()...

You'd think this would be straightforward; we could just take the code that we use to determine the type of a local variable declaration and use it on a field. Unfortunately, it is not nearly that easy. Doing so would actually require a deep re-architecture of the compiler.

Let me give you a quick oversimplification of how the C\# compiler works. First we run through every source file and do a "top level only" parse. That is, we identify every namespace, class, struct, enum, interface, and delegate type declaration at all levels of nesting. We parse all field declarations, method declarations, and so on. In fact, we parse everything except method bodies; those, we skip and come back to them later.

Once we've done that first pass we have enough information to do a full static analysis to determine the type of *everything* that is not in a method body. We make sure that inheritance hierarchies are acyclic and whatnot. Only once everything is known to be in a consistent, valid state do we then attempt to parse and analyze method bodies. We can then do so with confidence because we know that the type of everything the method might access is well known.

There's a subtlety there. The field declarations have two parts: the type declaration and the initializer. The type declaration that associates a type with the name of the field is analyzed during the initial top-level analysis so that we know the type of every field before method bodies are analyzed. But the initialization is actually treated as part of the constructor; we pretend that the initializations are lines that come before the first line of the appropriate constructor.

So immediately we have one problem; if we have "var" fields then the type of the field cannot be determined until the expression is analyzed, and that happens *after* we already need to know the type of the field.

But it gets worse. What if the field initializer in a "var" field refers to another (static) "var" field? **What if there are long chains, or even cycles in those references?** There can be arbitrary expressions in those initializers, expressions which contain lambdas which contain expressions which require method type inference or overload resolution. All of these algorithms that are in the compiler were written with the assumption that when they run, the types of every top-level program entity is already known. All of those algorithms would have to be rewritten and tested in a world where top-level type information is being determined *from* them rather than being *consumed* by them.

It gets worse still. If you have "var" fields then the initializer could be of anonymous type. Suppose the field is public. There is not yet any standard in the CLR or the CLS about what the right way to expose a field of anonymous type is. We don't have good policies for documenting them, versioning them, or interoperating with them across languages. Doing this feature would potentially cause huge costs across the division.

Inferred locals have none of these problems; inferred locals never have cycles or refer to things that haven't been analyzed yet. Inferred locals never escape into public visibility.

So apparently this simple-seeming feature has the potential to cause really, really bad implementation issues in multiple ways, and all in order to avoid a small redundancy. This seems like it is possibly not worth the cost. If our goal is to remove the redundancy, I would therefore prefer to remove it the other way. Make this legal:

private static readonly Dictionary\<TokenKind, string\> niceNames =  
  new()... 

That is, state the type unambiguously in the declaration and then have the "new" operator be smart about figuring out what type it is constructing based on what type it is being assigned to. This would be much the same as how the lambda operator is smart about figuring out what its body means based on what it is being assigned to.

Thoughts?

