<div id="page">

# WSC vs WSH

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/8/2003 4:57:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Following up on this morning's entry, a reader asked me why Windows Script Components don't have access to the WScript object.<span style="mso-spacerun: yes">  </span>"*it IS running in an instance of WSH isnt it?"* </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">No, it isn't.<span style="mso-spacerun: yes">  </span>That's a common misperception.<span style="mso-spacerun: yes">  </span>Let me clear it up.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Basically the whole point of scripting is to make it easy to write programs.<span style="mso-spacerun: yes">  </span>Two very common kinds of programs are **executables** and **components**.<span style="mso-spacerun: yes">  </span>I'm sure that you understand the difference between the two, but just let me emphasize that executables are "standalone" -- they have a well defined startup, they run for a while, and then they shut down.<span style="mso-spacerun: yes">  </span>Components are libraries of useful functionality packaged up as objects.<span style="mso-spacerun: yes">  </span>These objects cannot live on their own -- they have to be created by an executable or another component, and they live as long as the owner wants them to.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">We provide two ways to write "executables in script" -- Windows Script Host files (.VBS, .JS, .WSF) and HTML Applications (.HTA).<span style="mso-spacerun: yes">  </span>When you run one of these, the host application starts up, loads the script, runs it, and then shuts down.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">We also provide a way to write components in script -- the aptly named Windows Script Components.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">When you create a WSC, its no different from creating any other in-process COM object.<span style="mso-spacerun: yes">  </span>The fact that the object is implemented in script is irrelevant. <span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">People get WSCs and WSF's confused because of their similar names and syntaxes.<span style="mso-spacerun: yes">  </span>But logically, WSCs and WSH are completely separate entities.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Note though that under the covers, quite a bit of the WSF processing code is actually implemented in ScrObj.DLL, the WSC engine, which itself is consumed as a component by the WSH executable.<span style="mso-spacerun: yes">  </span>We saw no reason to implement two identical XML parsers, two debugger interfaces, etc, when we had one sitting right there in a DLL already.</span>

</div>

</div>

