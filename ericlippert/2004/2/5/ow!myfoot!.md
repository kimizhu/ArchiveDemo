# Ow\! My Foot\!

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/5/2004 2:39:00 PM

-----

Numbers are bad.

That might sound like a strange statement coming from a professional software developer with a degree in mathematics. Let me try to justify that.

Numbers are bad the way that power tools are bad. I mean, I like power tools. A lot. But you give me a power tool and pretty soon I start making up excuses to use it, whether it makes any sense to use it or not. And do I take the time to use it carefully, and read the manual first, and put on safety glasses? Usually. But maybe not always -- I haven't nailed my foot to the floor yet, but it's probably only a matter of time. When used judiciously by well-trained people, power tools can do great things, but there are a lot of trips to the emergency room for the rest of us dilettantes.

I see this all the time at work, on the news, everywhere: people lured in by the power of numbers, making up new, but not necessarily sensible, uses for the numeric power tools at their disposal.

Let me give you an example of a time when I almost misused numbers recently.

Once a year in the summer at Microsoft we have a formal review of what we've accomplished over the last twelve months and what we want to accomplish in the next. The review process is kind of complicated, and I don't want to go into the boring details of how it all works. Suffice to say that by the time we're done we have a long document describing accomplishments and goals, where the last thing on the review before the signatures is one of three categories. We could call the categories "I Met My Goals", "I Exceeded My Goals" and "I Totally Rocked\!" (In actuality there are more categories, but the vast majority of employees fall into one of these three buckets, so we might as well consider just these three.) The review system here is not perfect, but it works pretty well and I have no complaints with it.

I said that we *could* call the categories "I Rocked\!" and so on, but *in fact* we call them "3.0", "3.5" and "4.0". By naming the categories after numbers, **it becomes very tempting to use power tools on them**. Last night I got to thinking "*I wonder what my career average for yearly reviews has been?"* Fortunately, I stopped myself because I realized that I was about to nail my foot to the floor.

When you make something into a number you make it amenable to all the power tools that three thousand years of mathematicians have invented, **whether or not those power tools make the slightest bit of sense**. For example, you can't take an average of some Golds, some Silvers and some Bronzes. "Average" simply doesn't apply to those things\! But (let's make up some fictional numbers for a fictional employee Alice) you can take an average of three 4.0's, two 3.5's and a 3.0 \! What could it possibly mean, that average? What can we deduce from it? **Nothing, because we do not know whether the weighting is actually sensible.**

After six years, Alice has a career average of 3.67 -- so what? Compare Alice against Bob, who has a career of six 3.0's. How do we compare two numbers? How about percentages? Yeah, that's the ticket\! Clearly Alice rocks 22% more than Bob. Great\! But to what do we apply that 22%? Salary? Bonus? Vacation time? What does it mean?

The 22% has no meaning because I'm not taking a percentage of anything meaningful\! I have no evidence that the system was deliberately designed so that 3.0, 3.5 and 4.0 have meaningful mathematical properties when averaged. What if the numbers were 1.0, 10.0 and 100.0? Then Alice would have an average of 53, Bob would have an average of 1, Alice is over 5000% "better"\! What if the numbers were 0.0, 1.0 and 2.0? Then Alice would have an average of 1.33, Bob would have an average of 0.0, and **percentage differences would suddenly cease to make any sense.**

**By changing the weightings, the comparative benefit of being an overachiever changes and even the set of mathematical tools you can use changes.** If the weightings are arbitrary, an average is also arbitrary, so two averages cannot be sensibly compared. This is the old computer science maxim in action, only this time its "arbitrariness in, arbitrariness out". You can slice and dice numbers until the cows come home, but since the original data were completely arbitrary choices that do not reflect any measurable feature of reality, the results are going to be little more than curiosities.

ASIDE: I am reminded of a sign I saw in a pottery factory I once toured in England. It said that the 1100 degree Celcius kiln was "eleven times hotter than a kettle boiling at 100 degrees Celcius." I, being a smartass, asked the tour guide whether it was also "negative eleven hundred times hotter than an ice cube at -1 degrees Celcius." I got in reply a very strange look. Some things you just can't sensibly add, subtract, multiply and divide even though they are numbers. (Temperatures can only be divided by temperatures when measured in an absolute scale, like degrees Kelvin.)

It gets worse. In this particular case, we don't even need to consider the question of whether the weighting is sensible to determine that averages are meaningless. We can determine from first principles that an average like "3.67" is meaningless.

Consider a pile of two-by-fours, all of which are exactly 3.0, 3.5 or 4.0 feet long. You toss them into three piles based on their size, multiply the number in each pile by the length, add 'em up, divide through by the total number, and you obtain an extremely accurate average. Why you care what the average is, I don't know -- but at least you definitely have an accurate average.

Now consider a pile of two-by-fours of completely random lengths, but all between 2.75 and 4.25 feet long. Divide them up into piles by rounding to the nearest of 3.0, 3.5 and 4.0, and again, multiply the number in each pile by 3.0, 3.5 or 4.0, add 'em up, divide through, and what have you got?

You've lost so much information by rounding off that you've got an "average" which is only likely to be close to the actual average when the number of two-by-fours is large. Furthermore, I said "random", but I didn't say what the *distribution* was. In a "normal" or "bell curve" distribution, a 2.76 is NOT necessarily just as likely as a 3.01, and you have to take that into account when determining what the likelihood of error in the average is.

When the number of two-by-fours total is tiny -- say, six -- and you're averaging three 4.0 +/- .25, two 3.5 +/- .25 and one 3.0 +/- 0.25, well, I'm not sure what you've got. You've got 3.67 +/- "something in some neighbourhood of 0.25, some of the time", where I could work out the "somethings" if I dug out my STATS 241 notes (or called up my statistician friend David, which would be faster).

My point is that because of rounding error, the so called "average" is smeared over so large a range that it is largely useless.

Probably many of those 3.5's are "actually" 3.7's who didn't *quite* make it to the 4.0 bucket. But that information about the extra 0.2 is lost when it comes time to take an average. 3.67 is way, way too precise. All we know about Alice's average is that her *actual* average is somewhere between 3.0 and 4.0, probably closer to 4.0 -- which we knew already just from the range of the data\!

And we're just on averages\! We haven't even begun to consider the power-tool-mishap possibilities of, say, trend lines.

I'm glad I stopped myself. As a developer and a mathematician, I love both the practicality of numbers and the beauty of mathematics for its own sake. **But just because something has a number attached to it doesn't mean that any of that mathematical machinery is the right tool for the job**. Sometimes numbers are just convenient labels, not mathematical entities.

****

-----

In other news, I was looking at the blog server statistics last night. Of the 207 Microsoft bloggers on this site, I'm ninth in terms of page hits from non-rss-aggregator browsers on a per week basis. Clearly, I rock. But what about you guys, my readers? If we take the number of comments and divide by the number of posts on a per-week basis, and then take a (geometric fit) trend line, I see that... OW\! MY FOOT\!

