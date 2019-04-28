# More on Certificates and Trust Decisions

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/25/2003 4:44:00 PM

-----

 

I said earlier that certificates could be used to establish identity and hence the right to run trustworthy software on your machine.  I want to emphasize in the strongest possible terms that this is not all that certificates are used for.  A lot of people are confused by this, including a lot of very smart, technically savvy developers.  **Cryptographic security is complex and poorly understood.** 

 

 

Let me give you an example.  One time I went to a friend's web site.  My friend is a sharp and highly experienced developer.  His web site happens to use HTTPS, the "secure protocol", and I was very surprised when I got an error message from IE saying that the certificate associated with the site could not be trusted because it was self-signed.  In other words, the certificate says "The bearer's name is Sven Svenson.  The bearer's identity has been confirmed by Sven Svenson." 

 

 

Obviously that is not any kind of identity-establishing evidence\!  To trust the cert you have to trust the certifying authority, but the certifying authority is the person whose identity is in question\!  This chain of trust is not rooted in an explicitly trusted authority, so it is worthless.  What is stopping, say, Bjorn Bjornson from typing up the same certificate?  Nothing\!

 

 

I asked my buddy why he didn't use a Verisign certificate and his answer surprised me:  "*Nobody's told me that they won't use the service unless I got a Verisign cert. And if they did, I'd probably tell them that they shouldn't use my system if they don't trust me ;)* "

 

 

My friend was confusing code signing with secure HTTP.  HTTPS does **not** use certs to establish a **trust relationship between the client and the server by establishing the identity of the code author**\!  HTTPS uses certs to establish **secure communication over an insecure network by establishing identity of the web site**.  Those sure sound similar but they are in fact **completely different** uses of certs.  Why?  Because in the first case, it is the **code author** who is (potentially) untrusted.  In the second case, it is the **insecure network** that is (definitely) untrusted.

 

 

Before I continue let me briefly explain what the goal of HTTPS is, and how it works.

 

 

The goal of HTTPS is to ensure that when you send information over the internet, two things happen.  First, the information is encrypted so that if it is intercepted, the eavesdropper cannot read it.  Second, the server that you're talking to **really is the server that you THINK you're talking to.**  

 

We have these two goals because there are two threats: first, **someone might be listening in on the traffic between the client and the server**, so you need to ensure that they can't make sense of what they overhear.  Second, **someone may have set up a "fake" server** that fools your client into believing that it is talking to the "real" server, so you need to ensure that you are talking to the "real" server.

 

 

When you go to a secure web site -- like your bank, or Amazon or some other web site involving transmission of credit card numbers over the internet -- the first thing the web site does is send you a certificate containing the name of the web site.  The certificate has been signed by a trusted root -- Verisign, for example.  Your browser can then compare the certificate's web site name with the web site you actually went to, and because it is vouched for by Verisign, you know that **you really are talking to the right server**.  The certificate *also* contains information about how to encrypt messages so that only the server can decrypt them.  That is then enough information to mitigate both threats and establish a secure channel.  (The details of how that channel is established are not particularly germane; I may discuss this in a future post.)

 

 

I'll say it again: the certificate is NOT there to establish a **trust** relationship between the client and the server.  The certificate is there so that **you know that you are talking to the real server, and no one can eavesdrop**. This is the internet we're talking about: for all you know, **evil geniuses have cut every line between you and the entire internet and have inserted their own evil routers and servers that look just like the real internet**.  (For all you know, this is someone else's blog\!) It's Verisign's job to ensure that those evil geniuses never get their hands on an Amazon.com certificate vouched for by Verisign, because that is the only evidence you have that you are actually talking to the real internet.

 

 

So when you go to a web site with a self-signed HTTPS certificate, IE pops up an "abort or continue?" dialog box.  What that box is trying to communicate is:

 

 

"Hey, buddy, you are trying to communicate securely with FooCorp's web server but **because the certificate does not establish a chain of trust, IE is unable to determine if the insecure internet between you and FooCorp has been taken over by evil genius hackers who are presently spoofing you into believing that you are on the real internet**.  Do you want to go ahead and possibly be spoofed, or stop right now?"

 

 

Unfortunately, the dialog box is not particularly clear on this point, and besides, everyone clicks "OK" without reading dialog boxes.  But that's another story.

 

 

To sum up: whether you trust FooCorp or not is not the issue -- if you're going to send them your credit card number, presumably you already trust them\!  What you don't trust is the **wiring** between you and them.  There might be hackers at their ISP, or your ISP, or at any node on the network.  There might be people listening with packet sniffers and packet replacers. There might be people listening to your wireless network traffic.  HTTPS is designed to protect users against untrustworthy, insecure or hostile **network providers** by ensuring identity and encrypting traffic.

