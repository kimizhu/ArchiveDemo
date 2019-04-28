# You Want Salt With That? Part One: Security vs Obscurity

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/28/2005 10:24:00 AM

-----

A poster to one of the Joel On Software fora the other day asked what a "salt" was (in the cryptographic sense, not the chemical sense\!) and why it's OK to make salts public knowledge. I thought I might talk about that a bit over the next few entries.

But before I do, let me give you all my standard caution about rolling your own cryptographic algorithms and security systems: **don't.**  It is very, very easy to create security systems which are almost but not quite secure. A security system which gives you a false sense of security is worse than no security system at all\! This blog posting is for informational purposes only; don't think that after you've read this series, you have enough information to build a secure authentication system\!

OK, so suppose you're managing a resource which belongs to someone -- a directory full of files, say.  A typical way to ensure that the resource is available only to the authorized users is to implement some **authentication and authorization scheme**.  You first *authenticate* the entity attempting to access the resource -- you figure out who they are -- and then you check to see whether that entity is *authorized* to delete the file, or whatever.

A standard trick for authenticating a user is to create a *shared secret*. If only the authentication system and the individual know the secret then the authentication system can verify the identity of the user by asking for the secret.

But before I go on, I want to talk a bit about the phrase "security through obscurity" in the context of shared secrets. We usually think of "security through obscurity" as badness. A statistician friend of mine once asked me why security systems that depend on passwords or private keys remaining secret are *not* examples of bad "security through obscurity".

By "security through obscurity" we mean that the system remains secure *only* *if the implementation details of how the security system itself works are not known to attackers*. Systems are seldom obscure enough of themselves to provide any real security; given enough time and effort, the details of the system can be deduced. Lack of source code, clever obfuscators, software that detects when it is being debugged, all of these things make algorithms more obscure, but none of these things will withstand a determined attacker with lots of time and resources. A login algorithm with a "back door" compiled into it is an example of security through obscurity; eventually someone will debug through the code and notice the backdoor algorithm, at which point the system is compromised.

A strong authentication system should be resistant to attack *even if all of its implementation details are widely known*. The time and resources required to crack the system should be provably well in excess of the value of the resource being protected.

To put it another way, **the weakest point in a security system which works by keeping secrets should be the guy keeping the secret**, not the implementation details of the system. Good security systems should be so hard to crack that it is easier for an attacker to break into your house and install spy cameras that watch you type than to deduce your password by attacking the system, **even if all the algorithms that check the password are widely known**. Good security systems let you leverage a highly secured secret into a highly secured resource.

One might think that ideally we'd want it both ways: a strong security system with unknown implementation details. There are arguments on both sides; on the one hand, security *plus* obscurity seems like it ought to make it especially hard on the attackers. On the other hand, the more smart, non-hostile people who look at a security system, the more likely that flaws in the system can be found and corrected. It can be a tough call.

Now that we've got that out of the way, back to our scenario. We want to design a security system that authenticates users based on a shared secret. Over the next few entries we'll look at five different ways to implement such a system, and what the pros and cons are of each.

