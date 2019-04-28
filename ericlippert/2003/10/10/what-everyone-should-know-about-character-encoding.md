<div id="page">

# What Everyone Should Know About Character Encoding

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/10/2003 4:41:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Thank goodness Joel wrote this article -- that means that I can cross it off of my list of potential future blog entries\!<span style="mso-spacerun: yes">  </span>Thanks Joel\!</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><http://www.joelonsoftware.com/articles/Unicode.html> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Fortunately the script engines are entirely Unicode inside.<span style="mso-spacerun: yes">  </span>Making sure that the script source code passed in to the engine is valid UTF-16 is the responsibility of the host, and as Joel mentions, IE certainly jumps through some hoops to try and deduce the encoding.<span style="mso-spacerun: yes">  </span>WSH also has heuristics which try to determine whether the file is UTF-8 or UTF-16, but nothing nearly so complex as IE.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"></span> 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">I should mention that in JScript you can use the u0000 syntax to put unicode codepoints into literal strings.  In VBScript it is a little trickier -- you need to use the CHRW method.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

</div>

</div>

