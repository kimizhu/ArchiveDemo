<div id="page">

# WSF Files, Pedagogic Code, and Lippert's Paradox

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/25/2004 5:32:00 PM

-----

<div id="content">

<div>

<span>The Scripting Guys had [a blog entry the other day about WSF files](/gstemp/archive/2004/02/24/79183.aspx "http://blogs.msdn.com/gstemp/archive/2004/02/24/79183.aspx").  I thought I'd do a quick follow-up on some of the points raised and questions asked in Greg's entry. </span>

<span></span>

<span>For those of you who don't know, the Windows Script Host can run .VBS files and .JS files containing VBScript and JScript.  But it can also run .WSF files, which are xml files that contain script blocks.  They're somewhat reminiscent of script-in-HTML that you see in the web browser.  We invented them because there was no good way to put meta-information about the script itself into the script languages.  </span>

<span></span>

<span>Anyway, let me dig into Greg's post a bit.  (This isn't going to make any sense unless you read Greg's post first.)  I'll paraphrase here and there. </span>

<span></span>

<span>Pedagogic Code </span>

<span></span>

<span>.wsf files are the latest twist on Windows Script Host scripts. Shouldn’t we be promoting the most recent innovations? </span>

<span>No\!  The purpose in life for User Education experts like the Scripting Guys is to make developers more productive, not to shill the latest technology.  That's Marketing's job\! </span>

<span></span>

<span>Now, we *<span>sincerely hope</span>* that there is a positive correlation between "new technology" and "solves all your problems better than the old stuff"\!  But it does not follow that "new" automatically means "better", either from a problem solving or pedagogic standpoint. </span>

<span></span>

<span>This is a good time for me to reiterate a theme that's come up [before](http://blogs.msdn.com/ericlippert/archive/2003/11/18/53388.aspx) in my blog: **<span>use the right tool for the job</span>**.  I've had people come up to me at industry conferences and say *<span>"Eric, a .BAT file that deletes a file is one line long, but a .WSF that does the same thing is ten lines long\!" </span>* Then they usually give me this smug look like they're thinking *<span>"Aha\!  Now I've got you\!"</span>*  Well, then my advice to those people would be to use the .BAT file if you have one file to delete\!  I don't recall ever making the statement that script was the best solution for everything. </span>

<span>I have been known to point out that .BAT files are lousy at solving problems which benefit from variables, subroutines and COM object calls -- if you've got one of those problems, maybe a Windows Script solution would work out better for you than a .BAT file.  Your mileage may vary. </span>

<span></span>

<span>I'll digress for a moment to talk about pedagogic code in general.  As far as I know no one else has claimed it, so I'm hereby declaring that this is **<span>Lippert's Paradox</span>**:  **<span>Pedagogic code must be short and easily understood, but code is designed to manage complexity.  </span>**</span>

<span></span>

<span>How do you give *<span>simple</span>* examples of managing *<span>complexity</span>*?  By and large, you don't.  WSF files were designed in part to manage complex, multi-file scripts, but those are exactly the sorts of scripts you *<span>don't </span>*want to use as teaching aides. </span>

<span></span>

<span>I have a sneaking suspicion that Lippert's Paradox is a heavy contributor to **<span>Object Happiness Disease</span>**, a disease remarkably similar to [Thread Happiness Disease](/ericlippert/archive/2004/02/15/73366.aspx "http://blogs.msdn.com/ericlippert/archive/2003/09/18/53046.aspx").  Those afflicted with Object Happiness use the principles of OO design whether those principles make any sense to the task at hand or not.  When you're learning about OO design, usually the pedagogic examples are not massive multi-developer complex systems, but trivial little samples.  But very few OO design texts emphasize that their examples are contrived illustrative examples, not templates to be cut and pasted. </span>

<span></span>

<span>I think that the choice to eschew WSF files for pedagogic purposes was a good one. </span>

<span></span>

<span>Include Files </span>

<span></span>

<span>from an educational point of view, we have a problem: a lot of the things that are going on in this script are “hidden” from view; that’s because they’re taking place in the include file. If our goal is to show people an example of a script that connects to a database and then performs some action, we’ve only done half the job; after all, you don’t see the code that connects to the database. </span>

<span></span>

<span>On the other hand, you don't want to write one of those books where you repeat the same boilerplate code over and over again.  Also, there's a balance to be struck between handing someone a fish and teaching them a fish -- teaching to fish is **<span>not</span>** better if you need a fish RIGHT NOW.  Case in point, Stein Borge's book "**<span>Managing Enterprise Systems With The Windows Script Host</span>**" uses include files quite a bit.  It's not so much a book on teaching about scripting as a bunch of useful off-the-shelf scripts.  All the repetitive plumbing gets talked through once and then factored out into include files. </span>

<span></span>

<span>(Full disclosure: I was the technical editor for that book.  And yes, I suggested that title.  It's an awful title, I know, but the other options were worse.  That's as short as I could get it and keep the title accurate.) </span>

<span></span>

<span>Type Library Constants </span>

<span></span>

<span>Is it worth including all the XML tags just to avoid defining these constants? In the case of the FileSystemObject, I’m tempted to say no. But what about cases involving other COM objects?</span><span> </span>

<span></span>

<span>The CDONTS email tools library has a particularly enormous number of constants defined and it is often the case that even a small program will use lots of them.  This was a fairly frequently requested feature. </span>

<span></span>

<span>The details of how the type library parsing code adds the constants to the namespace are interesting.  Well, they're interesting to me, but then again, I wrote all that code.  I might do a blog entry on how that stuff works internally sometime.  The details of resolving collisions between identically named items were tricky. </span>

<span></span>

<span>Multiple Jobs </span>

<span></span>

<span>I have to admit that this feature has me a little stumped. I can make a case for the other capabilities of .wsf files (e.g., direct access to a type library), but I’m not sure why you would want to divide a script into multiple jobs.</span><span> </span>

<span></span>

<span>That feature was my idea, and in retrospect, it didn't pan out the way I had hoped.  The problem space I was attempting to attack was the situation where you have a large number of small scripts that are all interrelated in some way.  Like, a bunch of scripts to handle backing up and restoring a particular set of files, or a bunch of scripts for deploying various bits of software onto user machines, or whatever.  I wanted to be able to stick all of those related-but-separate scripts into one file so that they could be more easily maintained. </span>

<span></span>

<span>I think it was a good idea but the implementation didn't go far enough.  The inability of the jobs to share code and easily call each other is the most common complaint about this feature. </span>

<span></span>

<span>Multiple Languages </span>

<span></span>

<span>Not only do .wsf files allow you to write scripts that have multiple jobs, but each of those jobs could theoretically be written in a different language.</span><span> </span>

<span></span>

<span>Actually, it's even better than that.  A given job can contain multiple script blocks in multiple languages, just like you can do in IE. </span>

<span></span>

<span>are there reasons why you might want to write half of a script using VBScript and half a script using Jscript?</span><span> </span>

<span></span>

<span>Yes there are, but they're pretty weak.  VBScript and JScript are not mere syntactic variations on each other; there are some things that each language does better than the other.  Someone might want to use VBScript's localization abilities and JScript's built-in </span><span>sort</span><span> method in the same script.  </span>

<span></span>

<span>The far more common scenario is that one developer has a bunch of handy script functions already written in JScript, her coworker has a bunch written in VBScript, and someone needs to use both libraries in the same script.  We got that feedback quite a bit when we were asking what problems administrators had with scripting. </span>

<span></span>

<span>Argument Parsing </span>

<span></span>

<span>When we first looked at the XML elements for arguments, we got kind of excited. Combined with the usage stuff, we thought this was going to be really cool; we thought that if you started the script without one of the required arguments that the script would automatically quit and display usage instructions. Alas, it didn’t work that way; marking an argument as required is useful for someone reading your code, but it doesn’t affect the way the script runs.  What you don’t get is a more automated way to handle arguments. You can mark an argument as required, but it’s still up to you to write the code to see if this required argument was supplied and, if it wasn’t, to take some sort of action. </span>

<span></span>

<span>Indeed, our original design was to have the WSH engine do exactly what you described.  We cut that part of the feature for three reasons.  </span>

<span></span>

<span>The first was simply lack of time -- there were a lot of features we wanted to add but [only so many devs, testers, PMs, user education people, etc, available, so only the most important features got added](http://blogs.msdn.com/ericlippert/archive/2003/10/28/53298.aspx).  </span>

<span></span>

<span>Second, we decided that it was too hard to make the feature sufficiently flexible.  You remember way back in the early days of C compilers, when if there was an error, the compiler would say "parse error, somewhere", and that was it?  OK, that's better than nothing, but it's not very friendly.  When a user passes bad arguments to a program, usually what you want to do is display the "usage" text, but often you want to provide more information.  Or, conversely, you want to accept a somewhat complicated and interrelated set of arguments, like *<span>"if /frob is specified then you need to specify /source: and /dest: also"</span>*.  Etc, etc, etc.  </span>

<span></span>

<span>It would have been fairly hard work to make just a straight syntax verifier.  Putting the extra work in to allow custom error messages and bizarre syntaxes was too much work for too little gain.  I didn't want the feature to be half-baked. </span>

<span></span>

<span>Third, the verification code really wasn't that hard for the end user to write in script once we added the named and unnamed arguments collections.  It's the string parsing code that's a pain in the rear, so we concentrated on making the straightforward dictionary lookup work well rather than trying to come up with some sort of YACC-like automagical-parse-error-handler.</span>

<span></span> 

</div>

</div>

</div>

