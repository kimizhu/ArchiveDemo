# Aaargh\! Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/25/2004 2:45:00 PM

-----

I'm still at VSLive.  Both my talks are done, so its just booth duty from here on in.  The talks went... OK. Running VSTO on top of Virtual PC on a laptop was too slow; we'll have to devirtualize that for the next time.   

Unfortunately they put me in the keynote room, which seats over a thousand.  Now, when 200 people show up for a talk, that's great -- but in such an enormous room, it's really, really hard to get the audience enthusiam up when they're spread out over what seemed like a couple of acres.  But I got a lot of good questions afterwards, so hopefully a few people were awake and learned something.

No time to write, so another canned gripe about C++ idioms that are drivin' me nuts today -- but first, an excerpt from [Pirate Riddles For Sophisticates](http://www.mcsweeneys.net/2000/06/14pirates.html%20). 

Q: Whom did the pirate vote for in the first Haitian election?  
A: ARRRistide.

Q: Wait. Why did they let a pirate vote in the Haitian election?  
A: Remember, the nation was taking its first halting steps toward democracy, and balloting procedures were rather chaotic. The pirate just slipped in somehow. Arrr.

Gripe \#5: Exception Handling and operator new

In C++ you can easily change the new operator to throw an exception when it runs out of memory instead of returning NULL.  Cool idea, but it can be very dangerous because this causes a global change in the behaviour of a commonly used operator.  What happens when you do that in a program that contains libraries that do not have that assumption?

try  
{  
  m\_pbar = m\_pfoo-\>bar();  
} catch // ...

where you grabbed the source code for that class and just kind of compiled it in:

Bar \* CFoo::bar(void)  
{  
  HANDLE h = NULL;  
  h = GoGetMeAFileHandle();  
  m\_pBlah = new CBlah;  
  if (NULL == m\_pBlah){  
    FreeFileHandle(h);  
    return NULL;  
  } // ...

You didn't write CFoo::bar, but you've just broken its error handling if you redefined new to throw an error instead of returning NULL.  Now if the allocation of CBlah fails, you've leaked a kernel object. This is another example where idioms like smart pointers need to be **consistently applied across an application** in order to reap their benefits.

I have debugged leaks in IIS where these kinds of things happen and they are absolutely no fun for our customers.

I once debugged some code where the new operator had been redefined to throw an exception, and the authors did something very clever.  (Aside: clever is bad -- clever is hard to figure out\!  If the code is clever, rewrite it until it is brain-dead obvious.)  In the debug build, they wrote some magic into the operator so that it detected if it was being called without an exception handler and raised an assertion.  Sounds cool, right?  Any possible negative consequences of that?

Well, how about the fact that static objects which have constructors that call new cannot be linked in to the application without the program asserting on startup?  The standard map, vector, etc, objects are such objects.  Their constructors call new, and therefore you can't have a static map. Statics have no context to catch the possible throw, and therefore will assert when the debug application starts up.  (Guess how I found that bug?)

That said, I can see how it is useful to have a new operator that throws an exception.  You know what I'd do if I wanted that?  I'd define an operator overload that took an argument, create a dummy identifier called "throws", and then say pBar = new(throws) CBar  -- now it is perfectly clear what the semantics of that thing are.

Redefining the global new operator works great if you have 100% control over all contexts where the operator is used.  Unfortunately, that's frequently not a realistic assumption\!

