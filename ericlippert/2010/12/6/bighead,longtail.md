# Big head, long tail

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/6/2010 6:12:00 AM

-----

Here's a graph of the population size of the [one hundred largest urban areas in Canada](http://en.wikipedia.org/wiki/List_of_the_100_largest_metropolitan_areas_in_Canada): (Click on the graph for a larger version.)

[![4520.image\_6](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/7206.4520.image_6_thumb.png "4520.image_6")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/7888.4520.image_6_2.png)

Notice how there is an enormous spiky "head" on this graph: Toronto, Montreal and Vancouver are quite large cities by any measure. Then there is an immediate sharp drop to a mass of a dozen or so largish urban areas, including my home region of Kitchener-Waterloo-Cambridge. And then there is a very long tail. The most populous urban area with five million people is about 200 times larger than the 100th, with about 25 thousand people. Were we to extend this list out to the next hundred urban areas you can imagine that tail growing ever fainter and more indistinct.

It is very hard to read a graph like this, where there is a difference of a factor of 200 between the largest and smallest datum and the data are for the most part all crammed up against the axes. When data span multiple orders of magnitude it is a good idea to plot it on a log scale. I'm also going to abandon the bar chart format; even though this is not continuous data, let's plot it as a line:

[![2134.image\_4](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/7043.2134.image_4_thumb.png "2134.image_4")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/0525.2134.image_4_2.png)

OK, that's easier to read. We still get the idea that there's a big head here followed by a long tail, but because we've made the "thin" bits much "thicker" it is easier to see what is going on. Making the vertical axis logarithmic works out well. What gets really interesting though is if we make the horizontal axis also logarithmic. (Again, remember, this is *not actually continuous data* and there are only ten discrete data points in the left half of the graph, and the remaining ninety in the right half.) Notice how even though we are cramming most of the data points into a much smaller space, subtle details are even easier to read in the log-log scaling than the log scaling:

[![0486.image\_8](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/2843.0486.image_8_thumb.png "0486.image_8")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/1602.0486.image_8_2.png)

It is almost a straight line\! [And how I love straight lines on log-log charts\!](http://blogs.msdn.com/b/ericlippert/archive/2010/03/22/socks-birthdays-and-hash-collisions.aspx) You know what that original graph resembles? **That thing sure looks like a hyperbola**. Let's graph that against an actual hyperbola, which of course *is* a straight line (of slope -1) on a log-log chart. Here I've added the hyperbola y = 4000000 / n :

[![4617.image\_10](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/1018.4617.image_10_thumb.png "4617.image_10")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/6403.4617.image_10_2.png)

Uncanny\! To a reasonable approximation, **the population of the nth largest urban area in Canada has 1/n of the population of the largest area.**

I also grabbed from Wikipedia [a list of approximate sales figures of the hundred best selling books](http://en.wikipedia.org/wiki/List_of_best-selling_books). If we graph those figures (in millions) on a log-log scale against a hyperbola we get:

[![0081.image\_12](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/8546.0081.image_12_thumb.png "0081.image_12")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/1106.0081.image_12_2.png)

The first few data points are outliers, partly because the records of worldwide book sales from the last two centuries are not accurate and partly because at the top end there are only so many purchases of The Lord of the Rings that weren't accompanied by a purchase of The Hobbit. Frankly, 200 million copies of Dickens seems *very* low. But the rest of the data certainly seems to at least roughly conform to our long tail. Again, to a reasonable approximation **the sales numbers of the nth most popular book are 1/n of the sales of the most popular book**.

This is why I am never going to become wealthy writing a book. The "long tail" of the publishing industry is *immensely* long, even if you only consider the best selling books in a given year (instead of for 150 years, as I have here.) Obviously only a select few ever get to the exalted first couple dozen positions in the ranking in any given year; **if your book isn't on the top 1000, then odds are you will be getting far less than 1/1000th of the sales of the best selling book.**

Data sets which have the property that when sorted into descending order, the nth member has 1/n the size of the first member are called [Zipf's Law](http://en.wikipedia.org/wiki/Zipf%27s_law) (\*) data sets. George Zipf was a linguist who noticed that the frequency of a word's usage in English is approximately proportional to its position on the frequency list.  That is to say, if the most common word, "the", is 70000 of every million words in a corpus then the second then "of" makes up 35000 (half), "and" makes up 23000 (a third), and of course then there is the extraordinarily long tail of words that are used only once every thousand, ten thousand or hundred thousand words on average. (\*\*)

Just because none of us are going to get rich writing books is no reason not to love the Zipf's Law distribution; there are up sides as well. Here's another graph:

[![0312.image\_16](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/0447.0312.image_16_thumb.png "0312.image_16")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/6403.0312.image_16_2.png)

As you can see, this is slightly shallower a curve than a perfect hyperbola; the head isn't quite so huge and the tail isn't quite so sparse, but it is the same basic "big head, long tail" shape.

You know how when you install a new piece of software, often there is a checkbox in the installer that says "help us make this software better"? And you know how when an application crashes, you get the option to automatically report the crash to Microsoft? We have automated software that analyzes all those reports and attempts to figure out which "bucket" each crash report goes into. The log-log graph above is the **relative frequency distribution of the top 100 automatically reported crashes in Visual Studio** (which of course includes crashes both from Visual Studio proper, and from third-party components.) Because this forms a Zipf's Law distribution, it means that **we can concentrate our bug-fixing efforts on the big head first**. The considerable majority of crashing bugs that are actually experienced by users is the result of only a few dozen actual bugs, which are then prioritized extremely highly.

Please, if you're not doing so already, **get in the habit of turning on the software metrics and crash reporting**. It is an enormous help to us in determining how to concentrate our efforts on fixing flaws in our own products, and working with our partners at other companies to help them fix their problems as well. In my opinion no crashing bug is acceptable, but in a world where we have finite effort at our disposal, it's really important to get rid of the big head before tackling the long tail.

-----

(\*) Of course Zipf's Law is no more a "Law" than the equally misnamed "[Benford's Law](http://blogs.msdn.com/b/ericlippert/archive/tags/benford_2700_s+law/)" or "Moore's Law". Unlike the laws of physics, there is no external reality enforcing an inverse power distribution on city sizes, word distributions or book sales. Really it should be Zipf's Observation, or Zipf's Distribution.

(\*\*) A commenter points out that of course no real data sets actually can follow Zipf's Law if the number of items is reasonably large because then the fractions do not add up to 100%\! In fact 1/2 + 1/3 + 1/4 + 1/5 ... and so on diverges; it goes to infinity. Proving this is easy. Clearly 1/4 + 1/4 + 1/8 + 1/8 + 1/8 + 1/8 + 1/16 + 1/16 + 1/16 + 1/16 + 1/16 + 1/16 + 1/16 + 1/16 + 1/32 ... is less than 1/2 + 1/3 + 1/4 + 1/5 ...  But equally obvious is that 2 \* (1/4) + 4 \* (1/8) + 8 \* (1/16) + 16 \* (1/32) ... is 1/2 + 1/2 + 1/2 + 1/2... which diverges. Zipf's Law data sets do not actually strictly conform to the exact hyperbola shape; they cannot. The point is that Zipf's Law gives a good quick estimate of order of magnitude, not an exact result.

