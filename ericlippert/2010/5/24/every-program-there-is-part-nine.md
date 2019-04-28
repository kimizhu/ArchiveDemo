<div id="page">

# Every Program There Is, Part Nine

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/24/2010 6:24:00 AM

-----

<div id="content">

<div class="mine">

\[This is the final part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/20/every-program-there-is-part-eight.aspx).\]

We seem to have a bit of a performance problem here. We could slap a profiler on it, and normally I’d recommend just that. But in this case, let’s solve this problem by thinking. Suppose we’re trying to work out a problem in our previous grammar, say, <span class="code">S\[6\]</span>. That requires us to work out, among other things, <span class="code">PARENEND\[5\]</span>, <span class="code">BRACKETEND\[5\]</span>, and so on, each of which requires us to work out <span class="code">S\[4\]</span>. In short, we work out <span class="code">S\[4\]</span> four times for every time we work out <span class="code">S\[6\]</span>. Each one of those works out <span class="code">S\[2\]</span> four times, for a total of sixteen… and perhaps you begin to see where the performance problem lies.

There’s an old saying that doing the same thing twice and expecting different results is the definition of insanity. (Note that this is not *actually* the definition of insanity; it’s just a saying.) But LINQ was designed to work with databases; you might do the same query twice and get different results if the database has been updated, and LINQ does not know that the “database” in this case is a computation that will always give the same results every time. We are pessimistically assuming that the results will be different every time and *massively* re-computing results that we have computed already and discarded.

We can solve this problem by trading memory usage for speed; we’ll work out <span class="code">S\[4\]</span> once, and then the next time we're asked, we’ll re-use the previous result. The first thing to remember is that the result of a LINQ query is an object representing the query, not the results of the query; let’s make sure that we cache the results and not the query, by realizing the results into a list. We can write up a little memoizer:<span class="code"><span style="font-family: consolas; font-size: x-small"><span style="font-family: consolas; font-size: x-small"></span></span> </span>

<span style="font-size: small"><span style="font-family: courier new,courier"><span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">private</span></span></span> <span style="color: #2b91af"><span style="color: #2b91af"><span style="color: #2b91af">Dictionary</span></span></span>\<<span style="color: #2b91af"><span style="color: #2b91af"><span style="color: #2b91af">Tuple</span></span></span>\<<span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">string</span></span></span>, <span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">int</span></span></span>\>, <span style="color: #2b91af"><span style="color: #2b91af"><span style="color: #2b91af">List</span></span></span>\<<span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">string</span></span></span></span><span style="font-family: consolas"><span style="font-family: courier new,courier">\>\> memoizer =  
  </span></span></span><span style="font-size: small"><span style="font-family: courier new,courier"><span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">new</span></span></span> <span style="color: #2b91af"><span style="color: #2b91af"><span style="color: #2b91af">Dictionary</span></span></span>\<<span style="color: #2b91af"><span style="color: #2b91af"><span style="color: #2b91af">Tuple</span></span></span>\<<span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">string</span></span></span>, <span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">int</span></span></span>\>, <span style="color: #2b91af"><span style="color: #2b91af"><span style="color: #2b91af">List</span></span></span>\<<span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">string</span></span></span></span></span><span style="font-size: small"><span style="font-family: courier new,courier">\>\>();  
<span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">public</span></span></span> <span style="color: #2b91af"><span style="color: #2b91af"><span style="color: #2b91af">IEnumerable</span></span></span>\<<span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">string</span></span></span>\> All(<span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">string</span></span></span> symbol, <span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">int</span></span></span></span></span><span style="font-size: small"><span style="font-family: courier new,courier"> substitutions)  
{  
  <span style="color: #2b91af"><span style="color: #2b91af"><span style="color: #2b91af">List</span></span></span>\<<span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">string</span></span></span></span></span><span style="font-size: small"><span style="font-family: courier new,courier">\> results;  
  <span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">var</span></span></span> tuple = <span style="color: #2b91af"><span style="color: #2b91af"><span style="color: #2b91af">Tuple</span></span></span></span></span><span style="font-size: small"><span style="font-family: courier new,courier">.Create(symbol, substitutions);  
  <span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">if</span></span></span> (\!memoizer.TryGetValue(tuple, <span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">out</span></span></span></span></span><span style="font-size: small"><span style="font-family: courier new,courier"> results))  
  {  
    results = AllCore(symbol, substitutions).ToList();  
    memoizer.Add(tuple, results);  
  }  
  <span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">return</span></span></span></span></span><span style="font-size: small"><span style="font-family: courier new,courier"> results;  
}  
<span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">private</span></span></span> <span style="color: #2b91af"><span style="color: #2b91af"><span style="color: #2b91af">IEnumerable</span></span></span>\<<span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">string</span></span></span>\> AllCore(<span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">string</span></span></span> symbol, <span style="color: #0000ff"><span style="color: #0000ff"><span style="color: #0000ff">int</span></span></span></span></span><span style="font-family: consolas; font-size: x-small"><span style="font-family: consolas"><span style="font-size: small"><span style="font-family: courier new,courier"> substitutions)  
{  
</span></span></span></span><span style="font-size: small"><span style="font-family: courier new,courier"><span style="color: #008000"><span style="color: #008000"><span style="color: #008000">  // same body as before.  
</span></span></span>}</span></span>

and now we’ve traded a speed problem for a memory problem. :-)

But this is good enough. I now have a device I can use to generate *millions* of syntactically valid test cases for any part of the compiler I choose to test. For example, here are a few of the lines of output from an enumeration of the simplified class declaration grammar I mentioned last time:

<span class="code"> </span>

public class a { }  
public class b : a { }  
public class a \< a \> { }  
public class c \< a , a \> { }

Pretty slick\!

Well, that took us much farther than I expected to go when I started writing about generating all the binary trees a couple months ago. I hope you enjoyed this little trip into elementary computational linguistics as much as I did.

\[This is the final part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/20/every-program-there-is-part-eight.aspx).\]

</div>

</div>

</div>

