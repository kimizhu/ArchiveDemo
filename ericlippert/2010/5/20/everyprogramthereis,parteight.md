# Every Program There Is, Part Eight

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/20/2010 6:21:00 AM

-----

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/17/every-program-there-is-part-seven.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/24/every-program-there-is-part-nine.aspx).\]

OK, first things first. Let’s assume that we have a string that contains a grammar that has already been rewritten into our “nice” form, where there are exactly two terms in every production rule. We can easily write a parser that analyzes such a string. I’ll make a dictionary that maps each rule onto a list of two-symbol productions. So the grammar:

S: N NIL | S P  
P: + N  
N: 1 NIL | 2 NIL | 3 NIL

is turned into the dictionary: S –\> { (N, NIL), (S, P) }  
P –\> { (+, N) }  
N –\> { (1, NIL), (2, NIL), (3 NIL) }

Here we go:

 

sealed class Grammar  
{  
    private Dictionary\<string, List\<Tuple\<string, string\>\>\> rules;  
    public Grammar(string g)  
    {  
        rules = new Dictionary\<string, List\<Tuple\<string, string\>\>\>();  
        foreach (string s in g.Split(new\[\] { '\\n', '\\r' }, StringSplitOptions.RemoveEmptyEntries))  
        {  
            int i = s.IndexOf(':');  
            var name = s.Substring(0, i).Trim();  
            var list = new List\<Tuple\<string, string\>\>();  
            foreach (var o in s.Substring(i + 1).Split('|'))  
            {  
                var words = o.Split(new\[\] { ' ' }, StringSplitOptions.RemoveEmptyEntries);  
                Debug.Assert(words.Length == 2);  
                list.Add(Tuple.Create(words\[0\], words\[1\]));  
            }  
            rules.Add(name, list);  
        }  
    }

OK, super.

Now we need a way of saying “given a symbol and the exact number of substitutions to perform on it, give all the results that are strings of terminals”. We can make a simple modification to our binary tree enumerator.

First we’ll handle the trivial cases: if we are asked to make zero substitutions then there are three cases. If we have a nonterminal then there are no results because we need to make at least one more substitution. If we have a regular terminal, we'll return it. And if we have NIL, remember, it is a special nonterminal that is identical to the empty string.

If we have substitutions to do then either we have a terminal or we have a nonterminal. If we have a terminal then there are no results because you cannot substitute on a terminal. If we have substitutions to do on a nonterminal then clearly we’re not in a base case and we need to recurse.

 

private static string\[\] empty = { };  
public IEnumerable\<string\> All(string symbol, int substitutions)  
{  
    bool terminal = \!rules.ContainsKey(symbol);  
    if (substitutions == 0)  
    {  
        // No more substitutions; we'd better have a terminal.  
        if (terminal)  
            return new string\[\] { symbol == "NIL" ? "" : symbol + " " };  
        return empty;  
    }     // We're doing substitutions; we'd better have a nonterminal.  
    if (terminal)  
        return empty;     return from r in rules\[symbol\]  
            from i in Enumerable.Range(0, substitutions)  
            from left in All(r.Item1, i)  
            from right in All(r.Item2, substitutions - i - 1)  
            select left + right;  
}

Let’s test it out. Here’s a two-symbol-per-production grammar for the language “all properly matched combinations of parens, brackets, braces and angles:

 

const string parengrammar = @"  
S:           S S | ( PARENEND | \[ BRACKETEND | { BRACEEND | \< ANGLEEND  
PARENEND:    S ) | NIL )  
BRACKETEND:  S \] | NIL \]  
BRACEEND:    S } | NIL }  
ANGLEEND:    S \> | NIL \>  
";

static void Main()  
{  
    Grammar g = new Grammar(parengrammar);  
    for (int i = 1; i \< 5; ++i)  
    {  
        Console.WriteLine(i);  
        foreach (var s in g.All("S", i))  
            Console.WriteLine(s);  
    }  
}

And sure enough, the output is:

 

1  
2  
( )  
\[ \]  
{ }  
\< \>  
3  
4  
( ( ) )  
( \[ \] )  
( { } )  
( \< \> )  
\[ ( ) \]  
\[ \[ \] \]  
\[ { } \]  
\[ \< \> \]  
{ ( ) }  
{ \[ \] }  
{ { } }  
{ \< \> }  
\< ( ) \>  
\< \[ \] \>  
\< { } \>  
\< \< \> \>

Notice that these are not all the *four-character strings*; these are the strings that require *four substitutions*. ( ) ( ) isn’t on the list because that requires *five* substitutions in its derivation.

Try that program but going up to eleven, twelve, thirteen or more substitutions. Notice anything odd? Sure, obviously there are more and more strings to generate for each one. But more to the point, things seem to be getting slower… and slower… and slower.

**Next time:** Insanity\!

\[This is part of a series on generating every string in a language. The previous part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/17/every-program-there-is-part-seven.aspx). The next part is [here](http://blogs.msdn.com/b/ericlippert/archive/2010/05/24/every-program-there-is-part-nine.aspx).\]

