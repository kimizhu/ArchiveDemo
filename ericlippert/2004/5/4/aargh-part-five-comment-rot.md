<div id="page">

# Aargh, Part Five: Comment Rot

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/4/2004 12:04:00 PM

-----

<div id="content">

<div>

<span>Gripe \#6: Comment Rot </span>

<span></span>

<span>If you've been reading my [SimpleScript](http://blogs.msdn.com/ericlippert/category/4518.aspx "http://blogs.msdn.com/ericlippert/category/4518.aspx") code you might have noticed that there are very few comments in my code.  That's deliberate.  </span> <span>Why do we have comments in the first place?  We have comments because sometimes the [semantics of the code are not clear from the syntax](http://blogs.msdn.com/ericlippert/archive/2004/03/01/82168.aspx "http://blogs.msdn.com/ericlippert/archive/2004/03/01/82168.aspx").  Remember, the syntax tells you what the code does, the semantics tell you why the developer wrote that code, what the *<span>purpose</span>* is.  </span>

<span></span>

<span>I try to write code so that the semantics are clear just by reading the code, no comments required.  Code with one comment for every line [drives me nuts](http://blogs.msdn.com/ericlippert/archive/2004/03/10/87384.aspx "http://blogs.msdn.com/ericlippert/archive/2004/03/10/87384.aspx")\!  Imagine if people wrote books that way -- that their English was so unclear, so hard to understand that they needed to have a footnote on every sentence and paragraph explaining *in some other language* what the intended meaning was\!  It would be awful. </span>

<span></span>

<span>Worse, excessive comments cause "comment rot".  I don't know how many times I've been reading a piece of code, read a comment that said "there is such and such a problem with this code, I'll come back and fix it later", and sure enough, there is no problem anymore.  Or there is a problem, but it’s a different problem.  People change the code, compile it, test it, and never fix the comments.  The more comments there are, the more likely it is that this will happen.  **Wrong comments are usually much worse than no comments at all.** </span>

<span></span>

<span>Slightly less horrid than wrong comments are useless comments -- comments that do not explain the semantics, but restate the syntax.  How many times have you seen this: </span>

<span></span>

<span>j = j + 1  </span><span>'  Increment j </span>

<span></span>

<span>Thank you so much, developer, for letting me know what </span><span>j = j + 1</span><span> does\!  In my advanced old age I was beginning to forget the first thing I ever learned about programming\!  </span><span>Why do people do this?  This would be better, as it explains the semantics:</span> <span></span><span>j = j + 1  </span><span>'  Go to the next line </span>

<span></span>

<span>but surely this is an opportunity to eliminate the comment entirely: </span>

<span></span>

<span>currentLine = currentLine + 1 </span><span> </span>

<span></span>

<span>When I write a piece of code with comments I want those comments to stand out, not get lost in a sea of triviality\!  If I write a comment, you'd better believe that it's usually because the implementation of the intentions simply cannot be clearly expressed by the code, or because something really bizarre and important is going on.  (Like the weird garbage collection protection I mentioned earlier today.)  Comments should be signposts, your eye should be drawn to them because they're important.  Commenting style should train people to *<span>read</span>* the comments as they come across them in the code.  Writing lots of irrelevant comments trains people to *<span>ignore</span>* the comments\! </span>

<span></span>

<span>[Earlier today](http://blogs.msdn.com/ericlippert/archive/2004/05/04/125837.aspx "http://blogs.msdn.com/ericlippert/archive/2004/05/04/125837.aspx") I was talking about the default property semantics of </span><span>IDispatch</span><span>, and [earlier I commented](http://blogs.msdn.com/ericlippert/archive/2004/04/28/122259.aspx#122390 "http://blogs.msdn.com/ericlippert/archive/2004/04/28/122259.aspx#122390") that I'd blog about a conversation Matt Curland and I once had about the differences between VBScript's and VB's default property resolution in their respective Intellisense engines.  Now's a good chance to kill a few birds with one stone; one of my favourite comments I've ever written, and certainly one of the longest, is found in the VBScript Intellisense code that I wrote for Visual Interdev: </span>

<span></span>

<span>        // \[Eric Lippert 1998-03-27\] -- see Scripting bug 847  
</span><span>        //  
</span><span>        // We now have the following problem.  We have "foo.bar(".  
</span><span>        // What if bar returns an object that has a default property?  
</span><span>        // Suppose bar is a collection, for instance, with a default  
</span><span>        // property "item".  Then this should show statement completion  
</span><span>        // information for the "item" method, not for "bar" -- provided  
</span><span>        // that bar takes no arguments.  If bar takes arguments, then  
</span><span>        // we should show statement completion for bar.  
</span><span>        //  
</span><span>        // This generalizes to chains of defaults -- if bar is the default  
</span><span>        // property of foo, and bar takes no arguments, and bar  
</span><span>        // returns an object that has default property baz that takes  
</span><span>        // an argument blah, then  
</span><span>        //  
</span><span>        // foo(  
</span><span>        // foo.bar(  
</span><span>        // foo.bar.baz(  
</span><span>        //  
</span><span>        // should all return statement completion information for baz(blah).  
</span><span>        //  
</span><span>        // So here's what we'll do:  
</span><span>        //  
</span><span>        // First we determine if "bar" returns an object.  If it does,  
</span><span>        // we check and see if it has a default property chain.  If so, we  
</span><span>        // get type info for the final available function on the default  
</span><span>        // property chain.  
</span><span>        //  
</span><span>        // Second, if bar is a function that takes arguments, we return  
</span><span>        // function information for bar.  
</span><span>        //  
</span><span>        // If bar does not take arguments and bar returns an object with  
</span><span>        // a default property, then we return info on the default property.  
</span><span>        //  
</span><span>        // Otherwise, we return information on bar.  
</span><span>        //  
</span><span>        // This is not actually what Visual Basic does.  VB does the  
</span><span>        // following: (from email by Matt Curland, 1998-03-27)  
</span><span>        //  
</span><span>        // "If no parameters are supplied never call the default unless  
</span><span>        // you're in an assignment statement without a Set. If you're  
</span><span>        // in a non-ending statement of a call chain then only do  
</span><span>        // the default resolution if the specified function doesn't have  
</span><span>        // its own parameters and a parameter is actually specified."  
</span><span>        //  
</span><span>        // That is not exactly what we're doing here, but it's close enough  
</span><span>        // as far as I'm concerned.  
</span><span> </span>

<span>Gross, gross, gross.  And actually I've cut it short -- the comment then goes on to describe even more weirdnesses with the case where </span><span>foo.bar(blah)(</span><span> gets Intellisense on a method that returns a collection.  This is very complicated code; I wanted anyone who was about to change it to understand fully what the meaning behind it was. </span>

<span></span>

<span>The other thing that amuses me greatly about this comment is that apparently there were only 847 bugs in the scripting database at the time\!</span>

</div>

</div>

</div>

