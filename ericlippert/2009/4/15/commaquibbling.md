# Comma Quibbling

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/15/2009 10:06:00 AM

-----

\[UPDATE: Holy goodness. Apparently this was a more popular pasttime than I anticipated. There's like a hundred solutions in there. Who knew there were that many ways to stick commas in a string? It will take me some time to go through them all, so don't be surprised if it's a couple of weeks until I get them all sorted out.\]

[![Comma](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/CommaQuibbling_DE57/Comma_thumb.jpg "Comma")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/CommaQuibbling_DE57/Comma_2.jpg) The point of Monday’s post about comma-separated lists was not so much about the actual problem; it’s a rather trivial problem. Rather, I wanted to make two points. First, stating the *actual problem* rather than a much harder and more general version of the problem is likely to get you a realistic solution to your actual problem much faster. And second, reworking the statement of the problem into an equivalent but structurally different statement is a great way to see solutions that you might have otherwise missed.

But whenever I make a post illustrating such points with a specific example, lots of people pipe up with their ideas for how to solve the specific example. Which is awesome; I encourage this behaviour.

So in that spirit, here’s a slightly harder version of the string concatenation problem, just for the fun of it. Write me a function that takes a non-null IEnumerable\<string\> and returns a string with the following characteristics:

(1) If the sequence is empty then the resulting string is "{}".  
(2) If the sequence is a single item "ABC" then the resulting string is "{ABC}".  
(3) If the sequence is the two item sequence "ABC", "DEF" then the resulting string is "{ABC and DEF}".  
(4) If the sequence has more than two items, say, "ABC", "DEF", "G", "H" then the resulting string is "{ABC, DEF, G and H}". (Note: no Oxford comma\!)

I think you get the idea. You can post your solution in the comments or use the link on the blog page to email your solution to me.

The strings in the sequence can be assumed to be non-null but can otherwise be any string value, including empty strings or strings containing commas, braces and "and".

There’s no size limit on the sequence; it could be tiny, it could be thousands of strings. But it will be finite.

All you get are the methods of IEnumerable\<string\>; if you want to make that thing into a list or an array, you’re going to need to do that explicitly rather than casting it and hoping for the best.

I am particularly interested in solutions which make the semantics of the code very clear to the code maintainer.

Of course, C\# is most interesting to me, but if there are neat ways to express this in other languages, I’d love to see them too.

If there are any particularly amusing or interesting implementations I’ll dissect them on the blog in a future episode, probably in a week or so. I’m not going to have time to do a detailed analysis of every one.

And… go\!

