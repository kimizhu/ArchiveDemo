<div id="page">

# Why is it a bad idea to put script objects in Session scope?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/18/2003 10:35:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Often a web site will have a series of related pages requested one after the other by the same user.<span style="mso-spacerun: yes">  </span>As a convenience for the site developers, the ASP object model provides a Session object to store server-side state for a current user.<span style="mso-spacerun: yes">  </span>It also has a global "Application" object which stores state for an entire virtual root.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Every so often some ASP coder out there tries to put a JScript or VBScript object into the Session (or Application) object.<span style="mso-spacerun: yes">  </span>Things usually start going horribly wrong shortly thereafter -- either terrible performance ensues or things just break outright.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Well, like Groucho says, if it hurts when you do that, don't do that\!<span style="mso-spacerun: yes">  </span>Understanding why this is a bad idea will take us yet deeper into the land of threading models.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 12pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Marshaling </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">I mentioned earlier that when you have an apartment threaded object, you need to be in the right apartment (thread) if you want to talk to its occupants (object instances).<span style="mso-spacerun: yes">  </span>But what if you are not?<span style="mso-spacerun: yes">  </span>What if you are running some code in thread Alpha that *really* needs to call a method on an object running in thread Beta?<span style="mso-spacerun: yes">  </span>Fortunately, COM provides a mechanism called **<span style="FONT-WEIGHT: bold; mso-bidi-font-weight: normal">cross-thread marshaling</span>** to achieve this.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">The behind-the-scenes details are not particularly important to our discussion; suffice to say that windows messages are involved.<span style="mso-spacerun: yes">  </span>The important thing to know is that when you marshal a call across threads **<span style="FONT-WEIGHT: bold; mso-bidi-font-weight: normal">the calling thread pauses until the called thread responds</span>**.<span style="mso-spacerun: yes">  That seems reasonable -- after all, when you call an ordinary function you sort of "pause" until the function returns.  </span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"></span></span> 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes">But threads are usually busy doing *something*.  I</span>f the called thread is not responding to messages because it is busy doing work of its own then the calling thread waits, and waits, and waits...</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">To continue with our previous apartment threading analogy, it is rather like each apartment has a mailbox.<span style="mso-spacerun: yes">  </span>If you're in apartment Beta and you need someone in apartment Alpha to do something for you, you write up a request and hand it to the mailman who in turn sticks it in Alpha's mailslot.<span style="mso-spacerun: yes">  </span>Alpha's occupants might be busy doing work and ignoring their mail, or they may have a huge stack of mail to get through, or they might get right on it.<span style="mso-spacerun: yes">  </span>You, over in apartment Beta, can't do anything but wait for the mailman to deliver their reply.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">And of course, even if the callee thread is completely responsive and ready to service your request, obviously calling across threads is orders of magnitude more expensive than calling in-thread.<span style="mso-spacerun: yes">  </span>An in-thread call requires some arguments to be put on the stack and maybe stash a few values in registers.<span style="mso-spacerun: yes">  </span>A cross-thread call gets the operating system involved in a comparatively deep and complex way.</span>

 

<span style="FONT-SIZE: 12pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Performance Woes</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Now you have enough information to figure out why putting script objects in Session scope is a bad idea as far as performance is concerned.<span style="mso-spacerun: yes">  </span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"></span></span><span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"></span></span> 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"></span>Each ASP page is running on its own thread from the thread pool.<span style="mso-spacerun: yes">  </span>The thread that reads the object from session state will likely not be the thread that put the object there, but the script object is apartment threaded.<span style="mso-spacerun: yes">  </span>That means that essentially any page that accesses that session object must wait its turn for all other pages using that session object to finish up, because **the marshaling code blocks the calling thread until the callee is available.**<span style="mso-spacerun: yes">** ** </span>You end up with a whole lot of blocking threads, and blocking threads are not fast.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Application scope is even worse -- if you put a script object in Application scope then **every page in the entire vroot that accesses the Application object must wait its turn** for the original thread to be free.<span style="mso-spacerun: yes">  </span>You've effectively single-threaded your web server.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 12pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Script Engine Of The Living Dead</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">But it gets worse.<span style="mso-spacerun: yes">  </span>Remember, when the page is done being served up, the engine is torn down.<span style="mso-spacerun: yes">  </span>The compiled state remains, but the runtime state is thrown away.<span style="mso-spacerun: yes">  </span>So suppose you have a JScript object sitting in the Session object, and the page that put it there was destroyed a few microseconds ago and the engine put back into the engine pool.<span style="mso-spacerun: yes">  </span>Now on another page in the same session you try to fetch the JScript object and call a method on it.<span style="mso-spacerun: yes">  </span>Where's the runtime state associated with that object?<span style="mso-spacerun: yes">  </span>It's gone, dude.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">When the script engine is torn down after the original page is served, the teardown code detects that there are existing objects that are owned by external callers.<span style="mso-spacerun: yes">  </span>The script engine can't destroy these objects, otherwise the caller would crash when the caller tried to destroy them later.<span style="mso-spacerun: yes">  </span>That's a fundamental rule of COM -- you can't destroy an object to which someone holds a reference.<span style="mso-spacerun: yes">  </span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"></span></span> 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"></span>But we know that the object is going to be useless, so what we do is tell the object "the engine you require for life is going away.<span style="mso-spacerun: yes">  </span>Throw away everything you own and become a zombie."<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">These zombie objects look like script objects, but when you actually try to do something to them -- call a method, fetch a property, whatever -- they don't actually do anything.<span style="mso-spacerun: yes">  </span>They can't -- all the infrastructure they need to live is gone, but they can't die.<span style="mso-spacerun: yes">  </span>Basically they wander the night in ghostly torment until they are freed by whatever code is holding the reference.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">So not only are those script objects sitting in Session state wrecking your performance, they're not even *useful* for anything once the original page goes away.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Incidentally, Windows Script Components each have their own engine which stays alive as long as they do, so WSC's are not affected by the zombie issue.<span style="mso-spacerun: yes">  </span>They are still apartment threaded though.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 12pt; COLOR: navy; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">Arrays Are Almost As Bad</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">JScript arrays are objects, so everything said above applies to them.<span style="mso-spacerun: yes">  </span>VBScript arrays are not objects (more on the differences between these two kinds of arrays later) but even still, you shouldn't put them in Session scope either.<span style="mso-spacerun: yes">  </span>Though they do not suffer from the threading problems or the lifetime problems mentioned above, arrays are stored into and passed out of Session scope using our old friend copy-in-copy-out semantics.<span style="mso-spacerun: yes">  </span>That means that **<span style="FONT-WEIGHT: bold; mso-bidi-font-weight: normal">every single time </span>**you index into an array in Session scope, **<span style="FONT-WEIGHT: bold; mso-bidi-font-weight: normal">a copy of the array is made first</span>**.<span style="mso-spacerun: yes">  </span>If that array is big, that's a big performance problem.<span style="mso-spacerun: yes">  </span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"></span></span> 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"><span style="mso-spacerun: yes"></span>Why do we do copy-in-copy-out?<span style="mso-spacerun: yes">  </span>*<span style="FONT-STYLE: italic; mso-bidi-font-style: normal">Because</span>* arrays are not marshaled\!<span style="mso-spacerun: yes">  </span>We return to the fundamental problem: what if two pages tried to read and write the array in the Session object at the same time?<span style="mso-spacerun: yes">  </span>The memory could be corrupted.<span style="mso-spacerun: yes">  </span>We really don't have any good way to synchronize access to the array, so instead we simply make a complete copy of it every time someone tries to read or write it.<span style="mso-spacerun: yes">  </span>This is extremely expensive, but it keeps the server heap from being corrupted.<span style="mso-spacerun: yes">  </span>A corrupted web server heap can ruin your whole day.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;">My advice is to not even go there.<span style="mso-spacerun: yes">  </span>**Don’t put information into Session scope unless you absolutely have to.**<span style="mso-spacerun: yes">** ** </span>If you must, put in strings.<span style="mso-spacerun: yes">  </span>There are lots of ways to store arrays or objects as strings and reconstitute them as needed.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; mso-bidi-font-family: &#39;Times New Roman&#39;"> </span>

 

</div>

</div>

