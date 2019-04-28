# The Roslyn Preview Is Now Available

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/19/2011 2:00:00 PM

-----

I am super excited to announce that the Roslyn project code is now sufficiently coherent that we can start showing it to customers\!

But I am getting ahead of myself somewhat. What is this "Roslyn" project?

Here's the deal. We've got these great premiere languages for .NET development, C\# and Visual Basic. Obviously the compilers need to do considerable lexical, syntactic and semantic analysis of the code in order to first off, produce IL out the back end of the compiler, and second, produce all of that on-the-fly analysis needed for syntax colourization, IntelliSense, automatic refactorings, and so on. We do all this work to produce an analysis of the code, but we do not let you, the customer, take advantage of that analysis engine directly. Rather, you just get to see how the compiler and IDE teams make use of that analysis engine.

This is unfortunate. When I look around just Microsoft I see a dozen little C\# and VB language lexers, parsers and semantic analyzers that different teams have written to meet their own needs. There have got to be many more out there in the wild. This is a problem we can solve once for you, and then let you make use of the analysis engine for your own purposes.

Furthermore, it became clear as we were designing the next versions of C\# and VB that the existing compiler infrastructure built in unmanaged C++ was not going to meet our needs without a major overhaul at some point. We want a compiler infrastructure that supports both new interesting language features, and more dynamic ways to interact with your code as you're developing it.

To achieve all those goals we decided to join the C\# and VB teams together, and then split the resulting team apart into two subteams: one concentrating on upcoming C\# and VB features like async/await, and one concentrating on the long-term future of the compiler and tools infrastructure. The latter team is codenamed "Roslyn", and that's the team I've been working on for some time now.

We are now at the point where the Roslyn codebase is sufficiently fully-featured and coherent that it makes sense to start getting feedback from customers. **You can download the Roslyn Community Technology Preview as of right now**; it installs as an extension to Visual Studio 2010 SP1. We would love to get as much constructive feedback from you as possible; rather than leaving comments here, **please leave comments on the Roslyn Forum**. That way we'll be sure that our crack team of program managers sees the feedback and can aggregate it all appropriately. We're making a huge investment here and want to know that we're heading in the right direction to make something insanely great for you guys.

What we're looking for feedback on primarily is the set of **code analysis APIs** we're exposing to you guys. The APIs *themselves* as a set of classes with methods and so on, are now reasonably complete; we do not anticipate making major changes to the infrastructure for **Symbols**, **SyntaxNodes** and so on unless we hear loud feedback that we have gone with the wrong model. **Does this  set of APIs meet your needs?** What parts do you like, and what would you like to be different?

I want to make sure that we're setting expectations appropriately here. **This is a very early pre-beta-quality release of some extremely complicated software**. Keep that in mind as you are using it. The lexical and syntactic analysis engines that underlie those APIs are pretty much complete. The semantic analysis engine that sits behind the semantic analysis APIs is right now **nowhere near done**; on the C\# side we are still missing semantic analysis of major feature areas like **query comprehensions**, **attributes**, **iterator blocks**, **optional arguments**, **dynamic**, **async/await** and **unsafe** as well as a bunch of control flows; VB is in a similar state. **Extension methods**, **method type inference,** **lambdas**, and **generics** are working, so LINQ queries in the "method call" syntactic form should work.

Also, the **performance** of the entire analysis engine is not as high as we would like; we have done some performance tuning but there is more to come.

In short, **you're getting the earliest possible build we could reasonably show to people to get good feedback**; we are still a long way from being done. We are absolutely not announcing any dates or "ship vehicles" for a final release; even if I knew - which I don't - I couldn't tell you, so don't ask.

We are also as a part of this release shipping a preview of a new "interactive scripting" environment for C\# that will allow you to play around with code in a more experimental and free-form way, like you can do in F\# and other "Read-Eval-Print Loop" languages. Give it a spin and let us know what you think\! (Sorry, no VB version of this feature is available with the CTP.)

Over the next few months I'll do some more posts describing the interesting technology we've built behind-the-scenes to make the C\# and VB Roslyn code analysis engines work. For now, you can get more information on Roslyn at the following places:

  - [The Roslyn Community Technology Preview download site](http://msdn.com/roslyn)
  - [The Roslyn Forum, for **feedback and questions**](http://social.msdn.microsoft.com/forums/en-us/roslyn)
  - [The Visual Studio Connect page, for **bug reports and suggestions**](https://connect.microsoft.com/visualstudio)
  - [The Roslyn CTP Twitter feed](https://twitter.com/#!/search?q=%23RoslynCTP)

Some helpful blog posts:

  - [Soma on Roslyn](http://blogs.msdn.com/b/somasegar/archive/2011/10/19/roslyn-ctp-available-now.aspx)
  - [The C\# Team Blog on Roslyn](http://blogs.msdn.com/b/csharpfaq/archive/2011/10/19/introducing-the-microsoft-roslyn-ctp.aspx)
  - [The VB Team Blog on Roslyn](http://blogs.msdn.com/b/vbteam/archive/2011/10/19/introducing-the-microsoft-roslyn-ctp.aspx)
  - [The Visual Studio Team Blog on Roslyn](http://blogs.msdn.com/b/visualstudio/archive/2011/10/19/introducing-the-microsoft-roslyn-ctp.aspx)
  - [Shyam on building syntax visualizers using Roslyn](http://blogs.msdn.com/b/visualstudio/archive/2011/10/19/roslyn-syntax-visualizers.aspx)

Fun stuff:

  - [Wikipedia on Roslyn](http://en.wikipedia.org/wiki/Roslyn,_Washington)
  - We called it "Roslyn" because my office has a [Northern Exposure](http://en.wikipedia.org/wiki/Northern_Exposure). Ha ha ha\!

