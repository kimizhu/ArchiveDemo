<div id="page">

# Why do the script engines not cache dispatch identifiers?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/16/2003 8:44:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">You COM programmers out there are intimately familiar with the </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">IDispatch</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> interface, I'm sure.<span style="mso-spacerun: yes">  </span>To briefly sum up for the rest of you, the point of </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">IDispatch</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> is that it allows a caller to call a function without actually knowing **at compile time** any details about the name or signature of that function.<span style="mso-spacerun: yes">  </span>The caller passes a method name to </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">IDispatch::GetIdsOfNames</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> to get a dispatch identifier -- an integer -- which identifies the method, and then calls </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">IDispatch::Invoke</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> with the dispid and the arguments.<span style="mso-spacerun: yes">  </span>The implementation of Invoke is then responsible for analyzing the arguments and calling the actual function on the object's virtual function table.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">This is how VBScript and JScript work.<span style="mso-spacerun: yes">  </span>When you say</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Set Frob = CreateObject("BitBucket.Froboznicator")</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Frob.Gnusto 123, "skidoo"</span><span style="FONT-SIZE: 10pt; FONT-FAMILY: Arial"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">what VBScript actually does is pass "Gnusto" to </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">GetIdsOfNames</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, and then passes the dispid and arguments to </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Invoke</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.<span style="mso-spacerun: yes">  </span>The object then does the work of actually translating that into a real function call.<span style="mso-spacerun: yes">  </span>This is how we get away with not having to write a down-to-the-machine-code compiler for VBScript and JScript.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">One of the Fundamental Rules of OLE Automation is that for any object, any two calls to </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">GetIdsOfNames</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> with the same string must return the same number.<span style="mso-spacerun: yes">  </span>This is so that callers can fetch the dispid once and reuse it, rather than having to fetch it anew every time they try to invoke the dispatch object.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">But VBScript and JScript do not cache the dispid.<span style="mso-spacerun: yes">  </span>If you say</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Set Frob = CreateObject("BitBucket.Froboznicator")</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Frob.Gnusto 123, "skidoo"</span><span style="FONT-SIZE: 10pt; FONT-FAMILY: Arial"> </span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Frob.Gnusto 10, "lords a leaping"</span><span style="FONT-SIZE: 10pt; FONT-FAMILY: Arial"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">then the script engine will call </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">GetIdsOfNames</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> twice, and get the same value both times.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Surely this is wasteful.<span style="mso-spacerun: yes">  </span>Can't we cache that thing?<span style="mso-spacerun: yes">  </span>I mean, it is a small optimization; that call to Invoke is going to dwarf the expense of the </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">GetIdsOfNames</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.<span style="mso-spacerun: yes">  </span>It just seems like there ought to be something we could do here.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Appearances can be deceiving.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">You might first think that every time we see </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Frob.Gnusto</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> we can use the dispid we grabbed the first time and reuse it.<span style="mso-spacerun: yes">  </span>For example:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Frob.Gnusto 123, "skidoo" ' OK, Gnusto is dispid 0x1111.<span style="mso-spacerun: yes">  </span>Add to cache "Frob.Gnusto", 0x1111</span><span style="FONT-SIZE: 10pt; FONT-FAMILY: Arial"> </span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Frob.Gnusto 10, "lords a leaping"<span style="mso-spacerun: yes">  </span>' Look up "Frob.Gnusto" in cache, aha, it is 0x1111</span><span style="FONT-SIZE: 10pt; FONT-FAMILY: Arial"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">This doesn't work.<span style="mso-spacerun: yes">  </span>Consider this scenario:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Set Frob = CreateObject("BitBucket.Froboznicator")</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Frob.Gnusto 123, "skidoo" </span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Set Frob = CreateObject("MushySoft.Gronker")</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Frob.Gnusto 10, "lords a leaping"<span style="mso-spacerun: yes">  </span></span><span style="FONT-SIZE: 10pt; FONT-FAMILY: Arial"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Two objects sharing the same variable but with different types may have the same method name but different dispids.<span style="mso-spacerun: yes">  </span>We would have to invalidate our cache every time the variable was changed, which is a lot of work to save very little time.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">There are other reasons why this doesn't work.<span style="mso-spacerun: yes">  </span>Consider this JScript code:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">var Frob = new ActiveXObject("BitBucket.Froboznicator");</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">var Frab = new ActiveXObject("BitBucket.Flouncer");</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Frob.Gnusto(123, "skidoo"); // Cache "Frob.Gnusto", 0x1111</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">with(Frab)</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">{</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial"><span style="mso-tab-count: 1">      </span>Frob.Gnusto(10, "lords a leaping");</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">}<span style="mso-spacerun: yes">  </span></span><span style="FONT-SIZE: 10pt; FONT-FAMILY: Arial"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">OK, now is </span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">that</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Frab.Frob.Gnusto(10...)</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> , or </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Frob.Gnusto(10...)</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">?<span style="mso-spacerun: yes">  </span></span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Frab</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> might have a method </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Frob</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> which has a method </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Gnusto</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> which has a different dispid.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Similarly, there are problems with multiple scopes where local variables may shadow globals.<span style="mso-spacerun: yes">  </span>It's a huge mess.<span style="mso-spacerun: yes">  </span>The net result is that **we can't cache dispids against variable names**.<span style="mso-spacerun: yes">  </span>The variable names are just too ephemeral.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">What about the object POINTERS?<span style="mso-spacerun: yes">  </span>Every object has a unique 32 bit pointer associated with it, right?<span style="mso-spacerun: yes">  </span>Why not cache the dispids against the pointer-and-method-name pair? </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Unfortunately, that doesn't work either.<span style="mso-spacerun: yes">  </span>Suppose object </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Frob</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> is 0x12345000 in memory.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Frob.Gnusto 123, "skidoo" ' OK, Gnusto is dispid 0x1111.<span style="mso-spacerun: yes">  </span>Add to cache "0x12340000.Gnusto", 0x1111</span><span style="FONT-SIZE: 10pt; FONT-FAMILY: Arial"> </span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Frob.Gnusto 10, "lords a leaping"<span style="mso-spacerun: yes">  </span>' Look up "0x12340000.Gnusto" in cache, aha, it is 0x1111</span><span style="FONT-SIZE: 10pt; FONT-FAMILY: Arial"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">This may look fine, but again, it isn't.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Set Frob = CreateObject("BitBucket.Froboznicator")</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Frob.Gnusto 123, "skidoo"</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Set Frob = CreateObject("MushySoft.Gronker")</span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: Arial">Frob.Gnusto 10, "lords a leaping"<span style="mso-spacerun: yes">  </span></span><span style="FONT-SIZE: 10pt; FONT-FAMILY: Arial"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">What happened on that third line of code there?<span style="mso-spacerun: yes">  </span>We threw away the only reference to the current value of </span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Frob</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">, freeing the pointer.<span style="mso-spacerun: yes">  </span>Then the operating system went and created a new object.<span style="mso-spacerun: yes">  </span>The operating system could be very smart about pointer re-use.<span style="mso-spacerun: yes">  </span>It knows that it has a perfectly good free pointer at 0x12340000, so it might re-use it.<span style="mso-spacerun: yes">  </span>Now our cache needs to be invalidated every time an object is freed\!<span style="mso-spacerun: yes">  </span>We must keep track of when every single object is freed -- basically, we have to write a garbage collector for arbitrary COM objects\!</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

**<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">To cache dispids you need to ensure that the lifetime of the object is greater than the lifetime of the cache</span>**<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">.<span style="mso-spacerun: yes">  </span>But object lifetimes are so variable, it is very hard to know when the cache is invalid. We once considered adding dispid caching to those objects where we knew that the objects would live longer than the script -- the "</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">window</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">" object in IE, or the "</span><span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Sans Unicode&#39;">Response</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">" object in IIS, but rejected the proposal as too much complication for very little performance gain.</span>

</div>

</div>

