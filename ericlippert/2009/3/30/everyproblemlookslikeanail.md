# Every Problem Looks Like A Nail

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/30/2009 11:10:00 AM

-----

I wish all the questions I got were this straightforward:

> “I need to compare two strings for non-culture-sensitive equality. I notice that there are methods String.Equals and String.Compare which can both do that. What is the guideline on which one I should use?”

I’ll answer your question, but first, a funny story (\*).

[![Roofing Hammer](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/EveryProblemLooksLikeANail_7B33/Roofing%20Hammer_6.gif "Roofing Hammer")](http://www.hammersource.com/Specialty_Trades2.html)My buddy Steve and I were fixing the flashing that is leaking around the seal between my roof and my chimney the other day. “Steve,” I said, “hand me that screwdriver.” Steve handed me his roofing hammer. “No, dude, the *screwdriver*. Thin circular shaft, handle at one end, Robertson bit on the other.”

“Dude, if you wanted the *screw remover* you should have said so.”

Ha ha ha ha ha, I crack myself up. That joke never gets old.

Anyway, I seem to have digressed slightly from the topic of today’s blog.

The documentation in MSDN clearly states that the purpose of String.Equals is **for comparing strings for equality**, and the purpose of String.Compare is for **sorting a list of strings into a culture-sensitive alphabetical order**.

You can use a hammer to drive screws, and in a world without screwdrivers, you might have to. But in a world with screwdrivers, why would you? You would use the tool that was designed for the task.

You can use String.Compare to test for equality if you want, but why would you? You have a method that is designed to do exactly and specifically what you want to do, so use it.

Unless there is a really compelling reason to do otherwise, **always use the tool that was specifically designed for your problem**, if there is one. Don’t use some other tool designed for a different task that just happens to work; that’s a recipe for bugs.

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

(\*) [Not a true story](http://www.netfunny.com/rhf/jokes/92q3/scremov.html), except for the part about my roof leaking at the flashing. I need to fix that.

