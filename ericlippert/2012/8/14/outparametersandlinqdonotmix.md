# Out parameters and LINQ do not mix

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 8/14/2012 12:05:19 PM

-----

I am back from my annual vacation in beautiful southwestern Ontario; before I get into the subject of today's post, check out this shot I took with my Windows Phone camera from the plane on the trip home. We are at 37000 feet, just outside of Billings, Montana, a few minutes before sunset:

[![WP\_000062](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/3716.WP_000062_thumb.jpg "WP_000062")](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/29/89/metablogapi/0160.WP_000062_2.jpg)

The whole thing was chock full of immense lightning arcs which unfortunately I did not capture in the image. This is certainly the largest isolated thunderstorm I've ever seen from the outside. Notice the characteristic anvil shape; [as I've described before](http://blogs.msdn.com/b/ericlippert/archive/tags/weather/), we've got a huge heat engine here that is extracting the latent heat from the gaseous and liquid water, and then using that heat to power the updraft that sucks more warm water vapor upwards. Quite beautiful.

Well, enough chit-chat about the weather. Today: **what's wrong with this code?**

var seq = new List\<string\> { "1", "blah", "3" };  
int tmp;  
var nums =  
  from item in seq  
  let success = int.TryParse(item, out tmp)  
  select success ? tmp : 0;

The intention is pretty clear: take a sequence of strings and produce a sequence of numbers by turning the elements that can be parsed as numbers into those numbers and the rest into zero.

The C\# compiler will give you a definite assignment error if you try this, which seems strange. Why does it do that? Well, think about what code the compiler will translate the last statement into:

var nums =  
  seq.Select(item=\>new {item, success = int.TryParse(item, out tmp)})  
     .Select(transparent =\> transparent.success ? tmp : 0);

We have two method calls and two lambdas. Clearly the first lambda assigns tmp and the second reads it, but we have no guarantee whatsoever that the first call to Select invokes the lambda\! It could, for some bizarre reason of its own, never invoke the lambda. Since the compiler cannot prove that tmp is *definitely assigned* before it is read, this program is an error.

So is the solution then to assign tmp in the variable declaration? Certainly not\! That makes the program compile, but it is a "worst practice" to mutate a variable like this. Remember, that one variable is going to be shared by every delegate invocation\! In this simple LINQ-to-Objects scenario it is the case that the delegates are invoked in the sensible order, but even a small change makes this nice property go out the window:

int tmp = 0;  
var nums =  
  from item in seq  
  let success = int.TryParse(item, out tmp)  
  orderby item  
  select success ? tmp : 0;  
foreach(var num in nums) { Console.WriteLine(num); }

Now what happens?

We make an object that represents the query. The query object consists of three steps: do the "let" projection, do the sort, and do the final projection. Remember, the query is not executed until the first result from it is requested; the assignment to "nums" just produces an object that represents the query, not the results.

Then we execute the query by entering the body of the loop. Doing so initiates a whole chain of events, but clearly it must be the case that the entire "let" projection is executed from start to finish over the whole sequence in order to get the resulting pairs to be sorted by the "orderby" clause. Executing the "let" projection lambda three times causes "tmp" to be mutated three times. Only after the sort is completed is the final projection executed, and **it uses the current value of tmp**, not the value that tmp was back in the distant past\!

So what is the right thing to do here? The solution is to write your own extension method version of TryParse the way it would have been written had there been nullable value types available in the first place:

static int? MyTryParse(this string item)  
{  
    int tmp;  
    bool success = int.TryParse(item, out tmp);  
    return success ? (int?)tmp : (int?)null;  
}

And now we can say:

var nums =  
  from item in seq  
  select item.MyTryParse() ?? 0;

The mutation of the variable is now isolated to the activation of the method, rather than a side effect that is observed by the query. **Try to always avoid side effects in queries.**

-----

Thanks to Bill Wagner for the question that inspired this blog entry.

