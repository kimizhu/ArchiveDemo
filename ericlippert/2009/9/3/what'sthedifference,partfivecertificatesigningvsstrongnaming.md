# What's the Difference, Part Five: certificate signing vs strong naming

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/3/2009 9:48:00 AM

-----

Both strong naming and digital signatures use **public key cryptography** to provide **evidence** about the **origin** of an assembly, so that you can apply **security policy** to determine what **permissions** are granted to the **assembly**.

They differ most importantly not in their mathematical details, but in what problems they are intended to solve.

The purpose of a strong name is solely to ensure that **when you load an assembly by name, you are loading exactly the assembly you think you are loading**. You say "I want to load Frobber, version 4, that came from FooCorp". The strong name gear ensures that you actually load precisely that DLL, and not another assembly called Frobber, version 4, that came from Dr. Evil Enterprises. You can then set security policy which says "if I have an assembly from FooCorp on my machine, fully trust it." These scenarios are the only by-design purposes of strong names.

In order to achieve this, all that is required is that you know the public key token associated with FooCorp's private key. How you come to know that public key token is entirely your business. There is no infrastructure in place designed to help you get that information safely. You're just expected to know what it is, somehow. If evil people can trick you into believing that their key token is in fact FooCorp's key token, then you have a problem. You are expected to come up with some reasonable way to determine what FooCorp's real key token is.

The purpose of a digital signature from a publisher certificate is to **establish a verifiable chain of identity and trust**. The chain of trust goes from a hunk of code of unknown or uncertain origin up to a "trusted root" -- an entity which you have configured your operating system to trust.

You download some code, and the code has a digital signature with a certificate from FooCorp. You examine the certificate and it says "this program comes from FooCorp. The accuracy of this certificate is vouched for by VeriSign." Since VeriSign is one of your trusted roots, you now have confidence that this code actually did come from FooCorp.

Notice how much more complex the problem solved by digital signatures is. We're not trying to simply determine "is this hunk of code associated with this name or not?" Instead we're trying to determine where did this code come from, and who vouches for the existence of the company allegedly responsible, and should we trust that company?

The difference between strong names and digital signatures emphasizes what is hard about crypto-based security. The hard problem isn't the cryptography; that's just math. The hard problem is safely managing distribution of information about the keys and associating them with the correct entities. Strong names, because they attempt to solve a very small but important problem, do not have key management issues. Or, rather, they foist the key management problem off to you, the user. Digital signatures are all about trying to automate safe distribution of key information via certificates, in order to solve much more complex problems of trust and identity.

