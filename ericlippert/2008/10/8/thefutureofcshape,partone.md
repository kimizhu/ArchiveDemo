# The Future of C\#, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/8/2008 11:06:00 AM

-----

An attentive reader pointed me at this [long thread](http://www.codeproject.com/Lounge.aspx?fid=1159&select=2747603&tid=2747603#xx2747603xx) on a third-party forum where some people are musing about possible future features of C\#. There is far, far more here than I possibly have time to respond to in any kind of detail.

Also,** I am not going to spill the beans**. [Anders is giving a talk at the PDC](https://channel9.msdn.com/pdc2008/TL16/) at the end of the month. His talk is titled "The Future of C\#" and I'm not going to steal Anders' thunder (or give our marketing department conniptions) by giving specific details at this time.

Of course, I note also that we have not announced that there even will be a language called C\# 4. We have not announced that there will be new versions of Visual Studio or the CLR either. But we also know that you people who read blogs about programming languages are not idiots. I posted a link to a [video](http://blogs.msdn.com/ericlippert/archive/tags/Video/default.aspx) a while back where Charles interviews the C\# design team; the fact that there is a design team is evidence that we're designing *something*. I've been posting about adding more [covariance and contravariance](http://blogs.msdn.com/ericlippert/archive/tags/Covariance+and+Contravariance/default.aspx) to the type system, Charlie has been collecting comments about what [kinds of interactions people would like to see between C\# and dynamic languages](http://blogs.msdn.com/charlie/archive/2008/01/25/future-focus.aspx). Clearly we're up to something here, and it is not hard to figure out the broad outlines.

This puts me in a bit of a tight spot -- I want to respond to all these great ideas, I want to inform the community about our plans and get feedback early, but at the same time I want to make sure that we don't overpromise and underdeliver, and I want to ensure that we provide information that is fully-baked and professionally presented.

Therefore, I want to talk about these ideas *in the context of the design process we go through here*, rather than in the context of specific implementation decisions of specific features.

I made a new friend at a birthday party I attended the other day. We got to making small talk about our jobs. I mentioned that I designed programming languages as part of my job. Now, let me tell you, when I say that at a party I get one of three responses: (1) that's really cool, (2) excuse me, I need to refill this drink, or (3) a puzzled look, followed by "you mean, someone actually *designs* programming languages? I thought they just, you know, sprang into existence... somehow." I informed my new friend that no, in fact there is a huge amount of design that goes into a language -- six hours a week of meetings for almost ten years, in fact, for C\# alone.

I've already linked several times to [Eric Gunnerson's great post on the C\# design process](http://blogs.msdn.com/ericgu/archive/2004/01/12/57985.aspx). The two most important points in Eric's post are: (1) **this is not a subtractive process**; we don't start with C++ or Java or Haskell and then decide whether to leave some feature of them out. And (2) **just** **being a good feature is not enough**. Features have to be so compelling that they are worth the enormous dollar costs of designing, implementing, testing, documenting and shipping the feature. They have to be worth the cost of complicating the language and making it more difficult to design other features in the future.

After we finished the last-minute minor redesigns of various parts of C\# 3.0, we made a list of every feature we could think of that could possibly go into a future version of C\#. We spent many, many hours going through each feature on that list, trying to "bucket" it. Each feature got put into a unique bucket. The buckets were labelled:

Pri 1: Must have in the next version  
Pri 2: Should have in the next version  
Pri 3: Nice to have in the next version  
Pri 4: Likely requires deep study for many years before we can do it  
Pri 5: Bad idea

Obviously we immediately stopped considering the fours and fives in the context of the next version. We then added up the costs of the features in the first three buckets, compared them against the design, implementation, testing and documenting resources we had available. The costs were massively higher than the resources available, so we cut everything in bucket 2 and 3, and about half of what was in bucket 1. Turns out that some of those "must haves" were actually "should haves".

Understanding this bucketing process will help when I talk about some of the features suggested in that long forum topic. Many of the features suggested were perfectly good, but fell into bucket 3. They didn't make up the 100 point deficit, they just weren't compelling enough.

Next time, I think I'll talk a bit about the design process in the context of OOP.

