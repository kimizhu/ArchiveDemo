<div id="page">

# It Never Leaks But It Pours

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/13/2003 2:16:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">One of the easiest bugs to write is the dreaded **memory leak**.<span style="mso-spacerun: yes">  </span>You allocate some chunk of memory and never release it.<span style="mso-spacerun: yes">  </span>Those of us who grew up writing application software might sometimes have a cavalier attitude towards memory leaks -- after all, the memory is going to be reclaimed when the process heap is destroyed, right?<span style="mso-spacerun: yes">  </span>But those days are long gone.<span style="mso-spacerun: yes">  </span>Applications now often run for days, weeks or months on end.<span style="mso-spacerun: yes">  </span>Any memory that is leaked will slowly consume all the memory in the system until it dies horribly.<span style="mso-spacerun: yes">  </span>Web servers in particular are highly susceptible to leaks, as they run forever and move a LOT of memory around with each page request.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The whole point of developing a garbage collected language is to decrease the burden on the developer.<span style="mso-spacerun: yes">  </span>Because the underlying infrastructure manages memory for you, you don't have to worry about introducing leaks.<span style="mso-spacerun: yes">  </span>Of course, that puts the burden squarely upon the developer of the underlying infrastructure, ie, me.<span style="mso-spacerun: yes">  </span>As you might imagine, I've been called in to debug a LOT of memory leaks over the years.<span style="mso-spacerun: yes">  </span>The majority turned out to be in poorly written third-party components, but a few turned out to be in the script engines.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">I mentioned a while back that ASP uses a technique called "thread pooling" to increase its performance.<span style="mso-spacerun: yes">  </span>The idea is that you maintain a pool of idle threads, and when you need work done, you grab one from the pool.<span style="mso-spacerun: yes">  </span>This saves on the expense of creating a new thread and destroying it when you're done with it.<span style="mso-spacerun: yes">  </span>On a web server where there may be millions of page requests, the expense of creating a few million threads is non-trivial.<span style="mso-spacerun: yes">  </span>(Also, this ensures that you can keep a lid on the number of requests handled by one server -- if the server starts getting overloaded, just stop handing out threads to service requests.)</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">I think I also mentioned a while back that JScript has a per-thread garbage collector.<span style="mso-spacerun: yes">  </span>That is, if you create two engines on the same thread, they actually share a garbage collector.<span style="mso-spacerun: yes">  </span>When one of those engines runs a GC, effectively they all get collected.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">What do these things have to do with each other?<span style="mso-spacerun: yes">  </span>Well, as it turns out, there *is* a memory leak that we have just discovered in the JScript engine.<span style="mso-spacerun: yes">  </span>A small data structure associated **with the garbage collector** is never freed when the thread goes away.<span style="mso-spacerun: yes">  </span>What incredible irony\!<span style="mso-spacerun: yes">  </span>The very tool we designed to prevent your memory leaks is leaking memory.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">It gets worse.<span style="mso-spacerun: yes">  </span>As it turns out, this leak has been in the product for *years*.<span style="mso-spacerun: yes">  </span>Why did we never notice it?<span style="mso-spacerun: yes">  </span>Because it is a **per-thread leak**, and ASP uses thread pooling\!<span style="mso-spacerun: yes">  </span>Sure, the memory leaks, but only once per thread, and ASP creates a small number of threads, so they never noticed the leak.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">So why am I telling you this?<span style="mso-spacerun: yes">  </span>Because for some reason, it never rains but it pours.<span style="mso-spacerun: yes">  </span>We are suddenly getting a considerable number of people reporting this leak to us.<span style="mso-spacerun: yes">  </span>Apparently, all of a sudden there are third parties developing massively multi-threaded applications that continually create and destroy threads with script engines. Bizarre, but true. They are all leaking a few dozen bytes per thread, and a few hundred thousand threads in, that adds up.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">I have no idea what has led to this sudden uptick in multithreaded script hosts.<span style="mso-spacerun: yes">  </span>But if you're writing one, let me tell you two things:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-fareast-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-list: Ignore">1)<span style="FONT: 7pt &#39;Times New Roman&#39;">      </span></span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">We're aware of the problem.<span style="mso-spacerun: yes">  </span>Top minds on the Sustaining Engineering Team are looking at it, and I hope that we can have a patched version for a future service release.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-fareast-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-list: Ignore">2)<span style="FONT: 7pt &#39;Times New Roman&#39;">      </span></span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Use thread pooling\!<span style="mso-spacerun: yes">  </span>Not only will it effectively eliminate this leak, it will make your lives easier in the long run, believe me.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">This whole thing reminds me that I want to spend some time discussing some of the pitfalls we've discovered in performance tuning multi-threaded applications.<span style="mso-spacerun: yes">  </span>But that will have to wait for another entry.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

 

</div>

</div>

