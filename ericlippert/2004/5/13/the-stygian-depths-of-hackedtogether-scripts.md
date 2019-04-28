<div id="page">

# The Stygian Depths Of Hacked-Together Scripts

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/13/2004 4:48:00 PM

-----

<div id="content">

<span>[Last time](http://blogs.msdn.com/ericlippert/archive/2004/05/12/130840.aspx) on FABULOUS ADVENTURES:</span> <span>Eric Lippert: The second term is asymptotically insignificant compared to the first, so we'll throw it away, and declare that any sort on a list of <span>n</span><span> items has at least </span></span><span><span>a worst case of </span><span>O(n log n)</span><span> comparisons.  </span></span> <span><span>Leo McGarry: We must inform the President\!</span></span> <span>And now, the thrilling conclusion.</span> <span>To extract and analyze the Google queries from my logs, I used a bunch of one-off scripts, egregious hacks, and programs written in a whole bunch of different languages. The one-off script has little in common with the ship-it-to-customers code I write every day.  One-off scripts do not need to be readable, testable, performant, robust, localizable, secure, extensible or even bug-free.  The one-off script does a job once, and then I might delete it or toss it in my "scratch" directory or whatever.  As long as it gets the job done, everything is fine. </span>

<span></span> <span>First, let's pull down every referrer page and use a regular expression to parse out the query string. </span>

<span></span> <span>Set re = New RegExp  
</span><span>re.Pattern = "a href=""http:\\/\\/\[^\>\]\*google\[^\>\]\*q=(\[^&""\<\]\*)\[&""\<\]"  
</span><span>re.Global = True  
</span><span>Set HTTP = CreateObject("Microsoft.XMLHTTP")  
</span><span>URL = <http://blogs.msdn.com/ericlippert/admin/Referrers.aspx?pg=>  
</span><span>For page = 1 to 2193  
</span><span>    HTTP.Open "GET", URL & page, False  
</span><span>    HTTP.Send  
</span><span>    Set matches = Re.Execute(HTTP.ResponseText)  
</span><span>    If Not Matches Is Nothing Then  
</span><span>        For Each Match in Matches  
</span><span>            For Each SubMatch In Match.SubMatches  
</span><span>                WScript.Echo SubMatch  
</span><span>            Next  
</span><span>        Next  
</span><span>    End If  
</span><span>Next   </span>

<span></span> <span>I used VBScript because I prefer VBScript's syntax for parsing out submatches to JScript's.  See the stuff in the parens in the expression?  That's the *<span>submatch</span>* that I want to extract -- the actual query portion of the URL. </span>

<span></span> <span>Notice that I have the number of pages hard-coded.  I knew how many pages there were, and therefore why would I bother writing logic to deduce that?  *This is a one-off*.  Don't like it?  Fix it yourself, buddy. </span>

<span></span> <span>I ran that program -- which took quite a while, because downloading 2193 web pages isn't cheap -- and dumped the text to a file.  I now had a file that looked like this: </span>

<span></span> <span>dllmain  
</span><span>determine+character+encoding  
</span><span>how+to+use+loadpicture+in+asp  
</span><span>what+is+garbage+collector+exactly+works+in+.net  
</span><span>jscript+innertext  
</span><span>wscript+object+in+asp.net  
</span><span>vbscript+round+up  
</span><span>wshcontroller+activex  
</span><span>closures+jscript.net  
</span><span>c++++smart+pointers </span>

<span></span> <span>etc.  I was finding all those plus-for-space characters hard to read, so I loaded that thing into my editor and did a little vi magic to first replace all instances of c++ with CPLUSPLUS, then turned every + into a space, and then turned the CPLUSPLUSes back to c++.  That gave me </span>

<span></span> <span>dllmain  
</span><span>determine character encoding  
</span><span>how to use loadpicture in asp  
</span><span>what is garbage collector exactly works in .net  
</span><span>jscript innertext  
</span><span>wscript object in asp.net  
</span><span>vbscript round up  
</span><span>wshcontroller activex  
</span><span>closures jscript.net  
</span><span>c++  smart pointers </span>

<span></span> <span>Then I used my editor to search for the entries that started with "how", "what", and so on, because I figured that the questions would be the funny ones. </span>

<span></span> <span>I was also interested in things like "what's are the top ten most popular words in these queries?", just out of curiosity.  Mike wrote a [blog entry](http://www.mikepope.com/blog/DisplayBlog.aspx?permalink=631 "http://www.mikepope.com/blog/DisplayBlog.aspx?permalink=631") about that a while back, where he deduced from first principles how to solve the "what's the most popular word in this text?" problem.  </span> <span> </span> <span></span> <span><span>Let's break that problem down into three subproblems:</span></span> <span><span>(1)<span>     </span></span></span><span>Obtain a list of words.  
</span><span><span>(2)<span>     </span></span></span><span>Figure out the frequency of each word.  
</span><span><span>(3)<span>     </span></span></span><span>Output the words, sorted by frequency.</span> <span></span> <span>Suppose that there are </span><span>n</span><span> words in the list obtained from (1).  Recall that yesterday we came up with an </span><span>O(n<sup>2</sup>)</span><span> solution to part (2) -- for each word in the list, compare it to every word that comes after in the list.  If we added a counter, we could determine how many collisions each word had.  To solve part (3), we could create a new list of (word, count) pairs, and then sort that list by frequency, somehow. </span>

<span></span> <span>Mike came up with a much better solution than that for part (2).  Mike's solution was to **first sort the word list**.  Suppose you have the word list \<the, cat, in, the, hat\>.  Sorting it takes <span>O(n log n) </span>steps, as we learned yesterday: \<cat, hat, in, the, the\>  and suddenly finding the frequency of each word is easy.  You no longer need to compare each word to ALL the words that come after it, just the first few.  T</span><span>he sort is expensive but it made finding and counting the repeated words very cheap.  </span> <span>This is also a good solution because it uses off-the-shelf parts; it uses a built-in sort routine to do the heavy lifting. </span>

<span></span> <span>How then do you solve part (3)?  Mike again had the clever idea of re-using existing parts.  Dump the word and frequency data into a DataTable object, and use the DataTable's sorting methods to sort by frequency.  Another solution that he came up with [later](http://www.mikepope.com/blog/DisplayBlog.aspx?permalink=646 "http://www.mikepope.com/blog/DisplayBlog.aspx?permalink=646") was to implement </span><span>IComparable</span><span> on an object that stores the (word, frequency) pairs and sorts on frequency. </span>

<span></span> <span>Can we do better than </span><span>O(n log n)</span><span>?  Let's try.  And, as an added bonus, I'm going to use the new generic collection feature of C\#, coming in Whidbey.  If you haven't seen this feature before, it's pretty cool.  To briefly summarize, you can now declare types like "*<span>a dictionary which maps integers onto lists of strings</span>*".  The syntax is straightforward, you'll pick it up from context pretty easily I'm sure. </span>

<span></span> <span>Step one, read in the log and tokenize it using the convenient </span><span>split</span><span> method. </span>

<span></span> <span>StreamReader streamReader = new StreamReader(@"c:\\google.log");  
</span><span>string source = streamReader.ReadToEnd();  
</span><span>char\[\] punctuation = new char\[\] {' ', '\\n', '\\r', '\\t', '(', ')', '"'};  
</span><span>string\[\] tokens = source.ToLower().Split(punctuation, true); </span>

<span></span> <span>The true means "don't include any empty items" -- if we were splitting a string with two spaces in a row, for example, there would be an "empty" string between the two delimiters.  (In the next version of the framework, this method will take an enumerated type to choose options rather than this rather opaque Boolean.) </span>

<span></span> <span>We've solved subproblem (1).  Onto (2): </span>

<span></span> <span>Dictionary\<string, int\> firstPass = new Dictionary\<string, int\>();  
</span><span>foreach(string token in tokens)  
</span><span>{  
</span><span>    if (\!firstPass.ContainsKey(token))  
</span><span>        firstPass\[token\] = 1;  
</span><span>    else  
</span><span>        ++firstPass\[token\];  
</span><span>} </span>

<span></span> <span>On my first pass through the list of tokens, I look in the first pass table to determine if I've seen this token before.  If I have, then I increase its frequency count by one, otherwise I add it to the table with a frequency count of one. This loop is  </span><span>O(n)</span><span> because lookups and insertions on dictionary objects are of constant cost no matter how many items are already in the dictionary.  We've solved subproblem (2).  </span> <span>On to subproblem (3).  Now I'm going to take my mapping from words to frequency and reverse it -- turn it into a mapping from frequency to a list of words that have that frequency.  I'm also going to keep track of the highest frequency item I've seen so far, because that will be convenient later. </span>

<span></span> <span>As you can see, this uses the same trick as before.  If the frequency is already in the table then add the current word to the word list, otherwise create a new word list. </span>

<span></span> <span>int max = 1;  
</span><span>Dictionary\<int, List\<string\>\> secondPass = new Dictionary\<int, List\<string\>\>();  
</span><span>foreach(string key in firstPass.Keys)  
</span><span>{  
</span><span>    int count = firstPass\[key\];  
</span><span>    if (count \> max)  
</span><span>        max = count;  
</span><span>    if (\!secondPass.ContainsKey(count))  
</span><span>        secondPass\[count\] = new List\<string\>();  
</span><span>    secondPass\[count\].Add(key);  
</span><span>} </span>

<span></span> <span>Clearly the number of unique words in the </span><span>firstPass</span><span> table cannot be larger than </span><span>n</span><span>, so this loop is also </span><span>O(n)</span><span>.  Inserting items into tables and onto the end of lists is of constant cost, so there's no additional asymptotic load there. </span>

<span></span> <span>OK, great, but how does this help to solve subproblem (3)?  We need one more loop. </span>

<span></span> <span>for (int i = max ; i \>= 1 ; --i)  
</span><span>{  
</span><span>    if (secondPass.ContainsKey(i))  
</span><span>    {  
</span><span>        Console.WriteLine(i);  
</span><span>        foreach(string s in secondPass\[i\])  
</span><span>        {  
</span><span>            Console.WriteLine(s);  
</span><span>        }  
</span><span>    }  
</span><span>} </span>

<span></span> <span>Again, this straightforward loop cannot possibly print out more than </span><span>n</span><span> items and all the list and table operations take constant time, so this is </span><span>O(n)</span><span>. The algorithm spits out the list </span>

<span></span> <span>3362  
</span><span>vbscript  
</span><span>2253  
</span><span>jscript  
</span><span>990  
</span><span>object  
</span><span>910  
</span><span>in  
</span><span>879  
</span><span>array  
</span><span>\[...\]  
</span><span>1  
</span><span>\[...\]  
</span><span>forward  
</span><span>xbox  
</span><span>honest </span>

<span></span> **<span>Three loops, none</span>**<span> is ever worse than </span><span>O(n).</span><span> Therefore, the whole algorithm is</span><span> O(n).</span><span> As you can see, I've sorted the Google query keywords by frequency using a worst-case </span><span>O(n) </span><span>algorithm, thereby providing a **<span>counterexample</span>** to my proof from last time that such a sort algorithm is **<span>impossible</span>**.  Neat trick, proving a thing and then finding a counterexample\!  How'd I manage to do that? </span>

<span></span> <span>**I've pulled a bait-and-switch.**  What I proved yesterday was that any sort on a list **<span>where the only way to deduce the ordering requires data comparison</span>** is an </span><span>O(n log n)</span><span> problem.  But nowhere in this algorithm do I make so much as a single data comparison\!  The internals of the table lookups do data comparisons to determine **<span>equality</span>**, but not **<span>ordering</span>**, and the comparisons to determine </span><span>max</span><span> and iterate the last loop are not  comparisons between two data, so they don't count.  </span>

<span></span> <span>This sort algorithm *<span>doesn't follow our generalized sort schema at all</span>*, so clearly the proof doesn't apply. </span>

<span></span> <span>I can get away with not making any comparisons because I know additional facts about the data -- facts which I did not assume in my proof.  In this case, the relevant fact is that the thing which I am sorting -- the list of frequencies -- is a list of <span>**n**</span> **<span>integers all of which are between 1 and </span>**</span>**<span>n</span><span>.</span>**<span>  If you can assume that then as we've just seen, you don't have to use comparisons at all.  This algorithm is called **<span>CountingSort</span>** (for fairly obvious reasons). Another algorithm that has similar properties is **<span>RadixSort</span>**, to which a commenter alluded yesterday.  </span>

<span></span> <span>If you're sorting lists of arbitrary strings or arbitrary floating point numbers where you *<span>cannot</span>* make any assumptions about the data then you'll end up using some version of the generalized comparison sort from yesterday. </span>

<span></span> <span>The moral of the story (aside from "watch out for bait-and-switch schemes") is that hash tables, particularly generic hash tables, are incredibly useful off-the-shelf parts for a huge number of applications.  We use them all over the place in our code because they're **<span>easy to use and extremely efficient</span>** even when you throw lots of data at them. </span>

<span></span>

<span>But wait, something should be nagging at you.  There is something missing from this analysis.  Saying that this is an </span><span>O(n) </span><span>algorithm perhaps isn't quite the whole story.  What did I leave out?</span>

</div>

</div>

