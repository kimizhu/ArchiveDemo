<div id="page">

# Error Messages Considered Harmful

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/24/2003 1:07:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">My office nameplate was entirely apropos; </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">LoadPicture</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> continued to plague me throughout my career. During the Windows Security Push last year we finally turned it off. When VBScript is running in IE, the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">LoadPicture</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> method causes an illegal function call exception.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Why's that? There's a big security hole in </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">LoadPicture</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">. Not because its buggy, but because it works too well\!<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">When I added </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">LoadPicture</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> to VBScript I considered the security implications, but did I mention this was in my second week at Microsoft?<span style="mso-spacerun: yes">  </span>I knew very little about security then.<span style="mso-spacerun: yes">  </span>I reasoned that sure, this is a method that can access your hard disk, but what's the worst it can do?<span style="mso-spacerun: yes">  </span>It can open a locally stored picture file.<span style="mso-spacerun: yes">  </span>Big deal\!<span style="mso-spacerun: yes">  </span>It can't write any information back to the disk, so the risk seems very low.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Now, of course, we use the formal threat modeling process that my colleagues Mike Howard and David Leblanc describe in their book "Writing Secure Code".<span style="mso-spacerun: yes">  </span>We brainstorm possible threats and vulnerabilities, document them, and ensure that the code is built to resist those threats and mitigate those vulnerabilities.<span style="mso-spacerun: yes">  </span>But in the bad old days the kind of off-the-cuff informal analysis I've just described was pretty typical.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The vulnerability in </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">LoadPicture</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> is that an attacker can write a web page which, when you visit the page, **reports back to the web server your user name, what programs you have installed and potentially the names of your files**.<span style="mso-spacerun: yes">  </span>This is an **information leaking** vulnerability. <span style="mso-spacerun: yes"> </span>It can't actually harm you directly, but **attackers want to know everything about your machine**.<span style="mso-spacerun: yes">  </span>Knowing what all the user names are is a good first step in guessing their passwords.<span style="mso-spacerun: yes">  </span>Knowing what programs you have installed tells them whether you've installed the latest patches, and hence whether you're a potential target.<span style="mso-spacerun: yes">  </span>And of course there is also the privacy angle.<span style="mso-spacerun: yes">  </span>It isn't anybody else's business whether you have Office installed or not\!</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The vulnerability exists because, ironically, we did too good a job of reporting error messages.<span style="mso-spacerun: yes">  </span></span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">LoadPicture</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> succeeds if you give it a valid path to an image file.<span style="mso-spacerun: yes">  </span>But if you give it an invalid path, it gives you different error messages for "that's not a legal path", "that's a legal path to a file which is not an image", and "that's a legal path, but it’s a directory, not a file".<span style="mso-spacerun: yes">  </span>An attacker can make guesses about where various files and directories are on your system. The "Documents and Settings" folder contains a directory for every user on the machine. Thus the user names could be discovered via a dictionary attack (or brute-force attack; user names are usually short). </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Of course, it could be worse.<span style="mso-spacerun: yes">  </span>There was a bug in early versions of the CLR (which I believe was fixed before the first beta shipped, fortunately) where you could get an error message something like</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Path discovery security exception: You are not allowed to determine the name of directory 'c:foobar'</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Super\! Thanks for letting me know\!</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The moral of the story is that as developers we tend to design code that produces the error messages that we need to be able to understand the problem and fix it.<span style="mso-spacerun: yes">  </span>Unfortunately, those are the same data that attackers need to understand the system to break it.<span style="mso-spacerun: yes">  </span>We've got to be careful not to leak information in error messages to insecure callers.</span>

</div>

</div>

