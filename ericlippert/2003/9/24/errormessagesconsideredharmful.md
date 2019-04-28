# Error Messages Considered Harmful

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/24/2003 1:07:00 PM

-----

 

 

 

 

My office nameplate was entirely apropos; LoadPicture continued to plague me throughout my career. During the Windows Security Push last year we finally turned it off. When VBScript is running in IE, the LoadPicture method causes an illegal function call exception.   

 

 

Why's that? There's a big security hole in LoadPicture. Not because its buggy, but because it works too well\!   

 

 

When I added LoadPicture to VBScript I considered the security implications, but did I mention this was in my second week at Microsoft?  I knew very little about security then.  I reasoned that sure, this is a method that can access your hard disk, but what's the worst it can do?  It can open a locally stored picture file.  Big deal\!  It can't write any information back to the disk, so the risk seems very low.

 

 

Now, of course, we use the formal threat modeling process that my colleagues Mike Howard and David Leblanc describe in their book "Writing Secure Code".  We brainstorm possible threats and vulnerabilities, document them, and ensure that the code is built to resist those threats and mitigate those vulnerabilities.  But in the bad old days the kind of off-the-cuff informal analysis I've just described was pretty typical.

 

 

The vulnerability in LoadPicture is that an attacker can write a web page which, when you visit the page, **reports back to the web server your user name, what programs you have installed and potentially the names of your files**.  This is an **information leaking** vulnerability.  It can't actually harm you directly, but **attackers want to know everything about your machine**.  Knowing what all the user names are is a good first step in guessing their passwords.  Knowing what programs you have installed tells them whether you've installed the latest patches, and hence whether you're a potential target.  And of course there is also the privacy angle.  It isn't anybody else's business whether you have Office installed or not\!

 

 

The vulnerability exists because, ironically, we did too good a job of reporting error messages.  LoadPicture succeeds if you give it a valid path to an image file.  But if you give it an invalid path, it gives you different error messages for "that's not a legal path", "that's a legal path to a file which is not an image", and "that's a legal path, but it’s a directory, not a file".  An attacker can make guesses about where various files and directories are on your system. The "Documents and Settings" folder contains a directory for every user on the machine. Thus the user names could be discovered via a dictionary attack (or brute-force attack; user names are usually short). 

 

 

Of course, it could be worse.  There was a bug in early versions of the CLR (which I believe was fixed before the first beta shipped, fortunately) where you could get an error message something like

 

 

Path discovery security exception: You are not allowed to determine the name of directory 'c:foobar'

 

 

Super\! Thanks for letting me know\!

 

 

The moral of the story is that as developers we tend to design code that produces the error messages that we need to be able to understand the problem and fix it.  Unfortunately, those are the same data that attackers need to understand the system to break it.  We've got to be careful not to leak information in error messages to insecure callers.

