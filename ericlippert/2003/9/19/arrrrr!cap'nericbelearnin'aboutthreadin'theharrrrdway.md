# Arrrrr\! Cap'n Eric be learnin' about threadin' the harrrrd way

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/19/2003 1:00:00 PM

-----

Avast ye scurvy dogs, it be National Talk Like A Pirate Day\!

A scurvy bilge rat commented on the preceding discussion about putting apartment threaded objects in Session scope:

back in the era of the NT4 Option Pack I wrote a lot of code that involved stashing Scripting.Dictionary objects in both session and application scope. \[...\] I forget now which version of the runtime changed the threading model they were registered with and broke everything for me.

Shiver me timbers\! That be my fault. Sorry about that. When you create an ActiveX object, the COM runtime code checks the registry to see if the object is marked as participating in the Apartment, Free or Both threading models.  (We'll go into the difference between Free and Both at another time.)

Now, when I was a young swabbie seven years ago I was given the task of implementing the Scripting.Dictionary object, and I didn't yet understand [all the stuff I just told you maties about threading](http://blogs.msdn.com/b/ericlippert/archive/2003/09/18/why-is-it-a-bad-idea-to-put-script-objects-in-session-scope.aspx).  In one build that was released to the public I accidentally marked the dictionary as Both, even though it is a Single Threaded Apartment object. So when lubbers would put a dictionary into Session scope, it would be called by multiple threads at multiple times, in violation of the apartment contract.  As long as there were only readers, it was generally OK, but as soon as there were readers and writers, it would usually crash and die.

And of course when we corrected the mistake, all those pages went from sometimes-crashing-but-fast to not-crashing-but-slow. That was my first majorly customer-impacting mistake, and probably the worst I ever personally made. 

Speaking of mistakes, there was another interesting performance mistake in early releases of the Scripting.Dictionary object. It uses a very simple hash table for rapid lookup, but of course hash tables require that the hash function distribute hashes **broadly** given a **narrow** distribution of keys.  I screwed up the hash algorithm, and one of the consequences was that hashing on a string consisting of five digits was likely to go to a very small number of hash buckets.  

We discovered all this the day that msn.com decided to store **every zip code in the United States** in a Scripting.Dictionary object in Session scope\!  Perf of msn.com went way south, way fast.

The combination of the two mistakes above led the ASP team to write their own string table object, that really was Both threaded and blindingly fast. Arr\!

