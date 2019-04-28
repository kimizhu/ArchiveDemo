# I have a Fit, but a lack of Focus.

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/29/2009 6:54:00 AM

-----

Here's a statement I read the other day about making comparisons between objects of reference type in C\#:

> Object.ReferenceEquals(x,y) returns true if and only if x and y refer to the same object.

True or false?

My wife Leah recently acquired a Honda Fit, thanks to the imminant failure of the automatic transmission solenoids in her aged Honda Civic. The back seats in the Fit fold down flat. You can fit a llama or a whole pile of hula hoops or whatever into that thing. It's quite handy. Not what I would call a powerful engine by any means, but for quick trips around town, it certainly gets the job done.

Since we were married when she bought the car, and we continue to be married, what's mine is hers and what's hers is mine. So if x = Eric's Honda Fit, and y = Leah's Honda Fit, then x and y are "reference equals". Those two things refer to the same object, viz, the shiny black object full of llamas and hula hoops in my driveway.

Now, we could have bought a different car. Say, a Ford Focus. But we did not. We own a total of zero Ford Foci. Suppose I said that x = Eric's Ford Focus, and y = Leah's Ford Focus. What's the sensible way to characterize the nature of x and y? Do we say that x and y refer to the same Ford Focus, namely that they refer to the Ford Focus that does not exist? The mind boggles at the repugnant and paradoxical implication that there exists a Ford Focus that is the Ford Focus that does not exist\! (\*) Rather, the right way to characterize this is to say that neither x nor y refer to any object. They're "null references" -- references that do not have any referent, but rather, capture the notion of "a lack of referent".

And that's why it's incorrect to say that Object.ReferenceEquals(x,y) returns true **if and only if** x and y refer to the **same object**.If x and y both do not refer to any object, then clearly they do not *refer to* *the same object*, because *neither refers to an object in the first place*. The correct way to characterize the behaviour of reference equality is

> Object.ReferenceEquals(x,y) returns true if and only if either x and y refer to the same object, *or x and y are both null references.*

\*\*\*\*\*\*\*\*\*\*\*

(\*) And yet I am a fan of the "null object pattern". Life is just full of these little contradictions.

\[Eric is on vacation this week; this posting is pre-recorded.\]

