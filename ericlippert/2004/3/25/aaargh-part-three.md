<div id="page">

# Aaargh\! Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/25/2004 2:45:00 PM

-----

<div id="content">

<div id="idOWAReplyText18632" dir="ltr">

<div dir="ltr">

I'm still at VSLive.  Both my talks are done, so its just booth duty from here on in.  The talks went... OK. Running VSTO on top of Virtual PC on a laptop was too slow; we'll have to devirtualize that for the next time.   

Unfortunately they put me in the keynote room, which seats over a thousand.  Now, when 200 people show up for a talk, that's great -- but in such an enormous room, it's really, really hard to get the audience enthusiam up when they're spread out over what seemed like a couple of acres.  But I got a lot of good questions afterwards, so hopefully a few people were awake and learned something.

No time to write, so another canned gripe about C++ idioms that are drivin' me nuts today -- but first, an excerpt from [Pirate Riddles For Sophisticates](http://www.mcsweeneys.net/2000/06/14pirates.html%20). 

Q: Whom did the pirate vote for in the first Haitian election?  
A: ARRRistide.

Q: Wait. Why did they let a pirate vote in the Haitian election?  
A: Remember, the nation was taking its first halting steps toward democracy, and balloting procedures were rather chaotic. The pirate just slipped in somehow. Arrr.

</div>

<div>

<span lang="en-us">Gripe \#5: Exception Handling and operator new</span>

<span lang="en-us">In C++ you can easily change the</span><span lang="en-us"></span><span lang="en-us"> new</span><span lang="en-us"></span><span lang="en-us"> operator to throw an exception when it runs out of memory instead of returning</span><span lang="en-us"></span><span lang="en-us"> NULL</span><span lang="en-us"></span><span lang="en-us">.  Cool idea, but it can be very dangerous because this causes a global change in the behaviour of a commonly used operator.  What happens when you do that in a program that contains libraries that do not have that assumption?</span>

<span lang="en-us">try  
</span><span lang="en-us">{  
</span><span lang="en-us">  m\_pbar = m\_pfoo-\>bar();  
</span><span lang="en-us">} catch //</span><span lang="en-us"></span><span lang="en-us"> ...</span><span lang="en-us"></span><span lang="en-us"></span>

<span lang="en-us">where you grabbed the source code for that class and just kind of compiled it in:</span>

<span lang="en-us">Bar \* CFoo::bar(void)  
</span><span lang="en-us">{  
</span><span lang="en-us">  HANDLE h = NULL;  
</span><span lang="en-us">  h = GoGetMeAFileHandle();  
</span><span lang="en-us">  m\_pBlah = new CBlah;  
</span><span lang="en-us">  if (NULL == m\_pBlah){  
</span><span lang="en-us">    FreeFileHandle(h);  
</span><span lang="en-us">    return NULL;  
</span><span lang="en-us">  }</span><span lang="en-us"> // ...</span>

<span lang="en-us">You didn't write</span><span lang="en-us"></span><span lang="en-us"> CFoo::bar</span><span lang="en-us"></span><span lang="en-us">, but you've just broken its error handling if you redefined</span><span lang="en-us"></span><span lang="en-us"> new</span><span lang="en-us"></span><span lang="en-us"> to throw an error instead of returning</span><span lang="en-us"></span><span lang="en-us"> NULL</span><span lang="en-us"></span><span lang="en-us">.  Now if the allocation of</span><span lang="en-us"></span><span lang="en-us"> CBlah</span><span lang="en-us"></span><span lang="en-us"> fails, you've leaked a kernel object. This is another example where idioms like smart pointers need to be **consistently applied across an application** in order to reap their benefits.</span>

<span lang="en-us">I have debugged leaks in IIS where these kinds of things happen and they are absolutely no fun for our customers.</span>

<span lang="en-us">I once debugged some code where the</span><span lang="en-us"></span><span lang="en-us"> new</span><span lang="en-us"></span><span lang="en-us"> operator had been redefined to throw an exception, and the authors did something very clever.  (Aside: clever is bad -- clever is hard to figure out\!  If the code is clever, rewrite it until it is brain-dead obvious.)  In the debug build, they wrote some magic into the operator so that it detected if it was being called without an exception handler and raised an assertion.  Sounds cool, right?  Any possible negative consequences of that?</span>

<span lang="en-us">Well, how about the fact that static objects which have constructors that call</span><span lang="en-us"></span><span lang="en-us"> new</span><span lang="en-us"></span><span lang="en-us"> cannot be linked in to the application without the program asserting on startup?  The standard</span><span lang="en-us"></span><span lang="en-us"> map</span><span lang="en-us"></span><span lang="en-us">,</span><span lang="en-us"></span><span lang="en-us"> vector</span><span lang="en-us"></span><span lang="en-us">, etc, objects are such objects.  Their constructors call</span><span lang="en-us"></span><span lang="en-us"> new</span><span lang="en-us"></span><span lang="en-us">, and therefore you can't have a static</span><span lang="en-us"></span><span lang="en-us"> map</span><span lang="en-us"></span><span lang="en-us">. Statics have no context to</span><span lang="en-us"></span><span lang="en-us"> catch</span><span lang="en-us"></span><span lang="en-us"> the possible</span><span lang="en-us"></span><span lang="en-us"> throw</span><span lang="en-us"></span><span lang="en-us">, and therefore will assert when the debug application starts up.  (Guess how I found that bug?)</span>

<span lang="en-us">That said, I can see how it is useful to have a</span><span lang="en-us"></span><span lang="en-us"> new</span><span lang="en-us"></span><span lang="en-us"> operator that throws an exception.  You know what I'd do if I wanted that?  I'd define an operator overload that took an argument, create a dummy identifier called "throws", and then say</span><span lang="en-us"></span><span lang="en-us"> pBar = new(throws) CBar</span><span lang="en-us"></span><span lang="en-us">  -- now it is perfectly clear what the semantics of that thing are.</span>

<span lang="en-us">Redefining the global</span><span lang="en-us"></span><span lang="en-us"> new</span><span lang="en-us"></span><span lang="en-us"> operator works great if you have 100% control over all contexts where the operator is used.  Unfortunately, that's frequently not a realistic assumption\!  </span>

</div>

</div>

</div>

</div>

