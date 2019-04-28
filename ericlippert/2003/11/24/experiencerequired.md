# Experience Required

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 11/24/2003 5:55:00 PM

-----

I was intrigued by Neil Deakin's recent [post](http://www.xulplanet.com/ndeakin/article/226?show=c#comments) where he says that when he was a young user, he got mad about all kinds of things that, after a few years as an implementor, he didn't feel mad about anymore.

 

 

I'm with ya, Neil.  Many of my computer geek high school friends were rather surprised to learn that I had taken a job at Microsoft, given my earlier criticisms of all x86 based software.  (I was an Amiga partisan as a teenager.) I feel the same way you do -- I was a naïve teenager.

 

 

Neil, however, doesn't actually explain why it is that he finds his former beliefs to be naïve.  I can't speak for Neil, but I can take a stab at explaining what the revelation was for me.  Basically it comes down to realizing two things:  (1) good design is a series of **difficult compromises**, and (2) software is by its nature incredibly **complex**.  I failed to appreciate either of those facts until I'd actually done a lot of it myself.  Once I appreciated these things, I understood that we have imperfect software BECAUSE the people who make it are dedicated to writing quality software for end users, not in spite of that fact.

 

 

And it doesn't matter what your business model is -- open source or closed source, give away the software and sell consulting, shrink-wrapped boxes, micropayments, freeware, whatever.  The money story is irrelevant; all software development is about coming up with an **implementable**, **usable** design for a given **group of end users** and then finding **enough resources to implement that design**.  There are only a finite number of programmers in the world, and way more problems that need solving than there are resources available to solve them all, so deciding which problems to solve for what users is crucial.

 

 

Sometimes -- not always, but sometimes -- not fixing a trivial, obscure bug with an easy workaround *is* the right thing to if it means that you can spend that time working on a feature that benefits millions of customers. Sometimes features that are useless to you are life-savers for someone else.  Sometimes -- not always, but sometimes -- using up a few more pennies of hard disk space (ten megs costs, what, a penny these days?) is justifiable.  Sometimes -- not always, but sometimes -- proprietary designs allow for a more efficient design process and hence more resources available for a quality implementation.   These are all arguable, and maybe sometimes we humans don't make the \_best\_ choices.  But what is naive is to think that these are not hard problems that people think about hard.

 

 

When I was a teenager, I thought software engineering was trivial -- but my end user was myself, and my needs were simple.  Software development doesn't scale linearly\!  Complex software that exists to solve the real-world problems of millions of end-users is so many orders of magnitude harder to write, that intuitions about the former can be deeply misleading.

 

 

And this is true in ALL design endevours that involve tough choices and complex implementations\!  I was watching the production team commentary track for the extendamix of The Two Towers this weekend, and something that the producers said over and over again was that they got a lot of flack for even slightly changing the story from the book.  But before you criticize that choice, you have to **really think through all the implications** of being slaves to the source material.  How would the movie be damaged by showing Merry and Pippin's story **entirely in flashback**?  By *not* interlacing the Frodo-and-Sam story with the Merry-and-Pippen story?  By making Faramir's choice easy? By adding Erkenbrand as the leader of the Westfold?  Yes, these are all departures from the book, but they also prevent the movie from being confusing and weak at the end.  None are obvious -- they're all arguable points, and ultimately someone had to make a tough decision to produce a movie that wasn't going to satisfy everyone fully, but would nevertheless actually ship to customers\!

 

 

There is no perfect software; the world is far too complex and the design constraints are too great for there to be anything even approaching perfect software.  Therefore, if you want to ship the best possible software for your users, you've got to carefully choose your imperfections.  As the guy who writes tools for you software engineers, my mission is to make tools that afford **deeper abstrations** and hence require **fewer developer resources** to implement.  That's the only way we're going to get past the fundamental problems of complexity and expense.

