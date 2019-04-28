# Evil Security Twin Powers... Activate\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/25/2003 1:24:00 PM

-----

One day Peter Torr and I walked into Herman Venter's office (Herman was the architect and primary implementer of JScript .NET).  We were grinning.  Herman knew what that meant.  He took one look at us and said "Uh oh, here come the Evil Security Twins."  And indeed, we had found another potential vulnerability in JScript .NET -- fortunately, before we shipped.

 

 

My Evil Security Twin has an interesting post today about the need to trust both code and the code container.  I thought I might discuss the semantics of some of the words Peter uses.  Security experts often use word pairs such as **safe/dangerous**, **trusted/untrusted**, and **benign/hostile** without clearly distinguishing them from each other. These distinctions are both important and frequently misunderstood. Essentially the differences are these: 

 

 

A particular assembly is **safe** if it is unable to harm you: modify your disk, steal your data, deny service, and so on. The safety or dangerousness of an assembly is fundamentally a **technical** question: what does the assembly attempt to do? For instance, does it read the disk? Then it might steal secrets. Does it create dialog boxes? Then it might create hundreds of them and make it hard to use your machine. 

 

 

The level of **trust** you assign to an assembly is essentially the set of **permissions** you are willing to grant it based on **evidence**. For instance, if you believe that if an assembly from the Internet is granted permission to access your disk then it might misuse that ability to do you harm, you should not grant the permission. 

Administrators express their opinions about trust by configuring their **policies** appropriately. "Do not trust code from the Internet to access the disk" is an example of such a policy. 

 

 

Whether an assembly is **hostile** or **benign** depends on the mindset and goals of the assembly's author. An assembly that creates a file might be perfectly benign, or it might be creating the file in order to consume all the hard disk space on your machine as a denial-of-service attack. An assembly that creates a dialog box that looks just like the Windows password dialog might be attempting to steal your password.

 

 

Unfortunately, **there is no way to detect the intent of the programmer**. If there were then security would be easy: use your magical ESP powers to see what the programmer was thinking and then just prevent code written by hostile people who hate you from running\! In the real, non-magical world all that the .NET security system can do is restrict the abilities of an assembly based on the available evidence. 

 

 

Notice that essentially we use these words technically in the same way we do in everyday speech. A tennis ball is inherently quite **safe** and an axe is inherently quite **dangerous**. You trust that the axe maker made a **trustworthy** product that **accurately** advertises its **dangerous** nature (rather than masquerading as a safe object which you could then accidentally misuse).  Whether someone carrying an axe is **hostile** or **benign** has nothing to do with the axe, but everything to do with their **intentions**. And whether you **trust** that person to do yard work is a question of what **evidence** you have that they're **trustworthy**. A **policy** that codifies this trust decision might be "Don't give axes to unidentified strangers."

 

 

When we get into **cryptographic evidence**, things get even more confusing.  What are all of these certificates for, and who signed them, and why do we trust them?  We can continue our analogy. 

Some guy shows up at your door to do the yard work.  You're considering handing him an axe.  What evidence do you have that this is a wise thing to do?

 

 

You could start by asking for some ID.  The guy hands you a card that says "Hi, my name is Sven the Lumberjack".  Does this establish **identity**?  Obviously not.  Does it establish **trustworthiness**?  No\!  So what's missing?

 

 

What's missing is the **imprimatur of a trusted authority**.  If instead Sven hands you a driver's license with a photo on it that matches him and a name that matches the name he claims is his, then that's pretty good evidence of identity.  Why? Because *you have a trust relationship with the entity that issued the document*\!  You know that the department of transportation doesn't give a driver's license out to just anyone, and that they ensure that the picture and the name match each other correctly.   

 

 

So far we've established identity but have we established trustworthiness?  Maybe, maybe not.  Identity is a crisp, technical question.  Trustworthiness is a squishy, human question.   Maybe you're willing to say that anyone who is willing to identify themselves is trustworthy.  Maybe you demand extra credentials, like a lumberjack union membership card -- credentials that certify relevant abilities, not just identity.  Maybe you're worried about fake ID's.  All that is up to you -- **you're the one handing this guy an axe**.  How paranoid are you?  Set your policies appropriately.  (Some of us are more paranoid than others, like Peter "I only read ASCII email" Torr.  That's hard-core, dude.)

 

 

But I digress.  My point is that we construct **chains** of trust.  **Every** link in that chain must hold up, and the chain has to end in some authority which you explicitly trust.   

 

 

In the computer world, your machine has a certificate store containing a list of "trusted roots", like Verisign.  By putting Versign in the trusted root store, you are saying "I explicitly trust Verisign to do their job of issuing and revoking credentials that establish identity and trustworthiness."  

If Sven goes to Verisign and asks for a "code signing" cert, Verisign will confirm his identity and issue a certificate that says "this guy is known to Verisign and this certificate can be used to sign executable code."  It's a driver's license for coding\!

When you download some code off the internet, the security system checks to see if you trust it -- does it come from someone with an identity verified by Verisign?  Do you trust that individual?  And does the certificate they're showing you say that they have the right to sign executable code?  If so, then the code runs.  If not, then the chain of trust is broken somewhere, and it doesn't run.

