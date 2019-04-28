# Do Not Call IsBadFooPtr, Indeed

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/27/2006 2:16:00 PM

-----

Here’s a story that I said [a long time ago that I was going to tell you all](http://blogs.msdn.com/ericlippert/archive/2004/03/31/105329.aspx), and then promptly forgot about it. [Raymond Chen’s blog entry today](http://blogs.msdn.com/oldnewthing/archive/2006/09/27/773741.aspx) reminded me of it, because this is the story of how I found out the hard way that IsBadFooPtr is bad, bad, bad. Those of you who have been following releases of the script engines incredibly carefully may remember this story, but hopefully we caused and then fixed the problem sufficiently quickly that the vast majority of customers never noticed.

But I’m getting ahead of myself.

One of the most basic and important rules of COM programming is that you never, ever, **EVER** change an interface without changing its GUID. Or, to put it another way, you simply never change an interface; if you need to, you create an entirely new interface.

You may have wondered why is COM so strict about this rule. It’s because when you write a C++ program that uses an interface, the compiler will generate code that depends upon the particular pattern of stack frame layouts for passing arguments remaining exactly the same even when the user installs a new version of the COM object on their machine. Because COM objects and their “clients” can be versioned independently, the GUID has to be associated with a particular “binary contract” describing how the two pieces of code communicate.

Now, the script engines always talk to scriptable objects via IDispatch or IDispatchEx, so if a scriptable object accidentally breaks its contract, the script engines don’t notice a bit. They let the dispatch layer take care of all those irksome details about stack frame layout, and the dispatch layer figures all this stuff out from the type library data in the object itself.

Perhaps you see where this is going.

The Replace method of the VBScript regular expression object originally took two “in” arguments, both strings, and returned a string. This was the IRegExp interface. Later we realized that we wanted to be able to pass things other than strings for the second argument, so we created a new IRegExp2 interface where the Replace method took a string and a variant, and returned a string. So far, so good.

Then we ported the script engines to the IA64 processor. During the port for some unknown reason one of the developers temporarily changed the second argument from a VARIANT to a VARIANT\*, put a comment in the code saying that this needed to be changed back before it was checked in, checked it in, and, uh, we kinda shipped it like that.

We thoroughly violated the rules of COM, but none of our automated tests showed this, because of course, all of the scripting automated tests were written – you guessed it – in *script*. **We had no tests which used the scriptable objects from early-bound code.** (We do now\!)

Unfortunately, some of our developer customers were using the VBScript regular expression object from C++ and other early bound languages, so as soon as *their* customers upgraded to the latest version of scripting, all hell broke loose. The caller was fulfilling their end of the contract by putting a sixteen byte variant on the stack, and the callee was suddenly expecting a four byte pointer to a variant, so it was both crashing and misaligning the stack.

We *immediately* started getting bug reports, of course. About half the bug reports said “please change this back ASAP, our customers are broken”, and the other half said “we have already shipped a patch of our code to our customers using the new stack layout, please do not change it back, ever, we do not want to ship another patch”.

Obviously we realized that we had badly screwed up and were now in a hideous bind. We came up with three options:

First, we could revert to the previous layout, and inconvenience the small number of customers who had already shipped patches to *their* customers. (And of course, inconvenience all of those people as well, who would have to apply a second patch in as many days.)

Second, we could keep the current layout and tell the rest of our affected customers that they needed to patch our mistake.

Third, we could revert to the previous layout and yet NOT inconvenience anyone.

Obviously the third option is best, but how to do it?

I ended up writing heuristics into the Replace method that examined the stack frame, and then rerouted the call to a special helper method if we detected a VARIANT\* on the stack when we were expecting a VARIANT. The helper method did all the necessary magic to adjust the stack pointer, and so on, so that from the caller’s perspective, everything looked perfectly normal.

Now, remember, we expected that in the vast majority of cases, a VARIANT would be passed, not a VARIANT\*. My heuristic detected which case we were in by extracting four bytes from the middle of the stack frame and calling IsBadReadPtr on it. If that was a valid pointer then I’d cast it to a VARIANT\*, check to see whether the variant type field was a string (by far the most likely argument to Replace), and then call IsBadWritePtr on the bstrVal (because BSTR pointers are [almost always writeable](http://blogs.msdn.com/ericlippert/archive/2003/09/12/52976.aspx)). There were a few other details to the heuristic, but the point is that in the vast majority of cases, IsBadReadPtr was expected to fail. Only in the rare cases where we were in an early-bound and third-party patched program should it succeed.

We shipped the fixed version with the heuristics out immediately.

At the time I did not realize how IsBadReadPtr was implemented. It actually puts a try-except around an attempt to read the pointer\! If it gets an exception then it eats the exception and says yeah, it's bad. Essentially what I had done was inserted an almost-always-thrown exception into every call to Replace.

My goal of not inconveniencing anyone turned out to not quite work out as planned. The Active Server Pages team went nuts that afternoon. Their tests assume that all exceptions, whether handled or not, are flaws somewhere in ASP or scripting. Their test machines are all configured to track and report *all* first-chance exceptions, and suddenly every compatibility and stress test machine they had that happened to do a regular expression replace – that is, approximately all of them – suddenly started reporting hundreds of first-chance exceptions per minute when they upgraded to the latest released version.

So just when I thought that I could breathe easy again for a few minutes I started getting calls from the now highly vexed ASP team. I ended up writing my own versions of IsBadFooPtr which do the right thing – they call VirtualQuery to ask the operating system what the protection state of a given address is, rather than simply trying it out and throwing a first-chance exception on failure.

Long story short: don’t change an interface, don’t call IsBadFooPtr, and run your changes by *all* your important consumers *before* you ship a tricky fix\!

