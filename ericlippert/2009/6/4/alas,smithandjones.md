# Alas, Smith and Jones

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/4/2009 10:04:00 AM

-----

We have a feature in C\# which allows you to declare a "[friend assembly](http://msdn.microsoft.com/en-us/library/0tke9fxk.aspx)". If assembly Smith says that assembly Jones is its friend, then code in Jones is allowed to see "internal" types of Smith as though they were public(\*). It's a pretty handy feature for building a "family" of assemblies that share common implementation details.

The compiler enforces one particularly interesting rule about the naming of friendly assemblies. If assembly Smith is a strong-named assembly, and Smith says that assembly Jones is its friend, then Jones must also be strong-named. If, however, Smith is not strong-named, then Jones need not be strong-named either.

I'm occasionally asked "what's up with that?"

When you call a method in Smith and pass in some data, you are essentially trusting Smith to (1) make proper use of your data; the data might contain sensitive information that you don't want Smith to misuse, and (2) take some action on your behalf. In the non-computer world an entity which you share secrets with and then empower to take actions on your behalf is called an "attorney" (\*\*).  That got me thinking, which usually spells trouble.

You want to hire an attorney. You go to Smith & Jones, Esqs. You meet with Smith, the trustworthy-looking man on the left of this photo:

[![alassmithandjones](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/AlasSmithandJones_12853/alassmithandjones_3.jpg)](http://www.bbc.co.uk/comedy/alassmithandjones/)

Scenario (1):

You check Smith's ID. It really is Mel Smith. You are contemplating giving Smith a secret, and empowering Smith to act on your behalf with the knowledge of that secret. Smith says "I share my secrets with my partner Griff Rhys Jones, whose identity you may now verify, here is Jones and here is Jones' ID. Jones will also be acting on your behalf. Jones has full access to all secrets which are internal to this organization."

You decide that you trust both Smith and Jones, so you share your secrets with Smith. A series of wacky sketch comedy hijinks then ensues.

(Smith and Jones are both strong-named assemblies. That is, they have "ID" you can verify. Smith's internals are visible to Jones. Everything is fine.)

Scenario (2):

You check the ID. It is Mel Smith. Smith says "By the way, I share my internal secrets with everyone in the world who claims their name is Jones, I don't bother to check IDs, I just trust 'em, and you should too\! Everyone named Jones is trustworthy\! Oh, and that includes my partner over here, Jones. Good old Jones\! No, you can't check Jonesie's ID. Doesn't have any ID, that Jones."

This is ludicrous. Obviously this breaks the chain of trust that you showed you were looking for when you checked Smith's ID in the first place.

(The compiler keeps you from getting into this terrible situation by preventing Smith from doing that; Smith, a strong-named assembly, can only state that it is sharing his internal details with another strong-named assembly. Smith cannot share its internals with any assembly named Jones, a weakly-named assembly in this scenario. We restrict Smith from exposing internals to weakly-named assemblies so that Smith does not accidentally create this security hole or accidentally mislead you into believing that Smith's internals are in any way hidden from partially-trusted code.)

Scenario (3):

You don't bother to check Smith's ID. In fact, you give your secrets and power of attorney to anyone named Smith, regardless of who they actually are or where they came from. The first Smith you pick off the street says "By the way, I'll give my internal secrets to anyone named Jones". 

Do you care?  Why would you?  You're already giving secrets away to anyone named Smith\! If a con artist wants your secrets, he can pretend to be Jones and take them from Smith, or he can pretend to be Smith and get them from you directly, but either way, the problem is that you are giving away secrets and power of attorney to strangers off the street.

(Smith and Jones are both weakly named assemblies, Smith exposes internals to Jones. This is perfectly legal, because if you don't care enough about who you're talking to to check IDs then what's the point of the security system preventing those random strangers from talking to each other?)

\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
(\*) Fully trusted assemblies of course can always use reflection to see private and internal details of other assemblies; that is what "full trust" means. And in the new CLR security model, two assemblies that have the same grant set can see each other's private and internal details via reflection. But friend assemblies are the only mechanism that allow **compile time** support for peering at the internal details of another assembly.

(\*\*) or "delegate", which obviously I want to avoid because that means something technical in C\#.

