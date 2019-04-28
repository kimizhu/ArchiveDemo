# Writing Code Isn't Rocket Science (It's Worse Than That)

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/10/2006 10:30:00 AM

-----

Today, an old joke:

Q: What do rocket scientists say when they want to describe a portion of their work as easy?  
A: "This bit isn't exactly brain surgery."

I think that pretty much everyone would agree that rocket science and brain surgery are both intellectually demanding pursuits. But it seems to me that there's a fundamental qualitative difference between them. 

Rockets are devices constructed by humans for specific purposes. Though there may be considerable systemic interactions between all the parts of a rocket which must work harmoniously together, those interactions are the result of careful design. The rocket as a whole can be decomposed into its original parts, and those parts can be independently tested to see if they meet their design criteria.

Furthermore, the aim of rocket science is usually to deliver a specific payload to a specific place at a specific time.  Though it is no mean feat to get to the moon, we can least for all practical purposes calculate the future position of the moon at any time you care to name to any degree of precision.

And yes, it's hard, and yes, mistakes are made, sometimes with tragic consequences.  But fundamentally, rocket science is about shaping raw matter into precise forms to achieve precise tasks.

Brain surgery isn't nearly so much like that.  We're presented with a lump of thinking, dreaming meat that we barely understand how it works and is presently being used to solve all manner of problems that evolution did not design it for. Our understanding of brains is strongest at the very low level – the neurochemical level – and at the very high level – the gross divisions of the brain into the centers that control particular muscles, responses, etc. But we have practically no understanding whatsoever of any level between those – we are no where close to understanding what algorithms the brain uses to recognize faces or write music.  And a working, living brain has trillions of interacting parts which were not designed with orthogonality or functional decomposition in mind. 

I've been thinking about the difference between brain surgery and rocket science lately in the context of my coworker [Peter Hallam's essay on the difference between writing new code and modifying old code.](http://blogs.msdn.com/peterhal/archive/2006/01/04/509302.aspx)

Naively, one would think that adding new features to code would be like rocket science.  You understand all the parts, you figure out how to redesign the parts to admit the new desired behaviour, you implement it, test it, and you're done.  It's an intellectual challenge, but it's fundamentally amenable to analysis because every part has a clear, well-designed function that can be tested independently.

But as Peter points out, anyone who has actually tried to add major new functionality to existing code knows that most of the time its more like brain surgery.  We know what the code does on a gross level.  (Say, tokenize source, parse, create symbol tables, bind type annotations, check reachability, generate code.)  We know what any part of it does on the microscopic level. (Say, move the contents of this register to that memory location.)  The hard part is understanding what the algorithms are that make up the large-scale behaviour, and understanding how to tweak them to admit the new desired behaviour without breaking anything.  (How does tweaking early nullable realization during binding affect rewriting lambdas into expression trees?  Who knows?  Not me, that's for sure\!)  Understanding all that stuff is the incredibly hard part; actually putting the code under the knife is comparatively trivial.

A big part of good implementation of software-in-the-large is making sure that the program is more like a rocket than a brain.  Coming up with tools to enable that admirable goal is the hard part.

