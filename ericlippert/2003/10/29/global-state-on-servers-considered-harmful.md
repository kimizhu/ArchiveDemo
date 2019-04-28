<div id="page">

# Global State On Servers Considered Harmful

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/29/2003 2:25:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The other day I noted that extending the built-in objects in JScript .NET is no longer legal in "fast mode".<span style="mso-spacerun: yes">  </span>Of course, this is still legal in "compatibility mode" if you need it, but why did we take it out of fast mode?</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">As several readers have pointed out, this is actually a kind of compelling feature.<span style="mso-spacerun: yes">  </span>It's nice to be able to add new methods to prototypes:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">String.prototype.frobnicate = function(){/\* whatever \*/}</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">var s1 = "hello";</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">var s2 = s1.frobnicate();</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">It would be nice to extend the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Math</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> object, or change the implementation of </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">toLocaleString</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> on </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Date</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> objects, or whatever.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Unfortunately, it also breaks ASP.NET, which is the prime reason we developed fast mode in the first place.<span style="mso-spacerun: yes">  </span>Ironically, it is not the additional compiler optimizations that a static object model enables which motivated this change\!<span style="mso-spacerun: yes">  </span>Rather, it is the compilation model of ASP.NET.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">I discussed earlier how ASP uses the script engines -- ASP translates the marked-up page into a script, which it compiles once and runs every time the page is served up.<span style="mso-spacerun: yes">  </span>ASP.NET's compilation model is similar, but somewhat different.<span style="mso-spacerun: yes">  </span>ASP.NET takes the marked-up page and translates it into a **class** that extends a standard page class.<span style="mso-spacerun: yes">  </span>It compiles the derived class once, and then every time the page is served up **it creates a new instance of the class** and calls the </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Render</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> method on the class.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">So what's the difference?<span style="mso-spacerun: yes">  </span>The difference is that multiple instances of multiple page classes may be running in the same application domain.<span style="mso-spacerun: yes">  </span>In the ASP Classic model, each script engine is an entirely independent entity.<span style="mso-spacerun: yes">  </span>In the ASP.NET model, page classes in the same application may run in the same domain, and hence can affect each other.<span style="mso-spacerun: yes">  </span>We don't want them to affect each other though -- the information served up by one page should not depend on stuff being served up at the same time by other pages.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Now I'm sure you see where this is going.<span style="mso-spacerun: yes">  </span>Those built-in objects are shared by all instances of all JScript objects in the same application domain.<span style="mso-spacerun: yes">  </span>Imagine the chaos if you had a page that said:</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">String.prototype.username = FetchUserName();</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">String.prototype.appendUserName = function() { return this + this.username; };</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">var greeting = "hello";</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Response.Write(greeting.appendUserName());</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Oh dear me.<span style="mso-spacerun: yes">  </span>We've set up a race condition.<span style="mso-spacerun: yes">  </span>Multiple instances of the page class running on multiple threads in the same appdomain might all try to change the prototype object at the same time, and the last one is going to win.<span style="mso-spacerun: yes">  </span>Suddenly you've got pages that serve up the wrong data\!<span style="mso-spacerun: yes">  </span>That data might be highly sensitive, or the race condition may introduce logical errors in the script processing -- errors which will be nigh-impossible to reproduce and debug.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">A global writable object model in a multi-threaded appdomain where class instances should not interact is a recipe for disaster, so we made the global object model read-only in this scenario.<span style="mso-spacerun: yes">  </span>If you need the convenience of a writable object model, there is always compatibility mode.</span>

</div>

</div>

