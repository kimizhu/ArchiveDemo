# GUID guide, part three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/7/2012 8:18:00 AM

-----

Let's recap: a GUID is a 128 bit integer that is used as a globally unique identifier. GUIDs are not a security system; they do not guarantee uniqueness in a world where hostile parties are deliberately attempting to cause collisions; rather, they provide a cheap and easy way for mutually benign parties to generate identifiers without collisions. One mechanism for ensuring global uniqueness is to generate the GUID so that its bits describe a unique position in spacetime: a machine with a specific network card at a specific time. The downside of this mechanism is that code artifacts with GUIDs embedded in them contain easily-decoded information about the machine used to generate the GUID. This naturally raises a privacy concern.

To address this concern, there is a second common method for generating GUIDs, and that is to choose the bits at random. Such GUIDs have a 4 as the first hex digit of the third section.

First off, what bits are we talking about when we say "the bits"? We already know that in a "random" GUID the first hex digit of the third section is always 4. Something I did not mention in the last episode was that there is additional version information stored in the GUID in the bits in the fourth section as well; you'll note that a GUID almost always has 8, 9, a or b as the first hex digit of the fourth section. So in total we have six bits reserved for version information, leaving 122 bits that can be chosen at random.

Second, why should we suppose that choosing a number at random produces uniqueness? Flipping a coin is random, but it certain does not produce a unique result\! What we rely on here is probabilistic uniqueness. Flipping a single coin does not produce a unique result, but flipping the same coin 122 times in a row almost certainly produces a sequence of heads and tails that has never been seen before and will never be seen again.

Let's talk a bit about those probabilities. Suppose you have a particular randomly-generated GUID in hand. What is the probability that a *specific* time that you randomly generate another GUID will produce a collision with your particular GUID? If the bits are chosen randomly and uniformly, clearly the probability of collision is one in 2<sup>122</sup>. Now, what is the probability that over n generations of GUIDs, you produce a collision with your particular GUID? Those are independent rare events, so the probabilities add (\*); the probability of a collision is n in 2<sup>122</sup>. This 2<sup>122</sup> is an astonishingly large number.

There are on the order 2<sup>30</sup> personal computers in the world (and of course lots of hand-held devices or non-PC computing devices that have more or less the same levels of computing power, but lets ignore those). Let's assume that we put all those PCs in the world to the task of generating GUIDs; if each one can generate, say, 2<sup>20</sup> GUIDs per second then after only about 2<sup>72</sup> seconds -- **one hundred and fifty trillion years** -- you'll have a *very high* chance of generating a collision with your specific GUID. And the odds of collision get pretty good after only thirty trillion years.

But that's looking for a collision with a specific GUID. Clearly the chances are a lot better of generating a collision *somewhere else* along the way. Recall that a [couple of years ago I analyzed how often you get any collision when generating random 32 bit numbers](http://blogs.msdn.com/b/ericlippert/archive/2010/03/22/socks-birthdays-and-hash-collisions.aspx); it turns out that the probability of getting any collision gets extremely high when you get to around 2<sup>16</sup> numbers generated. This generalizes; as a rule of thumb, the probability of getting a collision when generating a random n bit number gets large when you've generated around 2<sup>n/2</sup> numbers. So if we put those billion PCs to work generating 122-bits-of-randomness GUIDs, the probability that two of them somewhere in there would collide gets really high after about 2<sup>61</sup> GUIDs are generated. Since we're assuming that about 2<sup>30</sup> machines are doing 2<sup>20</sup> GUIDs per second, we'd expect a collision after about 2<sup>11</sup> seconds, which is about an hour.

So clearly this system is not utterly foolproof; if we really, really wanted to, we could with high probability generate a GUID collision in only an hour, provided that we got every PC on the planet to dedicate an hour of time to doing so.

But of course we are not going to do that. The number of GUIDs generated per second worldwide is not anywhere even close to 2<sup>50</sup>\! I would be surprised if it were more than 2<sup>20</sup> GUIDs generated per second, worldwide, and therefore we could expect to wait about 2<sup>41</sup> seconds, for there to be a reasonable chance of collision, which is about seventy thousand years. And if we are looking for a collision with a specific GUID, then again, it will take about a billion times longer than our initial estimate if we assume that a relatively small number of GUIDs are being generated worldwide per second.

So, in short: you should expect that any *particular* random GUID will have a collision some time in the next thirty billion trillion years, and that there should be a collision between *any two* GUIDs some time in the next seventy thousand years.

Those are pretty good odds.

Now, this is assuming that the GUIDs are chosen by a perfectly uniform random process. **They are not**. GUIDs are in practice generated by a high-quality **pseudo-random** number generator, not by a **crypto-strength** random number generator. Here are some **questions that I do not know the answers to**:

  - What source of entropy is used to seed that pseudo-random number generator?
  - How many bits of entropy are used in the seed?
  - If you have two virtual machines running on the same physical machine, do they share any of their entropy?
  - Are any of those entropy bits from sources that would identify the machine (such as the MAC address) or the person who created the GUID?
  - Given perfect knowledge of the GUID generation algorithm and a specific GUID, would it be possible to deduce facts about the entropy bits that were used as the seed?
  - Given two GUIDs, is it possible to deduce the probability that they were both generated from a pseudo-random number generator seeded with the same entropy? (And therefore highly likely to be from the same machine.)

I do not know the answers to any of these questions, and therefore it is wise for me to assume that the answers to the bottom four questions is "yes". Clearly it is far, far more difficult for someone to work out where and when a version-four GUID was create than a version-one GUID, which has that information directly in the GUID itself. But I do not know that it is *impossible*.

There are yet other techniques for generating GUIDs. If there is a 2 in the first hex digit of the third section then it is a version 1 GUID with some of the timestamp bits have slightly different meanings. If there is a 3 or 5 then the bits are created by running a cryptographic hash function over a unique string; the uniqueness of the string is then derived from the fact that it is typically a URL. But rather than go into the details of those more exotic GUIDs, I think I will leave off here.

**Summing up:**

  - GUIDs are guaranteed to be unique but not guaranteed to be random. **Do not use them as random numbers.**
  - GUIDs that are random numbers are **not cryptographic strength** random numbers.
  - GUIDs are only unique when everyone cooperates; if someone wants to re-use a previously-generated GUID and thereby artificially create a collision, you cannot stop them. **GUIDs are not a security mechanism.**
  - GUIDs have an internal structure; at least **six of the bits are reserved** and have special meanings.
  - GUIDs are **allowed to be generated sequentially**, and in practice often are.
  - GUIDs are **only unique when taken as a whole**.
  - GUIDs can be generated using **a variety of algorithms**.
  - GUIDs that are generated randomly are **statistically highly unlikely to collide** in the foreseeable future.
  - GUIDs **could reveal information** about the time and place they were created, either directly in the case of version one GUIDs, or via cryptanalysis in the case of version four GUIDs.
  - GUIDs might be generated by some **entirely different algorithm** in the future.

-----

(\*) As the comments point out, this is an approximation that only holds if the probability is small and n is relatively small compared to the total number of possible outcomes.

