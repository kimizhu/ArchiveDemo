# Preventing third-party derivation, part two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/6/2008 10:50:00 AM

-----

If you find a class in a library that has all private/internal constructors, it is pretty clear that the author of the class is sending you the deliberate message that this class is not to be extended by you. In our previous example, it was not so clear. Obviously the author of the class in question did not think that extending the class was a by-design scenario (because if they did, their scenario tests would have immediately caught the problem) but it is not so clear that this restriction was deliberate, rather than accidental.

I say that writing code and writing mystery novels are two entirely different things; don't give the user of your code a twisty-turny puzzle to figure out with a surprise at the end. Make it obvious. If I wanted to make a class that was not third-party extensible by design, I'd not only make the constructors private, I'd slap one of these on the class:

 

\[PermissionSet(SecurityAction.InheritanceDemand, Name = "FullTrust")

Meaning: “Only classes inside assemblies which the current user’s security policy fully trusts may extend my class.”

Or this:

 

\[StrongNameIdentityPermissionAttribute(SecurityAction.InheritanceDemand,  
PublicKey="00240000048000009400000006020000002400005253413100040000010001005" +  
"38a4a19382e9429cf516dcf1399facdccca092a06442efaf9ecaca33457be26ee0073c6bde5" +  
"1fe0873666a62459581669b510ae1e84bef6bcb1aff7957237279d8b7e0e25b71ad39df3684" +  
"5b7db60382c8eb73f289823578d33c09e48d0d2f90ed4541e1438008142ef714bfe604c41a4" +  
"957a4f6e6ab36b9715ec57625904c6")\]

Meaning: “Only classes inside assemblies that were signed with the corresponding private key may extend my class.” (FYI, the binary blob there is an encoding of Microsoft's public key.)

You'll notice that I said that I would make the constructors inaccessible *and* put a security annotation in the metadata as well. Why both? Two reasons. First, **defense in depth**. Don't just make something impossible, make it impossible in multiple ways. That way, if you're wrong about one of them, you've still got all the others to back you up.

Second, because in this case, **going with just the metadata solution is not enough**.

This was surprising to me. I cut my teeth on the .NET security system back in its earliest incarnation; things have changed since then. The meaning I gave of the second attribute up there is subtly wrong. It should have said

 “Only classes inside partially trusted assemblies that were signed with the corresponding private key, or any fully trusted assembly, may extend my class.”

In recent versions of .NET, "full trust means full trust". That is, fully-trusted code satisfies all demands, including demands for things like "was signed with this key", whether it actually was signed or not.

Isn't that a deadly flaw in the security system? No. Fully trusted code always had the ability to do that, because fully trusted code has the ability to control the evidence associated with a given assembly. If you can control the evidence,then you can forge an assembly that looks like it came from Microsoft, no problem. (And if you already have malicious full-trust code in your process then you have already lost -- it doesn't need to impersonate Microsoft-signed assemblies; it already has the power to do whatever the user can do.)

**Credit where it's due:** Special thanks goes out to .NET security guru [Shawn Farkas](http://blogs.msdn.com/shawnfa/) for taking the time to explain to me the differences between various versions of the .NET security system.

**Coming up:** in anticipation of the PDC later this month, over the next few postings I'll be responding vaguely and overcautiously to recent newgroup postings about proposed language features for some fictitious, hypothetical, unnannounced language called "C\# 4".

