<div id="page">

# Eval is Evil, Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/4/2003 2:23:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">As I promised, more information on why eval is evil.  (We once considered having T-shirts printed up that said "Eval is evil\!" on one side and "Script happens\!" on the other, but the PM's never managed to tear themselves away from their web browsing long enough to order them.)</span>

 <span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Incidentally, a buddy of mine who is one of those senior web developer kinda guys back in Waterloo sent me an email yesterday saying "Hello, my name is Robert and I am an evalaholic".  People, it wasn't my *<span style="FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">intention</span>* to start a twelve step program, but hey, whatever works\! </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">As I [discussed the other day](http://blogs.gotdotnet.com/ericli/PermaLink.aspx/5f27ae83-ff82-4fea-97db-b6fef3922c3b), </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">eval</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> on the client is evil because it leads to sloppy, hard-to-debug-and-maintain programs that consume huge amounts of memory and run unnecessarily slowly even when performing simple tasks.<span style="mso-spacerun: yes">  </span>But like I said in my [performance rant](http://blogs.gotdotnet.com/ericli/PermaLink.aspx/bfd7cd56-caf1-47e5-b94a-0c812b1be28e), if it's **<span style="FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">good enough</span>**, then hey, it's good enough.<span style="mso-spacerun: yes">  </span>Maybe you don't **need** to write maintainable, efficient code.<span style="mso-spacerun: yes">  </span>Seriously\! Script is often used to write programs that are used a couple of times and then thrown away, so who cares if they're slow and inelegant?</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">But </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">eval</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> on the server is an entirely different beast.<span style="mso-spacerun: yes">  </span>First off, server scenarios are generally a lot more performance sensitive than client scenarios.<span style="mso-spacerun: yes">  </span>On a client, once your code runs faster than a human being can notice the lag, there's usually not much point in making it faster.<span style="mso-spacerun: yes">  </span>But<span style="mso-spacerun: yes">  </span>[as I mentioned earlier](http://blogs.gotdotnet.com/ericli/PermaLink.aspx/ae29f2ce-b06d-47a1-9520-857c350e3988), ASP goes to a lot of work to ensure that for a given page, the compiler only runs once. An </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">eval</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> defeats this optimization by making the compiler run every time the page runs\! On a server, going from 25 ms to 40 ms to serve a page means going from 40 pages a second to 25 pages a second, and that can be expensive in real dollar terms.<span style="mso-spacerun: yes">  </span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">But that's not the most important reason to eschew </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">eval</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> on the server.<span style="mso-spacerun: yes">  </span>Any use of </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">eval</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> (or its VBScript cousins </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">[Eval<span style="FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">, </span>Execute<span style="FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> and </span>ExecuteGlobal](http://blogs.gotdotnet.com/ericli/PermaLink.aspx/4939ad1e-b2d7-436e-a2dc-bd7665d207bf)</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">) is a **potentially enormous security hole.<span style="mso-spacerun: yes">  </span>** </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">\<%</span><span style="COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>var Processor\_ProductList;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>var Software\_ProductList;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>var HardDisk\_ProductList;</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>// ...</span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">  CategoryName = Request.QueryString("category");</span><span style="COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">  ProductList = eval(CategoryName & "\_ProductList");</span><span style="COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"> </span>

<span style="COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;"><span style="mso-spacerun: yes">  </span>// ...</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">What's wrong with this picture?<span style="mso-spacerun: yes">  </span>**The server assumes that the client is not hostile.**<span style="mso-spacerun: yes">  </span>Is that a warranted assumption?<span style="mso-spacerun: yes">  </span>Probably not\!<span style="mso-spacerun: yes">  </span>You know nothing about the client that sent the request.<span style="mso-spacerun: yes">  </span>Maybe **your** client page only sends strings like "Processor" and "HardDisk" to the server, but anyone can write their own web page that sends </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">((new ActiveXObject('Scripting.FileSystemObject')).DeleteFile('C:\*.\*',true)); Processor</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">which will cause </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">eval</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> to evaluate</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">((new ActiveXObject('Scripting.FileSystemObject')).DeleteFile('C:\*.\*',true)); Processor\_ProductList</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Obviously that's a pretty unsophisticated attack.<span style="mso-spacerun: yes">  </span>The attacker can put [any code in there that they want](http://blogs.gotdotnet.com/ericli/PermaLink.aspx/c40acab4-2215-40c6-999c-2ad2d0d05833), and it will run in the context of the server process.<span style="mso-spacerun: yes">  </span>Hopefully the server process is not a highly privileged one, but still, there's vast potential for massive harm here just by screwing up the logic on your server.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

**<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Never trust the input to a server, and try to never use </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">eval</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> on a server.<span style="mso-spacerun: yes">  </span></span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">Eval</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> injection makes SQL injection look tame\!</span>**

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">To try and mitigate these sorts of problems, JScript .NET has some restrictions on its implementation of </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">eval</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">, but that's a topic for another entry.</span>

 <span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

</div>

</div>

