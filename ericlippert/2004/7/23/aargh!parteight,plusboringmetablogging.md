# Aargh\! Part Eight, plus Boring Metablogging

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/23/2004 2:51:00 PM

-----

Q: What's a pirate's **second favourite** mode of transportation?

A: A caaaaargh\! Preferably a Jaguaaaaargh, but an early Oldsmobile Cutlass will do.

Q: Very amusing -- but what's a pirate's **favourite** mode of transportation?

A: A pirate ship, silly.

Gripe \#10: Don't use \_alloca

\_alloca allocates memory off the stack instead of the heap. It's very convenient to use when you need a temporary buffer and don't want to worry about freeing it, but I try to avoid it whenever possible for two reasons:

(1) it wrecks the ability for the compiler to make optimizations based on how much stack a function uses, and

(2) when combined with the previous point, it leads to terrible, terrible crashes. If the user asks the machine for lots and lots of heap then the program churns away trying to allocate lots of memory before finally throwing an exception or returning null. The worst that can happen is not too bad. The worst that can happen if the stack is overallocated is that the stack guard page is hit for the second time and the process is terminated. (The exact behaviour of the win32 stack under low-stack conditions is interesting, and I will blog about it at some point; the script engines do some fancy footwork in this department to keep the stack from blowing.)

Here's the thing -- if you have an upper bound on the amount of stack you're going to use, you might as well just ask for it. If you don't have an upper bound -- if the upper bound is determined by something at runtime and it could be arbitrarily big -- then use the heap, because really big stack allocations are too risky.

For example, I'll often see code where \_alloca is used to hold a temporary copy of a string when it is being converted from eight bit ASCII to UTF-16. Bad idea, particularly if the string can come from an untrusted internet script. The string could be a million bytes long, and blow the stack\! Aaargh\! It's drivin' me nuts\!

The thing is, stack allocation is incredibly fast, and in many situations the *common* scenario is for the allocation to be small even if that's not guaranteed. Therefore I often use a "best of both worlds" strategy if I have data that is usually small but might be huge. First, I declare a stack buffer twice the size I expect to use. Then detect if more than that is required and allocate it off the heap if it is. (Of course, you have to remember to clean up the heap buffer if you do that.)

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

You'll note how few new hit television series are called "Extreme Television Editing", or "New Issues In Satellite Uplink Futures Contract Law" -- though come to think of it, either of those would be more likely to catch my interest than the vast majority of the inaptly named "reality" television. Therefore I try to avoid talking about the blog in the blog -- I personally find blogs about blogging boring. I'll keep this short then.

A number of people have asked me questions like "can I quote your blog in my blog?", "can I post your article in my wiki?", "can I use this information in my MSDN Magazine article?" etc.

I'm putting this information on the web because I want it to reach the people who can use it, so by all means, disseminate the information widely. I ask only four things:

1\) Please give me a heads-up before you do.  
2\) Please attribute the source, preferably with a link back to the original.  
3\) Please don't quote me out of context.  
4\) Please understand that by doing so, you remove my ability to correct mistakes in your copy.

UPDATE:[I have posted a follow-up to these guidelines.](http://blogs.msdn.com/ericlippert/archive/2008/10/07/boring-metablogging-part-two.aspx)

