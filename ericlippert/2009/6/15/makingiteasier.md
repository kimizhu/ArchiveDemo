# Making it easier

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/15/2009 12:56:00 PM

-----

I read an article in a technology column on MSNBC a while back, the upshot of which was “I have umpteen-dozen passwords I’ve got to have memorized these days; I thought technology was supposed to make my life easier\!”

Really?

First of all, let’s leave aside the obvious fact that our column writer has a technology-driven standard of living which includes affordable access to varieties of [food, drink, shelter, clothing, medicine, travel, and entertainment](https://www.youtube.com/watch?v=hSELOCMmw4A) all of which were beyond the wildest dreams of the crowned heads of Europe less than two centuries ago, much less the peasants. Technology does seem to have made lives a lot easier all around. But I rather want to address the actual claim: is the purpose of technology to make your *life* easier?

I don’t think it is. I think the purpose of technology is to make *specific tasks* easier.

[![Surprise\!](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/Makingiteasier_8BFF/Surprise_thumb_1.jpg "Surprise!")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/Makingiteasier_8BFF/Surprise_4.jpg) Let’s take a task like “communicate with a person in Australia”. Suppose you live in, say, London. In the early 1800’s, the best way to get a message to a colleague in Australia was to write the message out with your goose quill pen, making multiple copies. Wrap each up in oilcloth, the most waterproof substance of the time. Find several convenient Royal Navy ships or merchant ships going to Australia by different routes and at different times; Napoleon might sink several of them but one of them will probably get through. Entrust your waterproofed messages to the captains of those ships, and then wait the several months it takes to get the message around either the Cape or the Horn and the reply back to you.

I take this opportunity to point out that this already requires an extremely high level of technology. Safely navigating a vessel from England to Australia is not an easy task under the best of circumstances. But obviously we can do better now; the 1800’s solution strikes me as somewhat more arduous and time-consuming as a whole than having to remember your email password.

The task of communicating with Australia is now much easier, thanks to improved technology. And because we have increased the scope of what is feasible, we can take advantage of new capabilities; in the 1800’s, real-time arbitraging of price differences between Australian and European commodity market derivatives was not anyone’s job description, but I’m sure that there are people pursuing this highly complex task today. The technology enables us to find new ways to complicate our lives, ways that were never possible before because we were too busy spending effort on solving other problems.

I think about this kind of thing a lot in the context of programming language design.

When we add a new feature to the language, we almost always make a specific task easier. But in doing so, we also almost always make the language as a whole more complex and therefore harder to learn. We make it more likely that there will be a communications divide between those who know how the new feature works and those who do not, making it harder for everyone to read and understand each other’s code.

This is one of the major reasons why new features start with “-100 points”, as I’ve often said. Coming up with features that make specific tasks easier is, well, easy. But that’s not enough; the feature has to make things so much better that it justifies the additional complexity added to the language.

\*\*\*\*\*\*

Wikimedia Commons photo by Ted “Rufus” Ross.

