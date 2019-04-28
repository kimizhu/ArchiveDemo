# Every Program There Is, Part Nine

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/24/2010 6:24:00 AM

-----

\[This is the final part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/20/every-program-there-is-part-eight.aspx).\]

We seem to have a bit of a performance problem here. We could slap a profiler on it, and normally I’d recommend just that. But in this case, let’s solve this problem by thinking. Suppose we’re trying to work out a problem in our previous grammar, say, S\[6\]. That requires us to work out, among other things, PARENEND\[5\], BRACKETEND\[5\], and so on, each of which requires us to work out S\[4\]. In short, we work out S\[4\] four times for every time we work out S\[6\]. Each one of those works out S\[2\] four times, for a total of sixteen… and perhaps you begin to see where the performance problem lies.

There’s an old saying that doing the same thing twice and expecting different results is the definition of insanity. (Note that this is not *actually* the definition of insanity; it’s just a saying.) But LINQ was designed to work with databases; you might do the same query twice and get different results if the database has been updated, and LINQ does not know that the “database” in this case is a computation that will always give the same results every time. We are pessimistically assuming that the results will be different every time and *massively* re-computing results that we have computed already and discarded.

We can solve this problem by trading memory usage for speed; we’ll work out S\[4\] once, and then the next time we're asked, we’ll re-use the previous result. The first thing to remember is that the result of a LINQ query is an object representing the query, not the results of the query; let’s make sure that we cache the results and not the query, by realizing the results into a list. We can write up a little memoizer: 

private Dictionary\<Tuple\<string, int\>, List\<string\>\> memoizer =  
  new Dictionary\<Tuple\<string, int\>, List\<string\>\>();  
public IEnumerable\<string\> All(string symbol, int substitutions)  
{  
  List\<string\> results;  
  var tuple = Tuple.Create(symbol, substitutions);  
  if (\!memoizer.TryGetValue(tuple, out results))  
  {  
    results = AllCore(symbol, substitutions).ToList();  
    memoizer.Add(tuple, results);  
  }  
  return results;  
}  
private IEnumerable\<string\> AllCore(string symbol, int substitutions)  
{  
  // same body as before.  
}

and now we’ve traded a speed problem for a memory problem. :-)

But this is good enough. I now have a device I can use to generate *millions* of syntactically valid test cases for any part of the compiler I choose to test. For example, here are a few of the lines of output from an enumeration of the simplified class declaration grammar I mentioned last time:

 

public class a { }  
public class b : a { }  
public class a \< a \> { }  
public class c \< a , a \> { }

Pretty slick\!

Well, that took us much farther than I expected to go when I started writing about generating all the binary trees a couple months ago. I hope you enjoyed this little trip into elementary computational linguistics as much as I did.

\[This is the final part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/20/every-program-there-is-part-eight.aspx).\]

