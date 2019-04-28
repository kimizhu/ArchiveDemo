# Binary Files and the File System Object Do Not Mix

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/20/2005 2:47:00 PM

-----

OK, back to scripting today. But before I get back to scripting issues, one brief correction. An attentive reader noted that "The Well-Tempered Clavier" was in fact designed to sound good on a "well tempered" instrument, not an "equally tempered" instrument. The difference is that a "well" temperament is designed so that every key sounds good, but is allowed to have some badly-out-of-tune intervals that must be avoided. (Traditionally these are called "wolf intervals".) There was considerable controversy when equal temperament was introduced in Europe. I suppose it was the "what is the One True Bracing Style?" ridiculous issue of the day. Another commenter pointed out that you could translate my wav-writing program into VBScript by using the File System Object to write out the bytes. To simplify their code down to a program that writes out individual bytes: ' DO NOT DO THIS  
Set FSO=CreateObject("Scripting.FileSystemObject")  
Set File=FSO.CreateTextFile("c:\\test.bin", True)  
For i = 0 to 255  
  File.Write Chr(i)  
Next  
File.Close And sure enough, this writes out a binary file consisting of those bytes. Please don't do that. See that line that says "CreateTextFile"? We wrote that method to **create a text file**, not a binary file. Though this code might appear to work, it actually does not. Text files are more than just binary files that can be interpreted as text. Text files have to conform to certain rules to ensure that they can be sensibly interpreted as text in the local code page. If that's not 100% clear to you, read [Joel's article on the subject](http://www.joelonsoftware.com/articles/Unicode.html)before we go on. Let me give you an example that *clearly* fails. What does this program do? Set FSO=CreateObject("Scripting.FileSystemObject")  
Set File=FSO.CreateTextFile("c:\\test.bin", True)  
For i = 0 to 255  
  File.Write Chr(\&hE0)  
Next  
File.Close If you said "it writes out a binary file consisting of 256 E0 bytes," bzzt\! Sorry, try again. The correct answer is "it writes out a binary file consisting of 256 E0 bytes on any operating system where the user's default ANSI code page does not define E0 as a lead byte in a DBCS encoding, like, say, Japanese, in which case it writes out 256 zeros." In the Japanese code page, just-plain-chr(E0) is **not even a legal character**, so Chr will turn it into a zero.  If I were whipping up a little one-off program on my own to write out a binary file -- well, I'd personally do it in C, but I can see how some people might want to do it in script. But there's a big difference between writing a one-off program that you're going to delete in five minutes, and writing a general-purpose utility program that you expect people around the world will use. That's an entirely different standard of robustness and portability. **Do not use the FSO to read/write binary files, you're just asking for a world of hurt as soon as someone in DBCS-land runs your code.** I have been asked many times over the years if I know of a scriptable object that can read-write true binary files in all locales. I do not. Anyone have any suggestions? I would have thought given the number of people that have asked me, that some third party would have come up with something decent by now.

