<div id="page">

# Rumours of VBScript's Death Have Been Greatly Exaggerated

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/9/2004 10:47:00 AM

-----

<div id="content">

<span>A reader recently left the following [comment](/ericlippert/archive/2004/03/11/88308.aspx#110149 "http://weblogs.asp.net/ericlippert/archive/2004/03/11/88308.aspx#110149") today which deserves a full and detailed response.  (Yep, I'm going to get prolix again.)  </span>

<span></span>

> <span>I teach a Windows network administration scripting course to about 1500 admins and auditors each year.  I've been using VBScript all along **<span>mainly because it's a good transition language to VBA/VB.NET</span>** ... but I will likely switch the courseware to ActiveState's Win32 Perl instead.  </span>
> 
> <span></span>
> 
> <span>It seems like MS is going to let VBScript die a slow death and **<span>I'm doing a disservice to my students</span>** by making them invest their time in it.  I think MS should definitively state its intentions for the future of VBScript on the Script Center website and perhaps start doing all of its example administration and ResKit scripts in JScript (or Perl) instead so as not to trick new scripters into wasting their time.  </span>
> 
> <span></span>
> 
> <span>It's too bad, I still think VBScript is easy to teach and a gentle intro to VBA/VB.NET, but if the language isn't going anywhere, then I have to look out for my students' long-term interests.  Perl looks like it will become the de facto **<span>cross-platform administration language</span>** (if it isn't already) and there's a gigantic and solid community of support behind it (whereaas MS seems to just abandon its user base sometimes).  </span>
> 
> <span></span>
> 
> <span>If I'm wrong here about VBScript's future, then please tell me otherwise.  Thanks for being straightforward and honest in this blog.  </span>

<span>\[emphasis added\] </span>

<span></span>

<span>There's a lot to respond to in there.  </span>

<span></span>

<span>Let me first of all state categorically Microsoft's position on the COM Scripting technologies.  It's not often that I get to be **<span>The Voice Of Microsoft</span>**, but here we go: </span>

<span></span>

**<span>We will continue to support VBScript and JScript for the foreseeable future.</span>**<span>  Obviously VBScript, JScript, WSH, etc, must continue to be shipped with the operating system forever, as huge amounts of existing business-critical code depends upon them. To characterize that as "dying a slow death" is excessively melodramatic.  We expect that the unmanaged COM scripting languages will continue to be useful for many, many years.  The Visual Studio Sustaining Engineering Team presently is responsible for VBScript, JScript, Windows Script Components, Windows Script Host, etc.  </span>

<span></span>

<span>I'm looking at the logs right now, and there have been 702 file changes in the last three years, almost all bug fixes and security improvements. As we find bugs, security issues, etc, the Sustaining Engineering Team will investigate them and **<span>issue new releases with operating system service packs as necessary</span>**, as we have always done.  (And I will code-review their changes\!)  </span>

<span></span>

<span>However, **<span>there will be</span>** **<span>no new features added to the languages</span>** -- indeed, there have been no new features in a long time.  The last person to actually add a new feature to any script team technology was me, on November 1st 2000\!  (The feature was adding code signing support to WSF files.) </span>

<span></span>

<span>An analogy might be helpful.  I have plenty of old hammers in my toolbox, and they do their intended job today just as well as they always have.  To criticize a hammer for not being a cool new hydrogen-powered nailgun (yes, they do exist -- my kitchen contractors had one), or for being a bad tool to build skyscrapers, is to miss the rather important point that a hammer is still a useful tool for a broad variety of tasks\!  There have been very few massive improvements in hammer technology in recent years, but it would be a serious mistake to say that hammers are dying a slow death.  It would also be a mistake to *not teach people how to use hammers* because hydrogen powered nailguns exist.  VBScript is a hammer, not a stone axe.  It'll be around for a long time. </span>

<span></span>

<span>I've been meaning to blog more about script languages as pedagogic tools, and now seems like as good a time as any. </span>

<div>

<span></span>

</div>

<span></span>

<span>First off, let's think about theory.  As I've [blogged before](/ericlippert/archive/2004/03/01/82168.aspx "http://weblogs.asp.net/ericlippert/archive/2004/03/01/82168.aspx"), the important thing when you're learning how to program is to understand the semantics of the language, to understand what abstractions the language provides and use them appropriately to manipulate data correctly.  Therefore, the question "what is the future of VBScript?" is somewhat irrelevant.  If you're teaching people to program, pick a language that teaches people to program.  Once they know how to program, they'll be able to move from language to language with relative ease.  A loop is a loop is a loop, whether it is </span><span>for(;;)</span><span> or </span><span>While Blah</span><span> or some other goofy syntax. </span>

<span></span>

<span>From that approach, the sensible question to ask is "*<span>what language most easily teaches the concepts that I want the students to learn?</span>*"  and not "*<span>what language is going to have new features next year?</span>*"  When I was learning how to program over the years I used Commodore Pet BASIC, Waterloo Structured BASIC, Pascal, Lisp, Scheme, New Jersey Standard ML, Java, C, C++, Ada and Turing.  (The latter was a language specifically designed at the University of Toronto for teaching programming concepts.)  Aside from C++, I *<span>use</span>* none of those languages for anything practical, but I use the *<span>concepts</span>* that I picked up in each every day.  I often use functional programming techniques, for example.  If you want to teach someone functional techniques then it makes sense to pick a functional language. Teach the concept and *<span>then</span>* show how to do the same thing (much less elegantly, though it is improving\!) in C\# or whatever their language of choice is. </span>

<span></span>

<span>Clearly you understand the value of this approach, as you stated twice in your comment that VBScript is great because it is a good introduction to more complex languages such as VBA and VB.NET, which are useful for more heavy-duty application programming tasks.  But [listen -- do you smell something](http://members.fortunecity.com/wavjunky/swl-l/listensm.wav "http://members.fortunecity.com/wavjunky/swl-l/listensm.wav")? </span>

<span></span>

<span>I can smell the skepticism from here.  Why the heck would you or your students care about this kind of theory?  They're not writing dissertations on the theory of computing machinery, they're writing logon scripts.  They're not going to have to design massive applications, they're going to change directory permissions. </span>

<span></span>

<span>Why do you mention -- twice\! -- that you use VBScript because it affords a smooth ramp up to VBA and VB.NET?  They are **<span>totally useless to you if your goal is to train administrators how to do administration tasks</span>**.  You are training possibly **<span>the most pragmatic set of IT professionals on the planet</span>**.  Administrators do not care about highfalutin' programming concepts like functional programming or object oriented programming.  Administrators do not *<span>need</span>* the language features afforded by VB.NET.  Administrators write thousand line scripts that look like this: </span>

<span></span>

<span>Directory.Create("CORPSERVER/BobSmith")  
</span><span>Directory("CORPSERVER/BobSmith").Permissions.Add("BobSmith", "ReadWriteDelete")  
</span><span>Directory("CORPSERVER/BobSmith").CreateFile("Logon.BAT")  
</span><span>Directory.Create("CORPSERVER/BillJones")  
</span><span>Directory("CORPSERVER/BillJones").Permissions.Add("BillJones", "ReadWriteDelete")  
</span><span>Directory("CORPSERVER/BillJones").CreateFile("Logon.BAT")  
</span><span>... a thousand more lines of this stuff</span>

<span></span>

<span>Now, perhaps our administrator could generate a more elegant and maintainable program by, say, abstracting away those operations into a function, putting the user names into a file, and writing a processing program that sucks the names out of the file one at a time and calls the function.  That would be great, but since this is a single-use script, any time spent making it elegant and maintainable before it is deleted this afternoon is rather wasted time.  </span>

<span></span>

**<span>Administrators like to learn how to use one language well enough to get by, and then do *<span>everything</span>* in that language. </span>**

<span></span>

<span>(Actually, it's even moreso than that.  When I write an administrative script that, say, copies a thousand files from one dev box to another, you know what I do?  I get a file list in my editor and then type a vi macro that turns the line of code into a legal statement in some script language, then I run that macro a thousand times, run the resulting script, and delete it.  My leet skillz in the admin space are a function of my mastery of my editor, not my deep knowledge of programming languages\!)</span>

<span>The theoretical approach gets us nowhere.  The theoretical approach says "*<span>the semantics are the program; all languages are basically just variations on ways to abstractly express those semantics. Get the semantics right in your head and you'll be able to write the program in the language of your choice</span>*."  But for administrators, the semantics are NOT **<span>in the program</span>** nearly so much as the semantics are **<span>in the object models</span>**.  As I've said  [before](/ericlippert/archive/2003/10/16/53224.aspx "http://weblogs.asp.net/ericlippert/archive/2003/10/16/53224.aspx"), for most administrative jobs, the programming language is the thing that mediates between you and the object model.  </span>

<span></span>

<span>Therefore, let me make the **<span>same argument</span>** from a completely opposite standpoint.  The language is unimportant because you're not going to be using nine tenths of the language features anyways.  Spend enough time to get the basics down on some language, any language, and then spend the whole time studying really kick-ass administrator object models like WMI. </span>

<span></span>

<span>The question is now neither "*<span>what's the best **<span>pedagogic language</span>**</span>*?" nor "*<span>what **<span>language</span>** will have **<span>new features</span>**</span>*?" The question to be answered now is "*<span>What language best mediates between administrators and today's object models</span>*?"  </span>

<span></span>

<span>And, if you're future-proofing, another good question is "*<span>what language best mediates between administrators and **<span>future</span>** object models</span>*?" </span>

<span></span>

<span>Clearly your choice depends on what object models you think that your students will be using to do their jobs.  Do you think that people will continue to use COM-based object models like WMI for a while?  Then a COM-based scripting language like VBScript or ActiveState's Perl might be a good idea.  Whether I'm going to add new features to VBScript is irrelevant to that decision -- VBScript exists now, Perl exists now, and if either of those are useful languages to call WMI, then by all means, teach whichever one you think your students will pick up on better. </span>

<span></span>

<span>But what about the future?  You wisely asked these questions in the first place with an eye to the future.  Let me again be the Voice of Microsoft here and say what the future is: </span>

<span></span>

**<span>Managed programming, the .NET Framework and the Longhorn WinFX classes are the way of the future.</span>**<span> **<span> We are betting the entire company on it.  </span>**That's why I haven't been adding new features to VBScript for the last three years\!  Yes, we will continue to support COM and VBScript forever, but if you want to enable people to take full advantage of all the incredible coolness that is coming in Longhorn, **<span>start thinking hard about managed code and the .NET Framework right now. </span>** Learn as much as you possibly can about [**<span>Monad</span>**](http://www.eweek.com/article2/0,4149,1550180,00.asp) and the **<span>Longhorm APIs</span>**, and start laying the groundwork for that curriculum.  Administrators in the future will use Monad and other managed languages to mediate between them and the WinFX framework, I guarantee it. </span>

<div>

<span></span>

</div>

<span></span>

<span>As I've [blogged before](http://blogs.msdn.com/ericlippert/archive/2004/03/08/86177.aspx), it is sometimes hard to know what tool to choose for a particular task.  Clarify your requirements.  Put these in priority order -- a good language for my curriculum </span>

<span></span>

1.  <span>works well with existing object models (VBScript)</span>
2.  <span></span><span>works well with future object models (any .NET language)</span>
3.  <span></span><span>affords a smooth ramp up to some other more complex language technology (VB.NET, C\#...)</span>
4.  <span></span><span>is available on every platform (perl)</span>
5.  <span></span><span>makes it easy to learn basic general-purpose algorithmic programming techniques (your choice)</span>
6.  <span></span><span>makes it easy to learn specific advanced programming techniques (OOP, functional programming, etc.) (JScript)</span>
7.  <span></span><span>has a large installed base, lots of documentation, etc (any established language)</span>
8.  <span></span><span>has lots of new features planned (C\#, VB.NET)</span>
9.  <span></span><span>is cool (C\#, VB.NET…)</span>

<span></span>

<span>Once you've got those in priority order, it should become easier to pick the right tool.  If \#4 is the most important to you, we shouldn't even be talking about this.  Use perl and be done with it if you want admins to be able to write programs for Solaris boxes\!  </span>

<span></span>

<span>If you have multiple conflicting priorities, well, not much I can do to solve that problem except by providing as many facts and as much analysis here as I can.  </span>

<span></span>

<span>My aim is and has always been to make the Windows platform as attractive as possible to all programmers, from administrative scripters to game designers to developers of multi-tiered enterprise developers.  If perl on Windows is the best tool for your students, by all means, use it.  I'll be happy as long as that "on Windows" is in there\! </span>

<span></span>

<span></span>

</div>

</div>

