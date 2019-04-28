# Unicode output and the Windows Script Host

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/11/2004 11:41:00 AM

-----

I'm working on a prolix essay on the future of declarative programming languages, but I've been sidelined by, horrors, actually having to do lots of real work on the next version of VSTO these last couple weeks.  Expect that rant sometime in the next week or so.  

In the meanwhile, a coworker asked me the other day what the //U "use Unicode" switch does in the console version of Windows Script Host.   *"Is this so that the host can read in script files that are saved in Unicode?"* he asked. 

Not quite -- actually it is the *output* that we are worried about, not the *input*.  The script engine (VBScript, JScript) expects the source code to be in a standard UTF-16 BSTR.  WSH  is responsible for getting the source code off the disk in whatever format and turning it into UTF-16 in memory.  WSH does this exactly the way you'd expect -- it calls IsTextUnicode and then does whatever is necessary to get it into UTF-16.  (Reverse bytes, strip the order mark, call MultiByteToWideChar, whatever.)  The //U flag doesn't affect this at all. 

What we don't know is what *output* encoding the user wants.  In short, the **//U flag only causes a change in behaviour when you are running cscript.exe on an NT-based operating system and redirecting the output to a file. ** To expand on that a bit: 

  - If you're running cscript.exe on an operating system that descends from the NT kernel (Windows 2000, XP, Server, Longhorn… you get the picture) and you're dumping text (WScript.Echo, error messages, whatever) **to a console window**, we always call WriteConsoleW  no matter whether the //U flag was passed or not.  That is, we always dump the text as Unicode and let the console sort out how to display it.

  - If you are running cscript.exe on an NT-based OS **and you've redirected your console output to a file**, and you did NOT specify the //U switch then we dump the text as ANSI by calling WideCharToMultiByte(CP\_OEMCP).

  - In the same scenario but with the //U switch we dump the text as UTF-16.

  - When running cscript.exe on any Windows 95 descendent (95, 98, ME) we always dump text as ANSI whether you're redirecting output or not.

The behaviour of the standard streams is kind of complicated depending on whether you're opening WScript.StdErr, StdIn, StdOut, on NT-based or 95-based OS, and whether streams are redirected or not, but suffice to say that basically it's the same as above.

