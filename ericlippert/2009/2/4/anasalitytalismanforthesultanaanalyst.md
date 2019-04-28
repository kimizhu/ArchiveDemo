# A nasality talisman for the sultana analyst

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/4/2009 4:52:48 PM

-----

The other day my charming wife Leah and I were playing *Scrabble Brand Crossword Game* (a registered trademark of Hasbro and Mattel) as is our wont. I went first, drawing the Q and a bunch of vowels. Knowing that the Q is death to hold onto, I immediately opened with QI for 22 points. I silently thanked Miriam-Webster for adding QI, KI and ZA to the OSPD 4th edition.

[![BigSnit](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/ericlippert/WindowsLiveWriter/Anasalitytalismanforthesultanaanalyst_C325/BigSnit_3.jpg)](http://en.wikipedia.org/wiki/The_Big_Snit)

My thankfulness was short-lived, as Leah thought for a few moments and then played ANALYST, for the fifty point "bingo" bonus, making QIS for good measure along the way. She then went on to thoroughly cream me. I am not very good at Scrabble.

We were wondering afterwards what all the possible seven and eight letter "bingo" words were that she could have made with the rack AALNST?. (A blank is conventionally written as "?".) I stared at it for a few moments and found SULTANA and SEALANT, but I was suspicious that there were a lot more. So I wrote a program to find out, which I shall share with you now.

(An interesting historical note: solving this problem efficiently was one of the questions I was given during my interviews when I first applied for full-time work at Microsoft; if you want to get a job here, you might need to know this\!)

The core of the program is a method SearchDictionary which takes a rack string and returns a sequence containing every word in a dictionary file which can be formed using **all** the letters in that rack. In this case I wanted to know not just what all the words matching the given rack were, but what all the words matching the given rack that used an existing letter on the board were. In this case the only two letters on the board were Q and I, but let's assume that it could have been any letter on the board.

The main loop of my little console program looks like this:

 

public static void Main()  
{  
    while (true)  
    {  
        Console.Write("Enter rack (use '?' for blank): ");  
        string rack = Console.ReadLine();  
        if (rack == "")  
            break;  
        Console.WriteLine("{0} : {1}", rack, SearchDictionary(rack).Join());  
        foreach (char c in "ABCDEFGHIJKLMNOPQRSTUVWXYZ")  
            Console.WriteLine("{0}+{1} : {2}", rack, c, SearchDictionary(rack + c).Join());  
    }  
}

Pretty straightforward so far. Of course, this is certainly not production quality code. This is a little utility that I hacked together for myself. A production-quality version would have a lot more error handling, for one thing.

The Join method is just a handy helper function that sticks a sequence of strings together:

 

private static string Join(this IEnumerable\<string\> strs)  
{  
    return string.Join(" ", strs.ToArray());  
}

The trick to efficiently searching for anagrams in a dictionary is to realize that all anagrams have the same letters, just in different order. If you "canonicalize" each word so that its letters are uppercase and in alphabetical order, then checking whether one word is an anagram of another is as simple as comparing their canonical forms:  

private static string Canonicalize(string s)  
{  
    char\[\] chars = s.ToUpperInvariant().ToCharArray();  
    Array.Sort(chars);  
    return new string(chars);  
}

I had a performance goal for this project; I wanted to be able to use the ~2MB 2006 Tournament Word List, searching for racks with up to two blanks, in reasonable human scale time, but not necessarily appearing to be instantaneous. My first naive implementation did not meet this goal so I made a few tweaks to the algorithm until it did, and then I stopped. (It is interesting to think about how this could be made much faster, but that's a subject for another day.)  

private static IEnumerable\<string\> SearchDictionary(string originalRack)  
{  
    const string dictionary = @"d:\\twl06.txt";

    // Calculate all the possible distinct values for the rack.  
    // As an optimization, stuff the resulting racks in an array so  
    // that we do not recalculate them during the query.

    var racks = (from rack in ReplaceQuestionMarks(originalRack)  
                 select Canonicalize(rack)).Distinct().ToArray();

    // Check every line in the dictionary to see if it matches  
    // any possible rack.  As an optimization, do an early  
    // out if the line length does not match the query length.

    return from line in FileLines(dictionary)  
           where line.Length == originalRack.Length  
           where racks.Contains(Canonicalize(line))  
           select line;  
}

LINQ queries are awesome. I love how the code reads like a description of what I'm trying to do, rather than how I'm doing it.

We need a way to turn a rack that might contain blanks into a sequence of the 26 or 26x26 possible racks without blanks. Here's a handy recursive method that does so:

 

private static IEnumerable\<string\> ReplaceQuestionMarks(string s)  
{  
    int index = s.IndexOf('?');  
    if (index == -1)  
    {  
        yield return s;  
        yield break;  
    }  
    foreach (char c in "ABCDEFGHIJKLMNOPQRSTUVWXYZ")  
    {  
        string s2 = s.Substring(0, index) + c.ToString() + s.Substring(index + 1);  
        foreach (string result in ReplaceQuestionMarks(s2))  
            yield return result;  
    }  
}

And of course, the code to extract the lines from the dictionary is our [old friend](http://blogs.msdn.com/ericlippert/archive/2008/09/08/high-maintenance.aspx%20):  

private static IEnumerable\<string\> FileLines(string filename)  
{  
    using (var sr = File.OpenText(filename))  
    {  
        while (true)  
        {  
            string line = sr.ReadLine();  
            if (line == null)  
                yield break;  
            yield return line;  
        }  
    }  
}

And there you go. Six very brief little methods that tell you that Leah could have made the seven letter bingos ANALYST CANTALS LATINAS PLATANS SALTANT SALTPAN SEALANT and SULTANA or by going through the "I", the eight letter bingos ALATIONS ANNALIST FANTAILS LANITALS NASALITY PLATINAS SANTALIC STAMINAL TAILFANS TALISMAN and VALIANTS.

