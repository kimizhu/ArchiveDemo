<div id="page">

# JScript Goes All To Pieces

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/27/2003 5:17:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">My entry the other day about fast mode in JScript .NET sparked a number of questions which deserve fuller explanations.  I’ll try to get to them in my next couple of blog entries.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">For example, when I said that it was no longer legal to redefine a function, I wasn’t really clear on what I meant.  JScript .NET still has closures, anonymous functions, and prototype inheritance.  We didn’t remove any of those.  Furthermore, it is very important to emphasize that **we implemented compatibility mode so that anyone who does need these features in JScript .NET can still get them – they will pay a performance penalty, but that’s their choice to make.**</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">What I meant was simply that this is now illegal:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">function foo() { return 1; } </span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">function foo() { return 2; }</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">whereas that is perfectly legal in JScript Classic.  In JScript Classic this means “discard the first definition”.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Pop quiz: what does this print out?</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">function foo() { return 1; } </span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">print(foo());</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">function foo() { return 2; }</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">print(foo());</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Of course that prints out “2” twice, because in JScript Classic, **function and variable declarations are always treated as though they came at the top of the block of code, no matter where they are found lexically in the block.**</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Obviously this is bizarre, makes debugging tricky, and is totally bug-prone.  The earlier definition is completely ignored, and yet it sits there in the source code, confusing maintenance programmers who do not see the redefinition, which might be a thousand lines later.  Thus, it is illegal in JScript .NET.  </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">But we only made this kind of redefinition illegal.  Other kinds of redefinition, like</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">var foo = function() { return 1; } </span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">print(foo());</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">foo = function() { return 2; } </span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">print(foo());</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">continue to work as you’d expect.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">So why was this *ever* legal?  Do language designers get some kind of perverse kick out of larding languages with “gotcha” idioms?  No, actually there was a pretty good reason for these semantics.  Two reasons actually.  The first is our old friend “muddle on through when you get an error”.  However, since this error can be caught at compilation time, this is not a very convincing point.  The more important point is this one:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">\<script language="JScript"\></span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">function foo(){ alert(1); }</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">foo();</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">\</script\></span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">\<script language="JScript"\></span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">function foo(){ alert(2); }</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">foo();</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;">\</script\></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Aha\!  Now we see what’s going on here.  I said “function and variable declarations are always treated as though they came at the top of the block of code”, and here we have **two** blocks.  IE will compile and run the first block, and **then** compile and run the second block, so this really will display “1” and then “2”.  **The IE compilation model allows for piecewise execution of scripts**. This scenario requires the ability to redefine methods on the fly, so, there you go.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">However, ASP does not have a piecewise compilation model, and neither does ASP.NET.  When we designed JScript .NET we removed this feature from fast mode because we knew that most “normal” hosts have all the source code at once and do not ever need to dynamically pull down new chunks from the internet after old chunks have already run.  By disallowing piecewise execution, we can do a lot more optimizations because we know that once you have a function, you’ve got it and no one is going to redefine it later.  </span>

</div>

</div>

