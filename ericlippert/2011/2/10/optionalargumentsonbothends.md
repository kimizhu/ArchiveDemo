# Optional arguments on both ends

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/10/2011 6:54:00 AM

-----

Before we get into today's topic, a quick update on my [posting from last year](http://blogs.msdn.com/b/ericlippert/archive/2010/12/16/hiring-for-roslyn.aspx) about Roslyn jobs. We have gotten a lot of good leads and made some hires but we still have positions open, both on the Roslyn team and on the larger Visual Studio team. [For details, see this post](http://blogs.msdn.com/b/visualstudio/archive/2011/02/08/visual-studio-is-hiring.aspx) on the [Visual Studio blog](http://blogs.msdn.com/b/visualstudio/). Again, please do not send me resumes directly; send them via the usual career site. Thanks\!

-----

Here's a recent question I got from a reader: what is the correct analysis of this little problem involving optional arguments?

 

static void F(string ham, object jam = null) { }  
static void F(string spam, string ham, object jam = null) { }  
...  
F("meat product", null);

The developer of this odd code is apparently attempting to make optional parameters on both ends; the intention is that both **spam** and **jam** are optional, but **ham** is always required.

Which overload does the compiler pick, and why? See if you can figure it out for yourself.

.

.

.

.

.

.

.

.

.

.

.

.

.

The second overload is chosen. (Were you surprised?)

The reason why is straightforward. The overload resolution algorithm has two candidates and must pick which is the best one on the basis of the conversions from the arguments to the formal parameter types. Look at the first argument. Either it is converted to string, or to... string. Hmph. We can conclude nothing from this argument, so we ignore it.

Now consider the second argument. We either convert null to object, if we pick the first overload, or to string, if we pick the second overload. String is obviously more specific than object; every string is an object, but not every object is a string. We strive to pick the overload that is more specific, so we choose the second overload, and call

 

F("meat product", null, null);

The moral of the story is, as I have said so many times, **if it** [**hurts**](http://blogs.msdn.com/b/ericlippert/archive/2008/05/09/computers-are-dumb.aspx) **when** [**you**](http://blogs.msdn.com/b/ericlippert/archive/2008/05/07/covariance-and-contravariance-part-twelve-to-infinity-but-not-beyond.aspx) **do** [**that**](http://stackoverflow.com/questions/3656918/variable-that-cant-be-modified/3660051#3660051) **then** [**don't**](http://blogs.msdn.com/b/ericlippert/archive/2009/12/07/query-transformations-are-syntactic.aspx) **do that\!** Don't try to put an optional argument on both the front and back ends; it's just confusing as all heck, particularly if the types collide as they do here. Write the code as a single method with each optional parameter declaration coming at the end:

 

static void F(string ham, string spam = null, object jam = null) { }

With only one overload it is clear what argument corresponds to what parameter.

Now perhaps you see why we were so reticent to add optional arguments to overload resolution for ten years; it makes the algorithm less intuitive in some cases.

