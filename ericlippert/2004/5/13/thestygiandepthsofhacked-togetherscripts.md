# The Stygian Depths Of Hacked-Together Scripts

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/13/2004 4:48:00 PM

-----

[Last time](http://blogs.msdn.com/ericlippert/archive/2004/05/12/130840.aspx)on FABULOUS ADVENTURES: Eric Lippert: The second term is asymptotically insignificant compared to the first, so we'll throw it away, and declare that any sort on a list of n items has at least a worst case of O(n log n) comparisons.   Leo McGarry: We must inform the President\! And now, the thrilling conclusion. To extract and analyze the Google queries from my logs, I used a bunch of one-off scripts, egregious hacks, and programs written in a whole bunch of different languages. The one-off script has little in common with the ship-it-to-customers code I write every day.  One-off scripts do not need to be readable, testable, performant, robust, localizable, secure, extensible or even bug-free.  The one-off script does a job once, and then I might delete it or toss it in my "scratch" directory or whatever.  As long as it gets the job done, everything is fine. 

 First, let's pull down every referrer page and use a regular expression to parse out the query string. 

 Set re = New RegExp  
re.Pattern = "a href=""http:\\/\\/\[^\>\]\*google\[^\>\]\*q=(\[^&""\<\]\*)\[&""\<\]"  
re.Global = True  
Set HTTP = CreateObject("Microsoft.XMLHTTP")  
URL = <http://blogs.msdn.com/ericlippert/admin/Referrers.aspx?pg=>  
For page = 1 to 2193  
    HTTP.Open "GET", URL & page, False  
    HTTP.Send  
    Set matches = Re.Execute(HTTP.ResponseText)  
    If Not Matches Is Nothing Then  
        For Each Match in Matches  
            For Each SubMatch In Match.SubMatches  
                WScript.Echo SubMatch  
            Next  
        Next  
    End If  
Next   

 I used VBScript because I prefer VBScript's syntax for parsing out submatches to JScript's.  See the stuff in the parens in the expression?  That's the *submatch* that I want to extract -- the actual query portion of the URL. 

 Notice that I have the number of pages hard-coded.  I knew how many pages there were, and therefore why would I bother writing logic to deduce that?  *This is a one-off*.  Don't like it?  Fix it yourself, buddy. 

 I ran that program -- which took quite a while, because downloading 2193 web pages isn't cheap -- and dumped the text to a file.  I now had a file that looked like this: 

 dllmain  
determine+character+encoding  
how+to+use+loadpicture+in+asp  
what+is+garbage+collector+exactly+works+in+.net  
jscript+innertext  
wscript+object+in+asp.net  
vbscript+round+up  
wshcontroller+activex  
closures+jscript.net  
c++++smart+pointers 

 etc.  I was finding all those plus-for-space characters hard to read, so I loaded that thing into my editor and did a little vi magic to first replace all instances of c++ with CPLUSPLUS, then turned every + into a space, and then turned the CPLUSPLUSes back to c++.  That gave me 

 dllmain  
determine character encoding  
how to use loadpicture in asp  
what is garbage collector exactly works in .net  
jscript innertext  
wscript object in asp.net  
vbscript round up  
wshcontroller activex  
closures jscript.net  
c++  smart pointers 

 Then I used my editor to search for the entries that started with "how", "what", and so on, because I figured that the questions would be the funny ones. 

 I was also interested in things like "what's are the top ten most popular words in these queries?", just out of curiosity.  Mike wrote a [blog entry](http://www.mikepope.com/blog/DisplayBlog.aspx?permalink=631 "http://www.mikepope.com/blog/DisplayBlog.aspx?permalink=631") about that a while back, where he deduced from first principles how to solve the "what's the most popular word in this text?" problem.      Let's break that problem down into three subproblems: (1)     Obtain a list of words.  
(2)     Figure out the frequency of each word.  
(3)     Output the words, sorted by frequency.  Suppose that there are n words in the list obtained from (1).  Recall that yesterday we came up with an O(n<sup>2</sup>) solution to part (2) -- for each word in the list, compare it to every word that comes after in the list.  If we added a counter, we could determine how many collisions each word had.  To solve part (3), we could create a new list of (word, count) pairs, and then sort that list by frequency, somehow. 

 Mike came up with a much better solution than that for part (2).  Mike's solution was to **first sort the word list**.  Suppose you have the word list \<the, cat, in, the, hat\>.  Sorting it takes O(n log n) steps, as we learned yesterday: \<cat, hat, in, the, the\>  and suddenly finding the frequency of each word is easy.  You no longer need to compare each word to ALL the words that come after it, just the first few.  The sort is expensive but it made finding and counting the repeated words very cheap.   This is also a good solution because it uses off-the-shelf parts; it uses a built-in sort routine to do the heavy lifting. 

 How then do you solve part (3)?  Mike again had the clever idea of re-using existing parts.  Dump the word and frequency data into a DataTable object, and use the DataTable's sorting methods to sort by frequency.  Another solution that he came up with [later](http://www.mikepope.com/blog/DisplayBlog.aspx?permalink=646 "http://www.mikepope.com/blog/DisplayBlog.aspx?permalink=646") was to implement IComparable on an object that stores the (word, frequency) pairs and sorts on frequency. 

 Can we do better than O(n log n)?  Let's try.  And, as an added bonus, I'm going to use the new generic collection feature of C\#, coming in Whidbey.  If you haven't seen this feature before, it's pretty cool.  To briefly summarize, you can now declare types like "*a dictionary which maps integers onto lists of strings*".  The syntax is straightforward, you'll pick it up from context pretty easily I'm sure. 

 Step one, read in the log and tokenize it using the convenient split method. 

 StreamReader streamReader = new StreamReader(@"c:\\google.log");  
string source = streamReader.ReadToEnd();  
char\[\] punctuation = new char\[\] {' ', '\\n', '\\r', '\\t', '(', ')', '"'};  
string\[\] tokens = source.ToLower().Split(punctuation, true); 

 The true means "don't include any empty items" -- if we were splitting a string with two spaces in a row, for example, there would be an "empty" string between the two delimiters.  (In the next version of the framework, this method will take an enumerated type to choose options rather than this rather opaque Boolean.) 

 We've solved subproblem (1).  Onto (2): 

 Dictionary\<string, int\> firstPass = new Dictionary\<string, int\>();  
foreach(string token in tokens)  
{  
    if (\!firstPass.ContainsKey(token))  
        firstPass\[token\] = 1;  
    else  
        ++firstPass\[token\];  
} 

 On my first pass through the list of tokens, I look in the first pass table to determine if I've seen this token before.  If I have, then I increase its frequency count by one, otherwise I add it to the table with a frequency count of one. This loop is  O(n) because lookups and insertions on dictionary objects are of constant cost no matter how many items are already in the dictionary.  We've solved subproblem (2).   On to subproblem (3).  Now I'm going to take my mapping from words to frequency and reverse it -- turn it into a mapping from frequency to a list of words that have that frequency.  I'm also going to keep track of the highest frequency item I've seen so far, because that will be convenient later. 

 As you can see, this uses the same trick as before.  If the frequency is already in the table then add the current word to the word list, otherwise create a new word list. 

 int max = 1;  
Dictionary\<int, List\<string\>\> secondPass = new Dictionary\<int, List\<string\>\>();  
foreach(string key in firstPass.Keys)  
{  
    int count = firstPass\[key\];  
    if (count \> max)  
        max = count;  
    if (\!secondPass.ContainsKey(count))  
        secondPass\[count\] = new List\<string\>();  
    secondPass\[count\].Add(key);  
} 

 Clearly the number of unique words in the firstPass table cannot be larger than n, so this loop is also O(n).  Inserting items into tables and onto the end of lists is of constant cost, so there's no additional asymptotic load there. 

 OK, great, but how does this help to solve subproblem (3)?  We need one more loop. 

 for (int i = max ; i \>= 1 ; --i)  
{  
    if (secondPass.ContainsKey(i))  
    {  
        Console.WriteLine(i);  
        foreach(string s in secondPass\[i\])  
        {  
            Console.WriteLine(s);  
        }  
    }  
} 

 Again, this straightforward loop cannot possibly print out more than n items and all the list and table operations take constant time, so this is O(n). The algorithm spits out the list 

 3362  
vbscript  
2253  
jscript  
990  
object  
910  
in  
879  
array  
\[...\]  
1  
\[...\]  
forward  
xbox  
honest 

 **Three loops, none** is ever worse than O(n). Therefore, the whole algorithm is O(n). As you can see, I've sorted the Google query keywords by frequency using a worst-case O(n) algorithm, thereby providing a **counterexample** to my proof from last time that such a sort algorithm is **impossible**.  Neat trick, proving a thing and then finding a counterexample\!  How'd I manage to do that? 

 **I've pulled a bait-and-switch.**  What I proved yesterday was that any sort on a list **where the only way to deduce the ordering requires data comparison** is an O(n log n) problem.  But nowhere in this algorithm do I make so much as a single data comparison\!  The internals of the table lookups do data comparisons to determine **equality**, but not **ordering**, and the comparisons to determine max and iterate the last loop are not  comparisons between two data, so they don't count.  

 This sort algorithm *doesn't follow our generalized sort schema at all*, so clearly the proof doesn't apply. 

 I can get away with not making any comparisons because I know additional facts about the data -- facts which I did not assume in my proof.  In this case, the relevant fact is that the thing which I am sorting -- the list of frequencies -- is a list of **n** **integers all of which are between 1 and ****n.**  If you can assume that then as we've just seen, you don't have to use comparisons at all.  This algorithm is called **CountingSort** (for fairly obvious reasons). Another algorithm that has similar properties is **RadixSort**, to which a commenter alluded yesterday.  

 If you're sorting lists of arbitrary strings or arbitrary floating point numbers where you *cannot* make any assumptions about the data then you'll end up using some version of the generalized comparison sort from yesterday. 

 The moral of the story (aside from "watch out for bait-and-switch schemes") is that hash tables, particularly generic hash tables, are incredibly useful off-the-shelf parts for a huge number of applications.  We use them all over the place in our code because they're **easy to use and extremely efficient** even when you throw lots of data at them. 

But wait, something should be nagging at you.  There is something missing from this analysis.  Saying that this is an O(n) algorithm perhaps isn't quite the whole story.  What did I leave out?

