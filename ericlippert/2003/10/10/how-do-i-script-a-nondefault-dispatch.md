<div id="page">

# How Do I Script A Non-Default Dispatch?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/10/2003 2:43:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">As I've discussed previously, the script engines always talk to objects on the late-bound </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">IDispatch</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> interface.<span style="mso-spacerun: yes">  </span>The point of this interface is to allow a script language to call a method or reference a property of an object by giving the name of the field and the arguments.<span style="mso-spacerun: yes">  </span>When the dispatch object is invoked, the object does the work of figuring out which method to call on which interface.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">But what if an object supports two interfaces </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">IFoo</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> and </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">IBar</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> both of which you want to be able to call late-bound?<span style="mso-spacerun: yes">  </span>The most common way to solve this problem is to create two late-bound interfaces, </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">IFooDisp</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> and </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">IBarDisp</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">.<span style="mso-spacerun: yes">  </span>So when you ask the object for </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">IFooDisp</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">, you get an interface that can do late-bound invocation on </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">IFoo</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">, and similarly for </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">IBarDisp</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">What happens when you ask the object for an </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">IDispatch</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> interface?<span style="mso-spacerun: yes">  </span>It's got to pick one of them\!<span style="mso-spacerun: yes">  </span>The one it picks is called the "default dispatch".</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">In JScript and VBScript, when you create an object (via </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">new ActiveXObject</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> in JScript or </span><span style="FONT-SIZE: 10pt; COLOR: #333399; FONT-FAMILY: &#39;Lucida Console&#39;">CreateObject</span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> in VBScript) the creation code always returns the default dispatch.<span style="mso-spacerun: yes">  </span>Furthermore, in JScript, when you fetch a property on an object and it returns a dispatch object, we ask the object to give us the default dispatch.<span style="mso-spacerun: yes">  </span>So in JScript, there is no way to script a non-default dispatch.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">What about in VBScript?<span style="mso-spacerun: yes">  </span>There's an irksome story here, again featuring a really bad mistake made by yours truly. <span style="mso-spacerun: yes"> </span>At least I *meant* well.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">I didn't write the variant import code, and I had always assumed that VBScript did the same thing as JScript -- when an object enters the script engine from outside, we query it for its default dispatch.<span style="mso-spacerun: yes">  </span>As it turns out, that's not true.<span style="mso-spacerun: yes">  </span>In VBScript, for whatever reason, we just pass imported dispatch objects right through and call them on whatever dispatch interface the callee gives us.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">One day long ago we found a security hole in IE.<span style="mso-spacerun: yes">  </span>The details of the hole are not important -- but what's interesting about it was the number of things that had to go wrong to make the hole an actual vulnerability.<span style="mso-spacerun: yes">  </span>Basically the problem was that one of the built-in IE objects had two dispatch interfaces, one designed to be used from script and one for internal purposes.<span style="mso-spacerun: yes">  </span>The for-script dispatch interface was the default, and it was designed to participate in the IE security model.<span style="mso-spacerun: yes">  </span>The internal-only interface did not enforce the IE security model, and in fact, there was a way to use the object to extract the contents of a local disk file and send it out to the internet, which is obviously badness. Furthermore, there was a way to make *another* object return the non-default interface of the broken object.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">JScript did not expose this vulnerability because it always uses the default dispatch even when given a non-default dispatch.<span style="mso-spacerun: yes">  </span>But the vulnerability *was* exposed by VBScript.<span style="mso-spacerun: yes">  </span>All these flaws had to work together to produce one vulnerability.<span style="mso-spacerun: yes">  </span>Now, when you find a security hole that consists of multiple small flaws, the way you fix it is **not** to just **patch one thing and hope for the best**.<span style="mso-spacerun: yes">  </span>You **patch everything you possibly can**.<span style="mso-spacerun: yes">  </span>Remember, secure software has **defense in depth**.<span style="mso-spacerun: yes">  </span>Make the attackers have to do *twelve* impossible things, not *one* impossible thing, because sometimes you're wrong about what's impossible.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Hence, when we fixed this hole, we fixed *everything*.<span style="mso-spacerun: yes">  </span>We made sure that the object's persistence code could no longer read files off the local disk.<span style="mso-spacerun: yes">  </span>In case there was still a way to make it read the disk that the patch missed, we fixed the object model so that it never returned a non-default interface to a script.<span style="mso-spacerun: yes">  </span>In case there was still a way to do both those things that we missed, **we also turned off VBScript's ability to use non-default dispatches.**</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">That last one turned out to be a huge mistake.<span style="mso-spacerun: yes">  </span>The fact that **I had believed that VBScript always talked to the default dispatch** does not logically imply that every **VBScript user read my mind and knew that successfully using a non-default dispatch was some sort of fluke**.<span style="mso-spacerun: yes">  </span>People naturally assume that if they write a program and it works, then it works because the language designers wanted them to be able to write that program\!</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">As it turned out, there were *plenty* of programs out there in the wild that used VBScript to script non-default dispatch interfaces returned by method calls, and I broke *all* of them through trying to make IE safer.<span style="mso-spacerun: yes">  </span>We ended up shipping out a new build of VBScript with the default dispatch feature turned back on a couple of days later.<span style="mso-spacerun: yes">  </span>(Of course all the fixes to IE were sufficient to mitigate the vulnerability on their own, and those were not changed back.)</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The morals of the story are:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-fareast-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-list: Ignore">1)<span style="FONT: 7pt &#39;Times New Roman&#39;">      </span></span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">If you want to use a non-default dispatch, you have to use VBScript.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-fareast-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-list: Ignore">2)<span style="FONT: 7pt &#39;Times New Roman&#39;">      </span></span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">There is no way to make VBScript give you a specific non-default dispatch.<span style="mso-spacerun: yes">  </span>In VBScript, once you have a dispatch, that's the one you're stuck with.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-fareast-font-family: &#39;Lucida Sans Unicode&#39;"><span style="mso-list: Ignore">3)<span style="FONT: 7pt &#39;Times New Roman&#39;">      </span></span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Defense in depth is a good idea, but it's not such a good idea to go so deep that backwards compatibility is broken if you can avoid it\!</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

</div>

</div>

