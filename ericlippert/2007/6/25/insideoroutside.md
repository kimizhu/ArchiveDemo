# Inside or Outside?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/25/2007 10:31:00 AM

-----

Here's a question I was asked recently which I got wrong. This is why I always encourage Microsofties to ask the entire C\#/VB/whatever-interest-group their question, rather than directing it to me specifically.

The question was which is "better"? (The following code snippets are not nested inside any namespace.)

using Blah;  
namespace Foo {  
  // etc  
}

or

namespace Foo {  
  using Blah;  
  // etc  
}

I don't know how to answer the question "which is better?" because I do not know the intended purpose of the code. I therefore tried to think of ways that these could be *different*, and answer the question "how do these differ?"

My answer was that if there is only one namespace in the file, it hardly matters. If there are two or more namespaces in the file, then put the directives you want shared amongst them at the top of the file, and the ones you do not want shared inside each the namespace. I further noted that having multiple top-level namespaces in the same file is a little weird, so maybe don't even go there and make it a moot point.

My first statement was not right. There is a way that these could be different that I forgot about. The first version is equivalent to

using global::Blah;  
namespace Foo {  
  // etc  
}

The second version is equivalent to

namespace Foo {  
  using global::Blah;  
  // etc  
}

**unless Foo contains a namespace called Blah.** In that case, it is equivalent to

namespace Foo {  
  using global::Foo.Blah;  
  // etc  
}

So there are at least two ways that things can get broken here. First, moving a not-fully-qualified directive from outside to inside (or vice versa) may cause a semantic change in the program. And second, if there is a not-fully-qualified directive inside the namespace which refers to a top-level namespace then adding a child namespace that collides with that name can introduce a semantic change in the program.

Therefore my advice is now:

  - have only one top-level namespace per file
  - do not have multiple namespaces at different points in the hierarchy with the same name; that's just confusing
  - if you choose to put using directives inside your namespaces and you are worried that you might have multiple namespaces at different points in the hierarchy with the same name, then fully qualify them with the global alias qualifier

If you take this advice then it really does not matter whether the directives go inside the namespace or outside; either way you will not accidentally break yourself by introducing a name collision.

