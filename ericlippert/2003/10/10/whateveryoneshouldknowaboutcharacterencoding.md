# What Everyone Should Know About Character Encoding

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/10/2003 4:41:00 PM

-----

Thank goodness Joel wrote this article -- that means that I can cross it off of my list of potential future blog entries\!  Thanks Joel\!

 

 

<http://www.joelonsoftware.com/articles/Unicode.html> 

 

 

Fortunately the script engines are entirely Unicode inside.  Making sure that the script source code passed in to the engine is valid UTF-16 is the responsibility of the host, and as Joel mentions, IE certainly jumps through some hoops to try and deduce the encoding.  WSH also has heuristics which try to determine whether the file is UTF-8 or UTF-16, but nothing nearly so complex as IE.

 

I should mention that in JScript you can use the u0000 syntax to put unicode codepoints into literal strings.  In VBScript it is a little trickier -- you need to use the CHRW method.

