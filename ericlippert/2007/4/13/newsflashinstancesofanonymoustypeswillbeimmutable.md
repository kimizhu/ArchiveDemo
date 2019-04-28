# News flash: Instances of anonymous types will be immutable

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/13/2007 1:11:00 PM

-----

I've been meaning to write a blog post about a recent design change that we've made here on the C\# team. As you may have gathered from my previous posts, we are introducing anonymous tuple types in C\# 3.0. We've recently decided to make instances of those tuple types immutable; once you construct a particular instance, it is stuck with those values forever.

There are a lot of good reasons to make tuples immutable: it ensures that they have stable hash codes, functional-style programming works better on immutable objects (functional programming is all about avoiding side effects; mutating state is a side effect) and concurrent programming works better on immutable objects.

So anyway, I was going to write this big long blog post about all these things, but I don't have to because the [guy across the hall from me, Sreekar Choudhary, has started blogging](http://blogs.msdn.com/sreekarc/). Sree is our local expert on anonymous types, extension methods and everything to do with the expression evaluator that runs in the debugger (ie, the watch window.) I'm looking forward to seeing what he has to talk about in his new blog.

