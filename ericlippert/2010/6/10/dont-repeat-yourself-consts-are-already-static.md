<div id="page">

# Don't repeat yourself; consts are already static

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/10/2010 3:30:00 PM

-----

<div id="content">

<div class="mine">

Today, [another entertaining question from StackOverflow](http://stackoverflow.com/questions/2631975/c-using-consts-in-static-classes). Presented again as a dialogue, as is my wont.

**The specification says "*even though constants are considered static members, a constant-declaration neither requires nor allows a static modifier*." Why was the decision made to not force constants to use the static modifier if they are considered static?**

Let's grant that it is sensible that constants are considered to be "static" - that is, members associated with the type itself, rather than members of a particular instance. Let's also grant that "<span class="code">const</span>" has to be in there somewhere. There are three possible choices for how to notate a constant declaration:

1\) Make <span class="code">static</span> **optional**: "<span class="code">const int x</span>..." or "<span class="code">static const int x</span>..." are both legal.  
2\) Make <span class="code">static</span> **required**: "<span class="code">const int x...</span>" is illegal, "<span class="code">static const int x</span>..." is legal  
3\) Make <span class="code">static</span> **illegal**: "<span class="code">const int x</span>..." is legal, "<span class="code">static const int x</span>..." is illegal.

Does that pretty much sum it up?

**Yes. Why did the design team choose option (3) instead of (1) or (2)?**

The design notes from 1999 do not say. But we can deduce what was probably going through the language designer's heads.

The problem with (1) is that you could read code that uses both "<span class="code">const int x</span>..." and "<span class="code">static const int y</span>..." and then you would naturally ask yourself "*what's the difference?*" Since the default for non-constant fields and methods is "instance" unless marked "<span class="code">static</span>", *the natural conclusion would be that some constants are per-instance and some are per-type, and that conclusion would be wrong.* This is bad because it is misleading.

The problem with (2) is that first off, it is redundant. It's just more typing without adding clarity or expressiveness to the language. And second, I don't know about you, but I personally hate it when the compiler gives me the error "You forgot to say the magic word right here. I know you forgot to say the magic word, I am one hundred percent capable of figuring out that the magic word needs to go there, but I'm not going to let you get any work done until you say the magic word". I once heard that tendency of compilers compared to the guy at the party who interrupts your conversation to point out that you said "*a historic occasion*" when clearly you meant to say "*[an historic occasion](http://grammartips.homestead.com/historical.html)*". No one likes that guy.

The problem with (3) is that the developer is required to know that const logically implies static. However, once the developer learns this fact, they've learned it. It's not like this is a complex idea that is hard to figure out.

The solution which presents the fewest problems and costs to the end user is (3).

**That seems reasonable. Does the C\# language apply this principle of eschewing redundancy consistently?**

Nope\! It is interesting to compare and contrast this with other places in the language where different decisions were made.

For example, overloaded operators are required to be both public and static. In this case, again we are faced with three options:

(1) make <span class="code">public static</span> **optional**,  
(2) make it **required**, or  
(3) make it **illegal**.

For overloaded operators we chose (2). Since the "natural state" of a method is private/instance it seems bizarre and misleading to make something that looks like a method public/static invisibly, as (1) and (3) both require.

For another example, a virtual method with the same signature as a virtual method in a base class is supposed to have either "<span class="code">new</span>" or "<span class="code">override</span>" on it. Again, three choices.

(1) make it **optional**: you can say <span class="code">new</span>, or <span class="code">override</span>, or nothing at all, in which case we default to new.  
(2) make it **required**: you have to say <span class="code">new</span> or <span class="code">override</span>, or  
(3) make it **illegal**: you cannot say <span class="code">new</span> at all, so if you don't say <span class="code">override</span> then it is automatically new.

In this case we chose (1) because that works best for the brittle base class situation of someone adds a virtual method to a base class that you don't realize you are now overriding. This produces a warning, but not an error.

Each of these situations has to be considered on a case-by-case basis. The general guidance is not , say "always pick option 3", but rather, to figure out what is least misleading to the typical user and go with that.

Eric is at TechEd; this posting was pre-recorded.

 

</div>

</div>

</div>

