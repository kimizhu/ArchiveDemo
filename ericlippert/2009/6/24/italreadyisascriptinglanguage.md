# It Already Is A Scripting Language

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/24/2009 11:07:00 AM

-----

My recent post about the *possibility* of *considering maybe someday perhaps* adding "top level" methods to C\# in order to better enable "scripty" scenarios generated a surprising amount of immediate emphatic pushback. Check out the comments to see what I mean.

Two things immediately come to mind.

First off, the suggestion made by a significant number of the commenters is "instead of allowing top-level methods, strengthen the using directive." That is, if you said "using System.Math;" then all the static members of that class could be used without qualification.

Though that is a perfectly reasonable idea that is requested frequently, it does not actually address the problem that top-level methods solve. A better "using" makes things easier for the developer writing the *call*. The point of top-level methods for scripty scenarios is to make it easier on the developer writing the *declaration*. The point is to eliminate the "ritual" of declaring an unnecessary class solely to act as a container for code.

Second, and more generally, I am surprised by this pushback because of course **C\# already is a scripting language, and has had this feature for almost a decade.** Does this code fragment look familiar?

 

\<%@ Page Language="C\#" %\>     
\<script runat="server"\>     
  void Page\_Load(object sender, EventArgs e)   
  {  
      // whatever  
  }  
\</script\>  
...  

Where's the class? Where are the using directives that allow "EventArgs" to be used without qualification? Where's the code that adds this method group to the event's delegate? All the ritual seems to have been eliminated somehow. That sure looks like a "top-level method" to me.

Of course we know that behind the scenes it isn't any such thing. ASP.NET does textual transformations on the page to generate a class file. And of course, we recommend that you use the "code behind" technique to make a file that actually contains the methods in the context of an explicit (partial) page class, to emphasize that yes, this is a class-based approach to server-side processing.

But you certainly do not have to use "code behind". If you're an ASP traditionalist (like me\!) and would rather see C\# as a "scripting language" that "scripts" the creation of content on a web server, you go right ahead. You can put your "top level" methods in "script" blocks and your "top level" statements in "\<% %\>" blocks and you'll thereby avoid the ritual of having to be explicit about the containers for those code elements. The ASP.NET code automatically reorganizes your "script" code into something that the C\# compiler can deal with, adding all the boring boilerplate code that has to be there to keep the compiler happy.

But consider now the burden placed upon the developers of ASP.NET by the language design. Those guys cannot simply parse out the C\# text and hand it to the C\# compiler. They've got to have a solid understanding of what the legal code container topologies are, and jump through hoops -- not particularly difficult hoops, but hoops nevertheless -- to generate a class that can actually be compiled and executed.

This same burden is placed upon *every* developer who would like to expose the ability to add execution of end-user-supplied code to their application, whether that's in the form of adding extensibility via scripting, or by enabling evaluation of user-supplied expressions (such as queries). It's a burden placed on developers of productivity tools, like Jon Skeet's "snippet compiler", or LINQPad. It's a burden on developers who wish to experiment with REPL-like approaches to rapid development of prototypes, test cases, and so on.

I am not particularly excited about the convenience of the ability to save five characters by eliding the "Math." when trying to calculate a cosine. The exciting value props are that we might be able to **lower the cost of building tools that extend the power of the C\# language into third-party applications,** and at the same time enable **new ways to rapidly experiment and develop high-quality code.**

Of course we do not want to wreck the existing value props of the language in doing so; as I said last time, we also believe that there is huge value in the language design naturally leading professional, large-scale application developers towards building well-factored component-based programs.

Like all design decisions, when we're faced with a number of competing, compelling, valuable and noncompossible ideas, we've got to find a workable compromise. We don't do that except by **considering all the possibilites**, which is what we're doing in this case.

