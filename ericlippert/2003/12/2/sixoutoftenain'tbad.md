# Six out of ten ain't bad

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/2/2003 12:58:00 AM

-----

Occasionally I interview C++ developers. I'm always interested in how people rate themselves, so I'll occasionally ask a candidate, "On a scale from one to ten, how do you rate your C++ skills?"

The point of the question is actually not so much to see how good a programmer the candidate is -- I'm going to ask a bunch of coding questions to determine that. Rather, it's sort of a trick question. What I'm actually looking for -- what I'm looking for in almost every question I ask -- is "how does the candidate handle a situation where there is insufficient information available to successfully solve a problem?" Because lemme tell ya, that's what every single day is like here on the Visual Studio team: hard technical problems, insufficient data, deal with it\!

The question has insufficient data to answer it because we have not established what "ten" is and what "one" is, or for that matter, whether the scale is linear or logarithmic. Does "ten" mean "in the 90th percentile" or "five standard deviations from the mean" or what? Is a "one" someone who knows NOTHING about C++? Who's a ten?

Good candidates will clarify the question before they attempt to answer it. Bad candidates will say "oh, I'm a nine, for sure\!" without saying whether they are comparing themselves against their "CS360: Algorithmic Design" classmates or Stanley Lippman.

I mention this for two reasons -- first of all, my favourite question to ask the "I'm a nine out of ten" people actually came up in a real-life conversation today: OK, smartypants: what happens when a virtual base class destructor calls a virtual method overridden in the derived class? And how would you implement those semantics if you were designing the compiler? (Funny how that almost never comes up in conversation, and yet, as today proved, it actually is useful knowledge in real-world situations.)

The second reason is that ten-out-of-ten C++ guru [Stanley Lippmann has started blogging](http://blogs.gotdotnet.com/slippman/). Getting C++ to work in the CLR environment was a major piece of design work, of a difficulty that makes porting JScript to JScript.NET look like a walk in the park on a summer day.

Compared to Stanley Lippmann, I give myself a six.

