# A Book By Any Other Name Would Smell As Sweet

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/2/2009 9:09:00 AM

-----

As you might have gathered from my previous posts on the subject, I occasionally edit technical books as a hobby. It’s nice having a hobby that pays money instead of costing money. And I always learn something from every book.

[![MESWTWSH](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/ABookByAnyOtherNameWouldSmellAsSweet_8AC3/MESWTWSH_5.jpg "MESWTWSH")](http://www.apress.com/book/view/9781893115675)Many years ago, on one of my first editing gigs, the editor asked me my opinion on what the book’s title should be. At the time I had no particularly coherent thoughts about how to title a book, but I’m almost always willing to offer an opinion on things I know little about. So I thought about it for a few minutes and responded that since the book was about *how to manage enterprise IT systems by writing scripts that use the Windows Script Host*, that the title should be “**Managing Enterprise Systems With The Windows Script Host**”. After some back-and-forth discussion between me, the editor and the author, we actually settled on that title, amazingly enough.

**This is not a very good title for what is really a quite good book.** I realize this now, with the hindsight of a lot more experience in thinking about how to name things.

Were I asked to criticize this title now, with that hindsight, I’d ask myself some pointed questions:

Q: Is “**managing**” what the target customer of the book describes themselves as doing?

A: Maybe sometimes, but we can do better. The target audience typically describes themselves as “systems *administrators*”, not “*managers*”. Cut “managing” and replace it with “**Administering**”.

Q: What function does the word “**Enterprise**” serve?

A: It discourages home-computer users from buying the book. Which is good, because they are not our target audience. But “Managing” or “Administering” already does that. It also discourages small- or medium-business sys admins from buying the book and they *are* in the target audience. Cut “Enterprise”.

Q: Do we really need to call out that the things being administered/managed are “**Systems**”?

A: No, obviously that goes without saying. Cut “Systems”.

Q: Then what noun is the object of the verb “Administering”?

A: We need to tell the potential reader that this is a book about administering ***Windows***. The noun “Windows” *must* be in the title somewhere. Admins of other operating systems are explicitly not in our target audience and we must discourage them from buying this book. We look disingenuous if we don’t do that.

Q: What does “**With Windows Script Host**” communicate to the potential buyer?

A: It logically ties the book to a *particular tool* used to solve the customer’s problems. It puts the emphasis on the solution mechanism rather than on the customer’s problem.

Q: If the customer needs a circular hole in a wall, do they care much about who manufactured the drill and the drill bit?

A: No. Though there are better and worse drills, good enough is good enough. As long as the drill works reasonably well, they mostly care about the result.

The implementation details of the solution are not particularly important to the customer, even if they’re the person doing the implementation. What’s important to the customer is the *benefit* obtained by taking the advice in the book.

Q: What is the compelling overall benefit to the customer of using the advice in the book?

A: They can write scripts to *automate tasks*, putting in some up-front time to write the script and accruing the benefit of having a previously time-consuming and irksome administration task performed *quickly and automatically* by the script host.

Whew. That was a lot of pointed questions.

[![AWA](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/ABookByAnyOtherNameWouldSmellAsSweet_8AC3/AWA_3.jpg "AWA")](http://www.apress.com/book/view/1590593979) I wasn’t there for this conversation, but I imagine that the editors and author went through a similar mental process when they prepared the second edition for publication. The second edition has a far better name: **Automating Windows Administration**. It contains keywords “Windows” and “Administration” which attract the target audience – professional administrators of Windows-based systems – and repels people not in the target audience. And it leads with the compelling benefit to the customer: automation. “Automation” makes work easier. Leading with “Managing” puts the emphasis on the hard part of the job, not on making it easier. And the new name doesn’t tie the book or the reader to committing to a particular toolset.

**Good naming is a hugely important aspect of design**; the name of a thing is almost always how the thing first enters the consciousness of a potential consumer. Next time, we’ll explore some more aspects of what makes a name “good” or “bad” for a particular purpose.

