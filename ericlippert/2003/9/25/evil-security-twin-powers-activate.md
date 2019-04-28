<div id="page">

# Evil Security Twin Powers... Activate\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/25/2003 1:24:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">One day Peter Torr and I walked into Herman Venter's office (Herman was the architect and primary implementer of JScript .NET).<span style="mso-spacerun: yes">  </span>We were grinning.<span style="mso-spacerun: yes">  </span>Herman knew what that meant.<span style="mso-spacerun: yes">  </span>He took one look at us and said "Uh oh, here come the Evil Security Twins."<span style="mso-spacerun: yes">  </span>And indeed, we had found another potential vulnerability in JScript .NET -- fortunately, before we shipped.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">My Evil Security Twin has an interesting post today about the need to trust both code and the code container.<span style="mso-spacerun: yes">  </span>I thought I might discuss the semantics of some of the words Peter uses.<span style="mso-spacerun: yes">  </span>Security experts often use </span><span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">word pairs such as <span class="ImportantWordsVB">**safe/dangerous**</span>, <span class="ImportantWordsVB">**trusted/untrusted**</span>, and <span class="ImportantWordsVB">**benign/hostile**</span> without </span><span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">clearly distinguishing them from each other. These distinctions </span><span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">are both important and frequently misunderstood. Essentially the differences are these: </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">A particular assembly is <span class="importantwords-PRODUCTION"><span style="FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">**safe**</span></span> if it is unable to harm you: modify your disk, steal your data, deny service, and so on. The safety or dangerousness of an assembly is fundamentally a <span class="importantwords-PRODUCTION"><span style="FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">**technical**</span></span> question: what does the assembly attempt to do? For instance, does it read the disk? Then it might steal secrets. Does it create dialog boxes? Then it might create hundreds of them and make it hard to use your machine. </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">The level of <span class="importantwords-PRODUCTION"><span style="FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">**trust**</span></span> you assign to an assembly is essentially the set of **permissions** you are willing to grant it based on **evidence**. For instance, if you believe that if an assembly from the Internet is granted permission to access your disk then it might misuse that ability to do you harm, you should not grant the permission. </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Administrators express their opinions about trust by configuring their **policies** appropriately. "Do not trust code from the Internet to access the disk" is an example of such a policy. </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Whether an assembly is <span class="ImportantWordsVB">**hostile**</span> or <span class="ImportantWordsVB">**benign**</span> depends on the mindset and goals of the assembly's author. An assembly that creates a file might be perfectly benign, or it might be creating the file in order to consume all the hard disk space on your machine as a denial-of-service attack. An assembly that creates a dialog box that looks just like the Windows password dialog might be attempting to steal your password.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Unfortunately, **there is no way to detect the intent of the programmer**. If there were then security would be easy: use your magical ESP powers to see what the programmer was thinking and then just prevent code written by hostile people who hate you from running\! In the real, non-magical world all that the .NET security system can do is restrict the abilities of an assembly based on the available evidence. </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt">Notice that essentially we use these words technically in the same way we do in everyday speech. A tennis ball is inherently quite <span class="ImportantWordsVB">**safe**</span> and an axe is inherently quite <span class="ImportantWordsVB">**dangerous**</span>. You trust that the axe maker made a **trustworthy** product that **accurately** advertises its **dangerous** nature (rather than masquerading as a safe object which you could then accidentally misuse).<span style="mso-spacerun: yes">  </span>Whether someone carrying an axe is <span class="ImportantWordsVB">**hostile**</span> or <span class="ImportantWordsVB">**benign**</span> has nothing to do with the axe, but everything to do with their **intentions**. And whether you <span class="ImportantWordsVB">**trust**</span> that person to do yard work is a question of what <span class="ImportantWordsVB">**evidence**</span> you have that they're **trustworthy**. A <span class="ImportantWordsVB">**policy**</span> that codifies this trust decision might be "Don't give axes to unidentified strangers."</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt">When we get into **cryptographic evidence**, things get even more confusing.<span style="mso-spacerun: yes">  </span>What are all of these certificates for, and who signed them, and why do we trust them?<span style="mso-spacerun: yes">  </span>We can continue our analogy. <span style="mso-spacerun: yes"></span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt"><span style="mso-spacerun: yes"></span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt">Some guy shows up at your door to do the yard work.<span style="mso-spacerun: yes">  </span>You're considering handing him an axe.<span style="mso-spacerun: yes">  </span>What evidence do you have that this is a wise thing to do?</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt">You could start by asking for some ID.<span style="mso-spacerun: yes">  </span>The guy hands you a card that says "Hi, my name is Sven the Lumberjack".<span style="mso-spacerun: yes">  </span>Does this establish **identity**?<span style="mso-spacerun: yes">  </span>Obviously not.<span style="mso-spacerun: yes">  </span>Does it establish **trustworthiness**?<span style="mso-spacerun: yes">  </span>No\!<span style="mso-spacerun: yes">  </span>So what's missing?</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt">What's missing is the **imprimatur of a trusted authority**.<span style="mso-spacerun: yes">  </span>If instead Sven hands you a driver's license with a photo on it that matches him and a name that matches the name he claims is his, then that's pretty good evidence of identity.<span style="mso-spacerun: yes">  </span>Why? Because *you have a trust relationship with the entity that issued the document*\!<span style="mso-spacerun: yes">  </span>You know that the department of transportation doesn't give a driver's license out to just anyone, and that they ensure that the picture and the name match each other correctly.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt">So far we've established identity but have we established trustworthiness?<span style="mso-spacerun: yes">  </span>Maybe, maybe not.<span style="mso-spacerun: yes">  </span>Identity is a crisp, technical question.<span style="mso-spacerun: yes">  </span>Trustworthiness is a squishy, human question.<span style="mso-spacerun: yes">   </span>Maybe you're willing to say that anyone who is willing to identify themselves is trustworthy.<span style="mso-spacerun: yes">  </span>Maybe you demand extra credentials, like a lumberjack union membership card -- credentials that certify relevant abilities, not just identity.<span style="mso-spacerun: yes">  Maybe you're worried about fake ID's.  All t</span>hat is up to you -- **you're the one handing this guy an axe**.<span style="mso-spacerun: yes">  </span>How paranoid are you?  Set your policies appropriately.  (Some of us are more paranoid than others, like Peter "I only read ASCII email" Torr.  That's hard-core, dude.)</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt">But I digress.  My point is that we construct **chains** of trust.<span style="mso-spacerun: yes">  </span>**Every** link in that chain must hold up, and the chain has to end in some authority which you explicitly trust.<span style="mso-spacerun: yes">  </span> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt">In the computer world, your machine has a certificate store containing a list of "trusted roots", like Verisign.<span style="mso-spacerun: yes">  </span>By putting Versign in the trusted root store, you are saying "I explicitly trust Verisign to do their job of issuing and revoking credentials that establish identity and trustworthiness."<span style="mso-spacerun: yes">  </span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt"><span style="mso-spacerun: yes"></span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt"><span style="mso-spacerun: yes"></span>If Sven goes to Verisign and asks for a "code signing" cert, Verisign will confirm his identity and issue a certificate that says "this guy is known to Verisign and this certificate can be used to sign executable code."<span style="mso-spacerun: yes">  It's a driver's license for coding\!</span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt"><span style="mso-spacerun: yes"></span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt"><span style="mso-spacerun: yes"></span>When you download some code off the internet, the security system checks to see if you trust it -- does it come from someone with an identity verified by Verisign?<span style="mso-spacerun: yes">  </span>Do you trust that individual?<span style="mso-spacerun: yes">  </span>And does the certificate they're showing you say that they have the right to sign executable code?<span style="mso-spacerun: yes">  </span>If so, then the code runs.<span style="mso-spacerun: yes">  </span>If not, then the chain of trust is broken somewhere, and it doesn't run.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; LINE-HEIGHT: 110%; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;; LETTER-SPACING: -0.2pt"> </span>

 

</div>

</div>

