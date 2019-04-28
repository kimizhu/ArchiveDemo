# Socks, birthdays and hash collisions

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/22/2010 6:18:00 AM

-----

[![Nath knits awesome socks](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/Socksbirthdaysandhashcollisions_9FBE/Socks_3.jpg "Nath knits awesome socks")](http://nathknits.wordpress.com/2009/06/16/knitting-catch-up/) Suppose you’ve got a huge mixed-up pile of white, black, green and red socks, with roughly equal numbers of each. You randomly choose two of them. What is the probability that they are a matched pair?

There are sixteen ways of choosing a pair of socks: WW, WB, WG, WR, BW, BB, … Of those sixteen pairs, four of them are matched pairs. So chances are 25% that you get a matched pair.

Suppose you choose three of them. What is the probability that amongst the socks you chose, there exists at least one matched pair?

Well, we already know that chances are 25% after you pick out just the first two. If you get a matched pair right off, great. If you don’t, then there are two colours in hand you might match. So the odds are going to be a lot better.

There are 64 ways of choosing three socks: WWW, WWB, … and so on. Of those 64 possible combinations, 40 of them have at least one matched pair, so that’s about a 63% chance.

Suppose you choose four. There are 256 possible combinations, 232 of which have at least one matched pairs, so that’s a 91% chance.

Of course by the time we get to five socks, we have a 100% chance of getting a pair; five socks, four colours, there have got to be two alike.

It might appear that we’ve slightly messed up the probabilities here because once you choose one white sock, odds are slightly better that the next sock you pick will not be white, since there are now fewer white socks in the pile. But if the pile is big enough then we can neglect this minor problem.

From now on we’ll call getting a matched pair a “collision”.

It seems clear that as we increase *the number of possible sock colours*, we decrease *the probability of getting a collision in some sample size*. And as we increase *the size of the sample*, we increase *the probability of the sample containing a collision*.

Suppose you have 365 different colours of socks - perhaps each sock has a number on it giving its colour number, so that we can tell them apart - and a pile of about six billion socks, with roughly equal numbers of each sock colour. What is the probability that we’ll get a collision if we pull out two socks at random? One in 365, clearly. Three socks? A little bit better than double that.  And so on. To work out the exact probabilities we’d work out the number of possible combinations, and the number of those combinations that contain at least one collision.

Turns out that the point where you have a better than 50% chance of having a collision is 23 socks. This is the famous “birthday paradox”; if instead of 365 colours of socks we have 365 possible birthdays (ignoring leap years, the fact that more people are born on certain days than others, and so on) and we have a large group of people to choose from at random, then once you get to 23 people the odds are about fifty-fifty that two of them have the same birthday. By 50 people, chances are about 97% that two have the same birthday.

Which is maybe a nice party trick next time you’re at a party with 30 to 50 people – if you go around the room and ask everyone to say their birthday, odds are very good that two people will say the same day. But what’s my point?

Suppose you have just over four billion possible sock colours and a truly enormous supply of socks of each colour, such that each one is about equally likely. You start pulling socks out of the pile. What is the probability that you get a collision based on the number of socks you pull out? Four billion is an awfully big number compared to 4 or 365. What’s your intuition about the likelihood of a collision? How long until you have to start worrying about it?

Not nearly as long as you might think. I’ve worked out the math and summarized it in this handy log-log chart:

[![Collision](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/Socksbirthdaysandhashcollisions_9FBE/Collision_thumb_1.png "Collision")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/Socksbirthdaysandhashcollisions_9FBE/Collision_4.png)

Man, is there anything better than getting a straight line on a log-log chart?

Anyway, you end up with a 1% chance of a collision after about 9300 tries, and a 50% chance after only 77000 tries. By the time you get into the mid six-digit numbers chances are for practical purposes 100% that there is a collision in there somewhere.

This is why **it is a really bad idea to use 32 bit hash codes as “unique” identifiers.** Hash values aren't *random* per se, but if they're well-distributed then they might as well be for our purposes. You might think “well, sure, obviously they are not truly unique since there are more than four billion possible values, but only four billion hash codes available. But there are so many possible hash values, odds are really good that I’m going to get unique values for my hashes”. But are the chances really that good? 9300 objects is not that many and 1% is a pretty high probability of collision.

