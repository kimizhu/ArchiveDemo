<div id="page">

# More on Certificates and Trust Decisions

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/25/2003 4:44:00 PM

-----

<div id="content">

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">I said earlier that certificates could be used to establish identity and hence the right to run trustworthy software on your machine.<span style="mso-spacerun: yes">  </span>I want to emphasize in the strongest possible terms that this is not all that certificates are used for.<span style="mso-spacerun: yes">  </span>A lot of people are confused by this, including a lot of very smart, technically savvy developers.<span style="mso-spacerun: yes">  </span>**Cryptographic security is complex and poorly understood.** </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Let me give you an example.<span style="mso-spacerun: yes">  </span>One time I went to a friend's web site.<span style="mso-spacerun: yes">  </span>My friend is a sharp and highly experienced developer.<span style="mso-spacerun: yes">  </span>His web site happens to use HTTPS, the "secure protocol", and I was very surprised when I got an error message from IE saying that the certificate associated with the site could not be trusted because it was self-signed. <span style="mso-spacerun: yes"> </span>In other words, the certificate says "The bearer's name is Sven Svenson. <span style="mso-spacerun: yes"> </span>The bearer's identity has been confirmed by Sven Svenson." </span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Obviously that is not any kind of identity-establishing evidence\!<span style="mso-spacerun: yes">  </span>To trust the cert you have to trust the certifying authority, but the certifying authority is the person whose identity is in question\!<span style="mso-spacerun: yes">  </span>This chain of trust is not rooted in an explicitly trusted authority, so it is worthless.  What is stopping, say, Bjorn Bjornson from typing up the same certificate?  Nothing\!</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">I asked my buddy why he didn't use a Verisign certificate and his answer surprised me:<span style="mso-spacerun: yes">  </span>"*Nobody's told me that they won't use the service unless I got a Verisign cert. And if they did, I'd probably tell them that they shouldn't use my system if they don't trust me ;)* "</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">My friend was confusing code signing with secure HTTP.<span style="mso-spacerun: yes">  </span>HTTPS does **not** use certs to establish a **trust relationship between the client and the server by establishing the identity of the code author**\! <span style="mso-spacerun: yes"> </span>HTTPS uses certs to establish **secure communication over an insecure network by establishing identity of the web site**.<span style="mso-spacerun: yes">  </span>Those sure sound similar but they are in fact **completely different** uses of certs. <span style="mso-spacerun: yes"> </span>Why?<span style="mso-spacerun: yes">  </span>Because in the first case, it is the **code author** who is (potentially) untrusted.<span style="mso-spacerun: yes">  </span>In the second case, it is the **insecure network** that is (definitely) untrusted.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Before I continue let me briefly explain what the goal of HTTPS is, and how it works.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">The goal of HTTPS is to ensure that when you send information over the internet, two things happen.<span style="mso-spacerun: yes">  </span>First, the information is encrypted so that if it is intercepted, the eavesdropper cannot read it.<span style="mso-spacerun: yes">  </span>Second, the server that you're talking to **really is the server that you THINK you're talking to.**<span style="mso-spacerun: yes">  </span></span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes"></span></span> 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"><span style="mso-spacerun: yes"></span>We have these two goals because there are two threats: first, **someone might be listening in on the traffic between the client and the server**, so you need to ensure that they can't make sense of what they overhear.<span style="mso-spacerun: yes">  </span>Second, **someone may have set up a "fake" server** that fools your client into believing that it is talking to the "real" server, so you need to ensure that you are talking to the "real" server.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">When you go to a secure web site -- like your bank, or Amazon or some other web site involving transmission of credit card numbers over the internet -- the first thing the web site does is send you a certificate containing the name of the web site.<span style="mso-spacerun: yes">  </span>The certificate has been signed by a trusted root -- Verisign, for example.<span style="mso-spacerun: yes">  </span>Your browser can then compare the certificate's web site name with the web site you actually went to, and because it is vouched for by Verisign, you know that **you really are talking to the right server**.<span style="mso-spacerun: yes">  </span>The certificate *also* contains information about how to encrypt messages so that only the server can decrypt them.<span style="mso-spacerun: yes">  </span>That is then enough information to mitigate both threats and establish a secure channel.<span style="mso-spacerun: yes">  </span>(The details of how that channel is established are not particularly germane; I may discuss this in a future post.)</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">I'll say it again: the certificate is NOT there to establish a **trust** relationship between the client and the server.<span style="mso-spacerun: yes">  </span>The certificate is there so that **you know that you are talking to the real server, and no one can eavesdrop**. This is the internet we're talking about: for all you know, **evil geniuses have cut every line between you and the entire internet and have inserted their own evil routers and servers that look just like the real internet**. <span style="mso-spacerun: yes"> (For all you know, this is someone else's blog\!) </span>It's Verisign's job to ensure that those evil geniuses never get their hands on an Amazon.com certificate vouched for by Verisign, because that is the only evidence you have that you are actually talking to the real internet.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">So when you go to a web site with a self-signed HTTPS certificate, IE pops up an "abort or continue?" dialog box.<span style="mso-spacerun: yes">  </span>What that box is trying to communicate is:</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">"Hey, buddy, you are trying to communicate securely with FooCorp's web server but **because the certificate does not establish a chain of trust, IE is unable to determine if the insecure internet between you and FooCorp has been taken over by evil genius hackers who are presently spoofing you into believing that you are on the real internet**.<span style="mso-spacerun: yes">  </span>Do you want to go ahead and possibly be spoofed, or stop right now?"</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">Unfortunately, the dialog box is not particularly clear on this point, and besides, everyone clicks "OK" without reading dialog boxes.<span style="mso-spacerun: yes">  </span>But that's another story.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;">To sum up: whether you trust FooCorp or not is not the issue -- if you're going to send them your credit card number, presumably you already trust them\!<span style="mso-spacerun: yes">  </span>What you don't trust is the **wiring** between you and them.<span style="mso-spacerun: yes">  </span>There might be hackers at their ISP, or your ISP, or at any node on the network.<span style="mso-spacerun: yes">  </span>There might be people listening with packet sniffers and packet replacers. There might be people listening to your wireless network traffic.<span style="mso-spacerun: yes">  </span>HTTPS is designed to protect users against untrustworthy, insecure or hostile **network providers** by ensuring identity and encrypting traffic.</span>

<span style="FONT-SIZE: 10pt; COLOR: purple; FONT-FAMILY: &#39;Lucida Sans Unicode&#39;"> </span>

 

</div>

</div>

