# West of House

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/17/2009 10:04:00 AM

-----

 

West of House  
You are standing in an open field west of a white house, with a boarded front door.  
There is a small mailbox here.

**\>open the mailbox**  
Opening the small mailbox reveals a leaflet.

**\>take the leaflet**  
Taken.

**\>read the leaflet**  
"WELCOME TO ZORK\!

ZORK is a game of adventure, danger, and low cunning. In it you will explore some of the most amazing territory ever seen by mortals. [No computer should be without one\!](http://www.infocom-if.org/downloads/downloads.html)"

And thus began in 1984 my lifelong enjoyment of "interactive fiction". Somewhere in a filing cabinet in my office closet I have *hundreds* of hand-drawn boxes-and-lines maps on graph paper reminding me of the layout of each classic Infocom game, where each object can be found, and so on.

I was eleven years old in 1984 and naively thought of myself as a *pretty darn sophisticated* computer programmer. I'd written some simple games for the Commodore PET (the classic [CBM 4032](http://en.wikipedia.org/wiki/Commodore_pet)) at school, obtained my very own [Commodore 64](http://en.wikipedia.org/wiki/Commodore_64) -- thanks mom -- and had found and fixed a bug in professionally-written software. Woo\! But I could not for the life of me figure out how the Implementors at Infocom had written such a huge, complex game that could understand English sentences, maintain consistent positions of hierarchical objects that interact with the environment (that is, the torch is in the coffin which is in the boat being swept downstream), and so on.

When a few years later I did learn how it all worked, I was blown away. It was one of those moments where you suddenly see clearly that there's a whole new way of looking at computers as problem-solving tools. As is well-known now, what the geniuses at Infocom did was **designed and implemented their own virtual machine, the [Z-Machine](http://en.wikipedia.org/wiki/Zmachine)**. They then wrote the games in the bytecode language of the virtual machine. This has two *enormous* advantages.

First, by abstracting over the real machine, you can write your huge, complex game once, write relatively small, easy Z-Machine implementations for as many different brands of computers as you like, and suddenly you have write-once-run-anywhere capabilities, massively increasing the number of platforms you can sell to. Most video games at the time were written in, say, Commodore 64 assembly language, and then re-implemented in Atari assembly language, and so on; the cost scaled linearly with the number of platforms. With the Infocom approach, the per-platform costs were a lot lower.

Second, the VM can implement what we now think of as **paging**. The (immutable) game code can be read in from disk a page at a time, executed, and then discarded. The only stuff that needs to stay in memory is the relatively small current game state and of course the Z-Machine implementation itself. But the code can be huge, much larger than available memory. Back when available memory was 16 to 32 kilobytes, that's a significant advantage.

Nowadays of course lots of people have written their own implementations of the Z-Machine for their own amusement. I never did, but I've always wanted to. Having worked on so many bytecode-interpreted languages over the last fifteen years, it would probably be pretty straightforward to do so. It would be a fair amount of work, but it would be fun to blog about, and I'm sure there would be a lot of great opportunities to illustrate how to make a real-world bytecode interpreter. But on the other hand, I have *lots* of stuff to keep me busy in my spare time already, and a tendency to bite off more than I can finish in one go.

It's therefore quite fortunate that I do not have to do so, because [Mike Greger is doing exactly that.](http://fcd3.blogspot.com/) He's already written several Z-machine implementations in C\# and is blogging about the process. I am very much looking forward to reading Mike's blog and learning about the obscure technical and historical details that make the Z-machine so interesting. Mike tells me that he's having a hard time finding people who are fascinated by both C\# and the Z-Machine; well, count me in, obviously\! I'm sure that a few of my readers are also interested in both.

