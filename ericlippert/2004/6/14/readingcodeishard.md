# Reading Code Is Hard

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/14/2004 11:52:00 AM

-----

Escalation Engineer JeremyK asks in his [blog this morning](http://blogs.msdn.com/jeremyk/archive/2004/06/14/155196.aspx):  

> how do you teach people this “art” of digging deep very quickly into unfamilar code that you had no hand in writing? I myself, I come from a very traditional process of learning how to code, by sitting down and writing it. I am struggling with how to tailor a delivery to focus on reading vs. writing source code. To me the only way you can be truly efficient in this process is by having written code yourself.

 No kidding Jeremy -- code is way easier to write than it is to read.  

 First off, I agree with you that there are very few people who can read code who cannot write code themselves.  It's not like written or spoken natural languages, where **understanding what someone else says does not require understanding** **why they said it that way**.  For example, if I were to say something like 

> "There are two recipes for producing code: a strict and detailed, and a vague and sloppy.  The first produces elegant, tiered wedding cakes, the second, spaghetti."

 you would understand what I meant to get across, without having to understand that I'm using the literary techniques of "zero anaphora" and "parallel clauses" to produce a balanced, harmonious effect in the listener/reader.  Heck, you don't even have to know what a "verb" is to understand a sentence\!  But with code, it is vitally important that the both *intention* of the code's author and *how* *the code produces the intended effect* be clear from the code itself. 

 Therefore, I would turn the question around -- how do we WRITE code that is more easily read by people who need to get up to speed very quickly on the code, but who didn't write any part of it? 

 Here are some of the things I try to do when writing code so that it can be more easily read: 

  - Make the code amenable to tools.  Object browsers and Intellisense are great, but I'll tell you, I'm old school.  If I can't find what I want via grep, I'm not happy.  What makes code greppable?
    
      - Variables with names like "i" are badness.  You can't easily search code without getting false positives.
    
      - Avoid making names that are prefixes of other names.  For example, we have a performance marker in our code called "perfExecuteManifest", and another called "perfExecuteManifestInitialize".  Drives me nuts every time I want to grep the source code for the former, I have to wade through all the instances of the latter.  
    
      - Use the same name for “tramp data” in both places.  By tramp data, I mean those variables that you pass to a method only because they need to be passed on to another method.  The two variables are basically the same thing, so it helps if they have the same name.
    
      - Don't use macros that rename stuff.  If the method is called get\_MousePosition then don't declare it with a macro like [GETTER(MousePosition)](http://blogs.msdn.com/ericlippert/archive/2004/03/23/94651.aspx)-- because then I can't grep for the actual function name.
    
      - Shadowing is evil.  Please don't do it. 

  - Pick a consistent naming scheme.  If you're going to use Hungarian, use it consistently and universally, otherwise it becomes an impediment rather than a benefit.  Use Hungarian to document [data semantics, not storage types](http://blogs.msdn.com/ericlippert/archive/2003/09/12/52989.aspx).  Use Hungarian to document [universal truths](http://blogs.msdn.com/ericlippert/archive/2003/09/16/53015.aspx), not temporary conditions.

  - Use [assertions](http://blogs.msdn.com/ericlippert/archive/2004/06/04/148620.aspx)to document preconditions and postconditions.

  - Don't abbreviate English words.  In particular, don't abbreviate them in really weird ways.  In the script engines, the structure that holds a variable name is called NME.  Drives me nuts\!  It should be called VariableName.  

  - The standard C runtime library is not a paragon of good library design.  Do not emulate it.

  - Don't write "clever" code; the maintenance programmers don't have time to figure out your cleverness when it turns out to be broken.  

  - Use the features of the language do to what they were designed to do, not what they can do.  Don't use exceptions as a general flow control mechanism even though you can; use them to report errors.  Don't cast interface pointers to class pointers, even if you know it will work.  Etc.

  - Structure the source code tree in functional units, not in political units.  For example, on my team now the root subdirectories are "Frameworks" and "Integration", which are team names.  Unfortunately, the Frameworks team now owns the Adaptor subdirectory of the Integration directory, which is confusing.  Similarly, the various sub-trees have some subdirectories which are for client side components, some for server side components.  Some for managed components, some for unmanaged components.  Some for in-process components, some for out-of-proc components.  Some for retail components, some for internal testing tools. It's kind of a mess. Of all the possible ways to organize a source tree, the political structure is the least important to the maintenance programmer\! 

 Of course, I haven't actually answered Jeremy's question at all -- how do I debug code that I didn't write? 

 It depends on what my aim is.  If I just want to dig into a very specific piece of code due to a bug, I'll concentrate on understanding data flow and control flow in the specific scenario I'm debugging.  I'll step through all the code in the debugger, writing down the tree of calls as I go, making notes on which methods are produces and which are consumers of particular data structures.  I'll also keep a watchful eye on the output window, looking for interesting messages going by.  I'll turn on exception trapping, because usually exceptions are where the interesting stuff is, and because they can screw up your stepping pretty fast.  I'll put breakpoints all the heck over the place.   I'll make notes of all the places where my suggestions above are violated, because those are the things that are likely to mislead me. 

 If I want to understand a piece of code enough to modify it, I'll usually start with the headers, or I'll search for the public methods.  I want to know what does this class implement, what does it extend, what is it for, how does it fit into the larger whole?   I'll try to understand that stuff before I understand how the specific parts are implemented. That takes a lot longer, but you've got to do that due diligence if you're going to be making changes to complex code.

