# Marshal-by-ref versus Serializable Objects

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/27/2004 11:09:00 AM

-----

(There's been a sudden influx in blog readers asking me good questions, which is great.  Be patient; I'll try to cover them over the next few entries.) 

In response to yesterday's entry on serializable JScript .NET objects, a reader asked 

> Please forgive my cargo-cultism: What is the difference between Marshal By Reference and Serialization?

First off, cargo cultists never stop to ask themselves "hey, how exactly DOES an airplane work?"  That's what makes them cargo cultists: they don't ask questions about what they don't understand, they just forge ahead. 

Let me give you an analogy. I want to talk to you.  I'm in Seattle, you're in Hong Kong, and neither of us want to move.  There is a barrier between us, namely the Pacific Ocean.  

There are two "obvious" ways to solve this problem. 

1\) Build a telephone system between Seattle and Hong Kong.  I get a telephone receiver with "CLIENT PROXY" written on it. You get a telephone receiver with "SERVER STUB" written on it.  Instead of talking to you, I talk into Proxy.  Proxy talks to Stub somehow -- I really don't care how the phone system works, so long as it does -- Stub talks to you.  We get the *illusion* that we're actually talking to each other, when we're actually talking to hunks of plastic, but the information content is the same, so who cares?  Maybe there is some delay and expense, but the proxy does a good enough job of sending and receiving messages that we can communicate across the barrier. 

2) Sequence your DNA into a string.  Run your brain through a Molecular Neuron Defrobnicator that extracts all your memories and saves them to disk.  Put the DNA string and memory data onto CD-ROMs, and FedEx the box of CD-ROMs to Seattle.  Once I get them in Seattle, I rebuild your DNA from the sequence information using nanorobots. I inject the rebuilt DNA into an egg cell.  We use the egg cell to grow a copy of you in the lab.  When the brain is developed enough, I use my Molecular Neuron Refrobnicator to insert your memories into the clone's brain.   Who needs the phone?  I can talk to you in person\! And now there are two of you running around, one in Hong Kong, one in Seattle, so you can get your work done in Hong Kong without worrying about waiting by the phone all the time. OK, maybe the second isn't *quite* as obvious as the first, but in principle it would work.  That it happens to be in real life cheaper to build telephone systems than Molecular Neuron Frobnicators is irrelevant; in the world of .NET objects, they are about equally expensive. 

The first option is marshal by ref -- the object is marshaled by creating a proxy/stub pair that knows how to talk across whatever barrier it is you're moving the object.  There is enormous, expensive machinery behind the scenes that moves the information around on your behalf, but you don't have to understand it, you just have to pay the performance penalty of using it. 

The second is serialization.  A serializable object knows how to dump its state (its memories) into a byte array. We move the byte array across the boundary, which is easily done -- it's just bytes.  We move the name of the type of the object (it's DNA) across the barrier as a string as well. On the destination side we can create an instance of that type and then dump the original state from the byte array into the new object. Now there are two identical objects, one on each side of the boundary. 

Clearly there are pros and cons of both approaches.  The telephone system between Seattle and Hong Kong was NOT cheap to build and is not cheap to use.  If multiple people are trying to talk to you on multiple phones at the same time, sorting out all the conversations can be difficult.  You might have to put people on hold for a while, which isn't cheap either.  But if you really need access to an individual, specific object that exists on the other side of a barrier, that's the way to go. 

You don't always need to talk to the original object; sometimes you want to get your own copy locally and talk to that thing.  Web services, for example, often serialize objects and send them across a wire to be reconstituted on the client side.  You *do not* want to talk to the original object back on the server; the server might have a million people to serve.  Or consider what happens to an exception object thrown across an appdomain boundary.  Does it really matter whether you can talk to the original object?  No -- all you need to do is extract the information from it, so it doesn't matter if you have a copy. 

I'm no big expert on .NET Remoting though, and this just touches the surface of this fascinating subject.  I've recently acquired Ingo Rammer's book, which looks quite fine but I haven't had a chance to sit down and read through it yet. 

**UPDATE:** 

Mike Dimmick pointed out something that I should have noted.  What I'm describing here is not really “Marshal By Ref vs Serialization”.  I'm describing “Marshal By Ref vs Marshal By Value”.  Serialization is how we *implement* Marshal By Value.  

This is an important distinction because serialization is useful for more than just marshaling an object by value across a boundary.  For example, it is also useful for **persistence**.  If you have an object in memory and you'd like to save it to disk, then being able to serialize that thing into a byte array is darn handy.  

In keeping with our ridiculous analogy, you save your memories and DNA to disk, but rather than shipping the disks to Seattle, you put the box of disks in a closet and vaporize yourself (with a Molecular Vapor-O-Matic).  When someone wants to talk to you again, they get the box of disks out of the closet and reconstitute you as in the MBV scenario.  This time the barrier the object is crossing is the time barrier of it's own death\!  

I saw a movie about that once, starring the governor of California.  Funny how life turns out, eh?

In other news, I recall that a while back the Wordzguy wrote a [blog entry](http://www.livejournal.com/users/wordzguy/107812.html)about various slang terms for serialize/deserialize.  Dehydrate/rehydrate is fairly common.  I offered up freeze-dry/rehydrate, and another reader pointed out that those wacky Python programmers are fond of pickle/unpickle.

