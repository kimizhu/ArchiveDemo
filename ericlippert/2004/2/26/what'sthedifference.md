# What's the difference?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/26/2004 5:08:00 PM

-----

The other day a program manager on another team sent me a question that I've seen in various forms many times over the last few years.  The question was "*Is there any documentation that describes all the differences between JScript and Javascript?"* 

This is definitely [one of those questions](/ericlippert/archive/2003/11/03/53333.aspx "http://blogs.msdn.com/ericlippert/archive/2003/11/03/53333.aspx") where knowing what the questioner is *really* after is necessary before expending any serious effort.  Time for another bad analogy\!

Is there any documentation that describes all the differences between American Standard English and British Standard English?  (“Two countries separated by the same language,” as Shaw said.) 

You could scan the American Heritage Dictionary and maybe a few editions of Webster's into a computer, and do a complex textual difference against Chambers and a few other canonical British dictionaries.  That would at least give you differences in *vocabulary* -- we'd learn that those wacky Brits spell "manoeuvre" the fun way, and that "windshield" and "windscreen" are the same thing, and all that rubbish.  I mean, garbage. 

But that's just vocab.  I was in Northern Ireland once visiting a Canadian friend whose roommates howled with laughter when she told them that I "gave her a ride all the way home".  Apparently that's some sort of *double entendre* in Portrush, or maybe they were just weirdoes, I don't know.  My point is that the number of subtle differences in meaning is enormous. 

And that's just usage\!  A *proper* document of the differences between American and British English would also have an analysis of some of the deeper structural and historical factors, and so on.  There are many thick books on this subject. 

So if some inquisitive soul were to ask one for documentation on the differences between British and American English, it would probably behoove one to find out *why* they need this information *before* writing the doctoral dissertation.  What if you find out that "*I need that documentation because I'm publishing a cookbook written by a British writer. I need to translate it into American Standard English" ?*  Then that dissertation isn't really going to be that helpful, is it?  

I told the PM that the only definitive documentation that I know of that describes most of the **differences** between JScript and Javascript is the aptly named "*Javascript: The Definitive Guide*" by David Flanagan, published by O'Reilly.  The definitive doc which describes the **commonalities** is the ECMAScript specification. 

Now, of course, those documents only describe differences and similarities in **behaviour**.  If you're interested in differences in **implementation** and the historical factors which led to those differences, then you're going to need to talk to the language designers who were in the ECMA working group. 

Also, if you are looking for differences between the IE and Netscape browser object models, the *Definitive Guide* will help you out somewhat, but the ECMA specification will not.  You'll need to consult the W3C DOM specifications for that.  Remember, JScript and Javascript are languages independent of the browser object model. 

And of course, it turned out that the reason the PM was asking in the first place is because a customer had written a script that ran in Netscape, and wanted to know if it would run in IE without changes.  

Well, the way to solve that problem is not to compile an exhaustive list of all the syntactic and semantic differences between JScript and Javascript, and then examine the program in the context of that list\!  No more than the way to translate a cookbook from "kilograms of biscuits" to "pounds of cookies" is to compile a complete list of all the differences between American and British English\! 

There's an easier way to find out if that script works -- try it and see\!  If it dies horribly then it probably doesn't work, so fix it and try again.  If you're worried about more subtle problems, write a series of unit tests and exercise them on both platforms until you're confident that the whole thing works the way you'd like.  

You'd have to do that **regardless** of whether you had a list of differences, so just do it\!

Every now and then someone will ask me what the difference is between Java and Javascript.  I usually ask them right back what the difference between English and German is...

