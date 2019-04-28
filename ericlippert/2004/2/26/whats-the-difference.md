<div id="page">

# What's the difference?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/26/2004 5:08:00 PM

-----

<div id="content">

<span>The other day a program manager on another team sent me a question that I've seen in various forms many times over the last few years.  The question was "*<span>Is there any documentation that describes all the differences between JScript and Javascript?"</span>* </span>

<span></span>

<span>This is definitely [one of those questions](/ericlippert/archive/2003/11/03/53333.aspx "http://blogs.msdn.com/ericlippert/archive/2003/11/03/53333.aspx") where knowing what the questioner is *<span>really</span>* after is necessary before expending any serious effort.  Time for another bad analogy\!</span>

<span>Is there any documentation that describes all the differences between American Standard English and British Standard English?  (“Two countries separated by the same language,” as Shaw said.) </span>

<span></span>

<span>You could scan the American Heritage Dictionary and maybe a few editions of Webster's into a computer, and do a complex textual difference against Chambers and a few other canonical British dictionaries.  That would at least give you differences in *<span>vocabulary</span>* -- we'd learn that those wacky Brits spell "manoeuvre" the fun way, and that "windshield" and "windscreen" are the same thing, and all that rubbish.  I mean, garbage. </span>

<span></span>

<span>But that's just vocab.  I was in Northern Ireland once visiting a Canadian friend whose roommates howled with laughter when she told them that I "gave her a ride all the way home".  Apparently that's some sort of *<span>double entendre</span>* in Portrush, or maybe they were just weirdoes, I don't know.  My point is that the number of subtle differences in meaning is enormous. </span>

<span></span>

<span>And that's just usage\!  A *<span>proper</span>* document of the differences between American and British English would also have an analysis of some of the deeper structural and historical factors, and so on.  There are many thick books on this subject. </span>

<span></span>

<span>So if some inquisitive soul were to ask one for documentation on the differences between British and American English, it would probably behoove one to find out *<span>why</span>* they need this information *<span>before</span>* writing the doctoral dissertation.  What if you find out that "*<span>I need that documentation because I'm publishing a cookbook written by a British writer. I need to translate it into American Standard English" ?</span>*  Then that dissertation isn't really going to be that helpful, is it?  </span>

<span></span>

<span>I told the PM that t</span><span>he only definitive documentation that I know of that describes most of the **<span>differences</span>** between JScript and Javascript is the aptly named "*<span>Javascript: The Definitive Guide</span>*" by David Flanagan, published by O'Reilly.  The definitive doc which describes the **<span>commonalities</span>** is the ECMAScript specification. </span>

<span></span>

<span>Now, of course, those documents only describe differences and similarities in **<span>behaviour</span>**.  If you're interested in differences in **<span>implementation</span>** and the historical factors which led to those differences, then you're going to need to talk to the language designers who were in the ECMA working group. </span>

<span></span>

<span>Also, if you are looking for differences between the IE and Netscape browser object models, the *<span>Definitive Guide</span>* will help you out somewhat, but the ECMA specification will not.  You'll need to consult the W3C DOM specifications for that.  Remember, JScript and Javascript are languages independent of the browser object model. </span>

<span></span>

<span>And of course, it turned out that the reason the PM was asking in the first place is because a customer had written a script that ran in Netscape, and wanted to know if it would run in IE without changes.  </span>

<span></span>

<span>Well, the way to solve that problem is not to compile an exhaustive list of all the syntactic and semantic differences between JScript and Javascript, and then examine the program in the context of that list\!  No more than the way to translate a cookbook from "kilograms of biscuits" to "pounds of cookies" is to compile a complete list of all the differences between American and British English\! </span>

<span></span>

<span>There's an easier way to find out if that script works -- try it and see\!  If it dies horribly then it probably doesn't work, so fix it and try again.  If you're worried about more subtle problems, write a series of unit tests and exercise them on both platforms until you're confident that the whole thing works the way you'd like.  </span>

<span>You'd have to do that **<span>regardless</span>** of whether you had a list of differences, so just do it\!</span>

<span>Every now and then someone will ask me what the difference is between Java and Javascript.  I usually ask them right back what the difference between English and German is...</span>

</div>

</div>

