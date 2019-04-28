# Use the right tool for the job

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/14/2009 6:02:00 AM

-----

Consider the following scheme:

I have some client software which I sell. When the client software starts up for the first time, it obtains a “token” from the user. The token string can be anything; the user can choose their name, their cat’s name, their password, the contents of some disk file, whatever. How the client software obtains the token from the user is not particularly relevant; perhaps they type it in, perhaps it is set in a registry key, but whatever it is, it is the user’s choice what information they use to identify themselves.

Anyway, the first time the client software starts up it obtains the token, and then phones home over the internet to my server. My server does some unspecified expensive and painful process by which I satisfy myself that the user is authorized to use my client software. (Assume for the sake of argument that such a process exists.)

I encrypt the token message with my RSA private key and send it back to the client. The client writes the encrypted message out to disk.

I do not want to go through the expensive and painful process of contacting the server again every subsequent time the software starts up. Instead, the client software decrypts the stored encrypted token using my RSA public key, and then compares it byte-by-byte to the user’s token. (Again, perhaps they type it in, or perhaps it is in a file or registry key, or whatever.) If the encrypted token matches the decrypted token then I know that I’ve authorized this user at least once already, so I don’t need to do it again.

If the token cannot be authenticated then I go into some error path and deny access to the software.

Does any of this seem like a good idea to you? Because *none of this seems like a good idea to me.*

The *fundamental* problem here is that I am attempting to build my own security system instead of purchasing a solution off the shelf that was written by security experts. There are plenty of third-party software licensing solutions available; just buy one.

But it gets worse. Consider the attacks that are possible against this system.

First off, clearly there is nothing stopping any hostile client from simply decompiling my client software, removing the security checks, recompiling, and partying on. *The premise of the scenario is that the client is hostile; doing the security check on the client and trusting the result seems ill-advised.*

Second, suppose the attacker simply obtains a valid encrypted token once, and then puts the pair up on the internet. Any hostile user can now use the unmodified software without contacting the server. If I notice, I suppose I can check my records to see who put their token up on the internet and sue that person. Or can I? As we’ll see later, a clever attacker can get around this.

Third, suppose there is a hostile eavesdropper Eve, listening in on the connection between a benign client (Alice) and the server (Bob). Alice’s token goes out, the encrypted token comes back, and the eavesdropper has now captured enough information to use the software.

One could mitigate this attack by encrypting the token with the public key first. Then only the server can determine the contents of the token, right? As we’ll see, that’s maybe not true either…

Fourth, there’s no mechanism proposed whereby encrypted tokens expire or can otherwise be treated as “known to be compromised” in some way. There’s no way to identify hostile clients and revoke their ability to run the software.

All of these are pretty fundamental problems with this licensing scheme. Those aren’t the specific problem I want to get at today though. **There’s also an extremely dangerous mistake in the design of the crypto mechanism.**

The expression “use the right tool for the job” never applies more than when considering crypto solutions to real-world problems. **Cryptosystems are designed to solve very specific problems very well**, but as soon as you deviate even slightly from the by-design scenario of a cryptosystem, you are off the trail and skiing downhill through the trees; you hit a tree, that’s your fault – you shouldn’t have gone off the trail. 

The RSA cryptosystem is extremely good at solving two problems:

1\) Alice (again, the benign client) wishes to send a message of her choice to Bob (the server), encrypted with Bob’s public key. The message can be decrypted by Bob, but not by Eve, the hostile eavesdropper.

2\) Bob wishes to publish a message of his choice that could only have come from Bob. The message is encrypted with Bob’s private key and can be decrypted by anyone with his public key. The fact that the message can be decrypted by Bob’s public key is evidence that the message came from Bob.

(And obviously these scenarios can be combined; Alice’s message to Bob could be encrypted with *her* private key, so that Bob is the only one who can read it, and Alice is the only one who could have sent it. *Provided, of course, that Alice and Bob have found some secure mechanism by which to reliably exchange public keys*. We assume that such a mechanism exists and has been used. This might not be a valid assumption in the real world\!)

Does the scheme outlined above actually fall into either of these scenarios? No, absolutely not. It might appear that we are in scenario two, but we’re not. Scenario two is that Bob is encrypting **a message of Bob’s choice** with Bob’s private key. But in the scheme outlined here, the server is encrypting a message of the presumed-hostile client’s choice\! **The RSA cryptosystem was not designed to be secure against attacks wherein you allow an attacker to encrypt an arbitrary message with your private key.** Such attacks are called “chosen ciphertext” attacks because the attacker chooses a text which is then “decrypted” -- that is, encrypted with the key that Bob would normally use to decrypt a message from Alice.

And in fact, RSA is not secure against such attacks. If you allow an arbitrary attacker to send you an arbitrary message, which you then encrypt with your private key, then the attacker can carefully craft messages which allow her to do all kinds of crazy things:

\* Most obviously, an attacker can simply make a token which is a document that says something that Bob would never willingly say. The existence of such a service means that no one can use the fact that a message is decryptable with Bob's public key to conclude that this message is vouched for by Bob. But it is far worse than that.

\* Suppose Eve has captured a token message from Alice to Bob, encrypted with Bob's public key. Since encrypting a message with the private key decrypts messages encrypted with the public key, Eve simply sends that as her token to Bob. Bob decrypts Alice's token for Eve. Now Eve knows Alice's token. Of course, Bob can detect that Eve has submitted a token which looks suspiciously like it decrypts to Alice's token, so this might not be the smartest thing for Eve to do. But...

\* An attacker can contrive a *novel* message which, when encrypted with the private key, results in *enough* information to decrypt a previously-captured message that was encrypted with the public key. That is: Alice encrypts her token with Bob's public key and sends it to the server (Bob), who decrypts it with the private key, encrypts the result with the private key, and sends it back. Eve intercepts both messages. Eve then crafts a very special message based on a transformation of the captured content and asks Bob to encrypt it with Bob’s private key. *The contents of the result can be used by Eve to determine what Alice’s token is*. Bob has no easy way of knowing that Eve’s special message is in fact an attempt to decrypt Alice’s token.

\* A hostile authenticated client can choose a token she would like to be encrypted with Bob’s private key, but she does not want Bob to actually see the token. It is possible to contrive a random-seeming token such that you can figure out what a chosen token’s encryption with Bob's private key *would* be. The hostile client can use this attack in order to generate a token/encrypted token pair that can be distributed to other attackers, but which does not point back to the authenticated client.

\* And so on. We haven’t even gotten into issues like correct padding of the message to prevent replay attacks. There are potentially many more issues here.

I don’t know much about cryptography; the most important thing that I do know on the subject is that **I don’t know nearly enough about cryptography to safely design or implement a crypto-based security system**.

So, what have we learned?

0\) If you possibly can, simply don’t go there. **Encryption is extremely difficult to get right and is frequently the wrong solution in the first place.** Use other techniques to solve your security problems.

1\) If the problem is an untrustworthy client then **don’t build a security solution which requires trusting the client.**

2\) If you can use **off-the-shelf parts** then do so.

3\) If you cannot use off-the-shelf-parts and do have to use a cryptosystem then **don’t use a cryptosystem that you don’t fully understand**.

4\) If you have to use a cryptosystem that you don’t fully understand, then at least **don’t use it to solve problems it was not designed to solve**.

5\) If you have to use a cryptosystem to ski through the trees, then at least **don’t allow the presumably hostile client to choose the message which is encrypted.** Choose the token yourself. If the token must include information from the client then sanitize it in some way; require it to be only straight ASCII text, insert random whitespace, and so on.

6\) If you have to allow the client to choose the token, then **don’t encrypt the token itself**. Sign a cryptographically-secure hash of the token. Its much harder for the attacker to choose a token which produces a desired hash.

7\) **Don’t use the same key pair for encrypting outgoing messages as you do for protecting incoming messages**. Get a key pair for each logically different operation you're going to perform.

8\) Encrypt the communications **both ways**.

9\) Consider having a **revocation** mechanism, so that once you know that Eve is attacking you, you can at least revoke her license. (Or you can revoke a known-to-be compromised license, and so on.)

\*\*\*\*\*\*\*\*\*

And with those cheerful thoughts, I'm off on my annual flight south to Canada to spend a whirlwind Christmas vacation with family and friends back in Waterloo. I hope you all have a festive holiday season\!

