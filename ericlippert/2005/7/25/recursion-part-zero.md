<div id="page">

# Recursion, Part Zero

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/25/2005 6:00:00 AM

-----

<div id="content">

I’ve mentioned recursive programming techniques several times over the years in this blog ([Topological Sort](http://blogs.msdn.com/ericlippert/archive/2004/03/16/90851.aspx), [How Not To Teach Recursion](http://blogs.msdn.com/ericlippert/archive/2004/05/19/135392.aspx), [Fibonacci Challenge](http://blogs.msdn.com/ericlippert/archive/2004/05/20/136327.aspx), [Recursion and Dynamic Programming](http://blogs.msdn.com/ericlippert/archive/2004/07/21/189974.aspx), [Sometimes Breadth Is Better Than Depth](http://blogs.msdn.com/ericlippert/archive/2004/09/27/234826.aspx), and probably a few others).  A lot of developers, particularly those without a formal education in computer science, are still pretty vague on this weird idea of functions that call themselves.  If you’re interviewing at Microsoft for a development position you can expect that at least one person will ask you a whiteboarding question that involves some kind of recursive problem. This isn’t because we’re a bunch of architecture astronauts or theory wonks, it’s because **if you want to be a dev here, recursion is simply one of the tools that you have to have in your toolbox.** We use recursive data structures and algorithms quite a bit, and people are expected to know how they work. I’d therefore like to take this opportunity to do a series on some of the theory and practice of writing recursive and recursive-like functions. I want to show that (a) they're not as mysterious as they seem, **but** (b) that they are potentially dangerous, and (c) there are **some absolutely crazy-but-fun** techniques for turning a recursive program into a nonrecursive program. Hopefully by the time we're done here your head will hurt, but hey, **it builds character**. I'm going to get as much of this done as possible and queued up, because I'm going to go dark shortly. I'm getting married in less than two weeks\! Between getting ready for that and finishing up stuff at work, I'm going to be swamped. (And you'd better believe I'm not blogging about recursive programming techniques from my honeymoon.)  I'll be back in late August and things should pick up then.

</div>

</div>

