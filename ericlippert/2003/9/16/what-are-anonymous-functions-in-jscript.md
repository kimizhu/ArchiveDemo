<div id="page">

# What Are "Anonymous Functions" In JScript?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/16/2003 7:57:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">One of our excellent customer support staff in the United Kingdom asked me this morning what "anonymous functions" are in JScript.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">It's a little complicated. Two things come to mind: realio-trulio anonymous functions, and what we used to call "scriptlets".</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">First up are actual anonymous functions -- functions which do not have names are, unsurprisingly enough, called "anonymous functions".</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">"**What the heck do you mean, functions without names**?" I hear you ask.<span style="mso-spacerun: yes">  </span>"**Surely all functions have names\!"** -- well, no, actually, some don't.<span style="mso-spacerun: yes">  </span>This is perfectly legal in JScript:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Console&#39;">print ( <span style="BACKGROUND: yellow; mso-highlight: yellow">function(x){return x \* 2;}</span> (3) );</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">That prints out "6".<span style="mso-spacerun: yes">  </span>What's the name of that function?<span style="mso-spacerun: yes">  </span>It has no name.<span style="mso-spacerun: yes">  </span>Of course, we could give it one if we chose.<span style="mso-spacerun: yes">  </span>This is exactly the same as declaring a named function:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; BACKGROUND: yellow; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Console&#39;; mso-highlight: yellow">function double(x){return x \* 2;}</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Console&#39;">print ( double(3) );</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">But functions don't need names any more than strings or numbers do.<span style="mso-spacerun: yes">  </span>Functions are just functions whether they’re named or not.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Remember, JScript is a functional language.<span style="mso-spacerun: yes">  </span>Functions are objects and can be treated like any other object.<span style="mso-spacerun: yes">  </span>You don't have to give an object a name to use it, or you can give them multiple names.<span style="mso-spacerun: yes">  </span>Functions are no different.<span style="mso-spacerun: yes">  </span>For example, you can assign them to variables.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; BACKGROUND: yellow; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Console&#39;; mso-highlight: yellow">var myfunc = function(x){return x \* 2;}</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Console&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Console&#39;">var myfunc2 = myfunc;</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Console&#39;">print ( myfunc(3) );</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Those of you who are familiar with more traditional functional languages, such as Lisp or Scheme, will recognize that functions in JScript are fundamentally the Lambda Calculus in fancy dress. <span style="mso-spacerun: yes"> </span>(The august Waldemar Horwat -- who was at one time the lead Javascript developer at AOL-Time-Warner-Netscape -- once told me that he considered Javascript to be just another syntax for Common Lisp.<span style="mso-spacerun: yes">  </span>I'm pretty sure he was being serious; Waldemar's a hard core language guy and a heck of a square dancer to boot.)<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Anyway, I'll discuss other functional language properties of JScript in a future post.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">One can also construct anonymous functions at runtime with the Function constructor, though why you’d want to is beyond me.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Console&#39;">print ( <span style="BACKGROUND: yellow; mso-highlight: yellow">new Function("x", "return x \* 2;")(</span>3) );</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">I recommend against constructing new functions at runtime based on strings, but that's a subject for a future post.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Interestingly enough, what this does internally is constructs the following script text, and compiles it:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Console&#39;">function anonymous(x){return x \* 2;}</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">So in fact, **this actually does not compile up an anonymous function** -- it compiles up a non-anonymous function named "anonymous".<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">The mind fairly boggles.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Second, there are the anonymous functions constructed when IE builds an event handler. <span style="mso-spacerun: yes"> </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">ASIDE : In the source code for the script engine these things are called "scriptlets", which is a terrible, undescriptive, confusing name.<span style="mso-spacerun: yes">  </span>At one point there were three technologies all competing for the name "scriptlet". <span style="mso-spacerun: yes"> </span>The technology presently known as Windows Script Components was originally called "Scriptlets", which explains the name of the Wrox book "Instant Scriptlets" -- they published based on a beta released before the name was finalized.<span style="mso-spacerun: yes">  </span>We considered calling Windows Script Components "Scriptoids" and "Script Thingies", but fortunately cooler heads prevailed.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">But I digress. When you have a button in IE</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Console&#39;">\<BLAH BLAH BLAH NAME="BUTTON1" ONCLICK="<span style="BACKGROUND: yellow; mso-highlight: yellow">window.alert('Oooh, you clicked me\!');"</span> \></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">then the script engine creates a separate compilation scope for the form and compiles up this string:</span>

<span style="FONT-SIZE: 10pt; BACKGROUND: yellow; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Console&#39;; mso-highlight: yellow"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Console&#39;; mso-bidi-font-family: &#39;Lucida Console&#39;">function anonymous(){<span style="BACKGROUND: yellow; mso-highlight: yellow">window.alert('Oooh, you clicked me\!');</span>}</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">It then passes the function object back as a dispatch pointer and IE assigns it to the button's onclick property.<span style="mso-spacerun: yes">  </span>Again, this is a non-anonymous function named "anonymous".</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

</div>

</div>

