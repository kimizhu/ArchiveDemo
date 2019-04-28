# Aargh, Part One: A Pirate Walks Into A Bar…

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 3/10/2004 10:20:00 AM

-----

One of my former housemates was fond of pirate jokes,

[as am I](http://blogs.msdn.com/ericlippert/archive/2003/09/19/53054.aspx). My personal favourite of his was:

> A pirate walks into a bar, and the barkeep says "*Excuse me, cap'n, but did you know that you've got your ship's wheel stuck in your pantaloons*?"
> 
> "*Aye*," says the pirate, "*that thing be drivin' me nuts\!  Aaargh\!*"

Today: stuff that drives me nuts, part one of an ongoing series which I will continue whenever I'm too busy to write.  (Complaining is easy.)

I've reviews a lot of code over the years, most of which was very high quality.  But no code is perfect. There are some bad patterns I see over and over again that are easy to fall into but suboptimal in subtle ways.  M

ost of this series will apply specifically to COM programming, but I may throw in a few scripting peeves as well.

I have already griped about

[smart pointers](http://blogs.msdn.com/ericlippert/archive/2003/09/16/53016.aspx) and [bad Hungarian](http://blogs.msdn.com/ericlippert/archive/2003/09/16/53015.aspx), so I'll give them a miss.

Gripe \#1: Too Many Comments

Comments should explain places where the semantics is not obvious from the syntax. Here's some code I found deep in the persistence code of an unnamed Microsoft product:

// Read cItems  

CheckHresult( pStm-\>Read( \&cItems, sizeof(LONG), NULL), pStm, IID\_IStream );  
// Allocate a block which can hold cItems  
pItems = new DWORD\[cItems\];  
// Read the block of items  
CheckHresult( pStm-\>Read(pItems, cItems \* sizeof(LONG), NULL), pStm, IID\_IStream);

Aargh\! It's drivin' me nuts\!

(Aside: Why no error checking on the new? More on that next time.  Also, this reminds me that I want to do a blog on security aspects of serialization code some time.)

Having one comment per line which simply restates the following line in English instead of C++ doesn't help anyone *understand* the code, it just makes the code *longer*.

**

It's worse than that. It *dilutes* the value of comments -- you should be able to look through a piece of code and find all the really interesting bits by looking for the comments. If there are comments everywhere, 90% of which explain nothing that couldn't be gleaned from reading the code itself then the 10% of the useful comments -- the ones that explain the semantics -- will never be noticed.

Furthermore, comments "go bad" -- when you do this habitually, it is really easy to change the code and forget to change the comments. Then the comments no longer accurately describe how the code *is*, but rather how it *was*.

Comments should also be of appropriate size. Here is a fragment of a script I recently code reviewed.

''''''''''''''''''''''''''''''''''''''''''''''''''''''''  
''   DISABLE SCREEN SAVER  
''''''''''''''''''''''''''''''''''''''''''''''''''''''''  
  

WScript.Echo "Disabling Screen Saver"  
\[ then code here that disabled the screen saver \]

OK, I'm reading the code, and I see the line

WScript.Echo "Disabling Screen Saver"

and I can probably guess what the next line of code does without having the **five lines** above it consumed by comments and whitespace\! I want these huge block comments to be catching my eye for really incredibly important stuff. Otherwise, it's just consuming valuable screen real estate, preventing me from getting more semantics-bearing code onto one screen.

Gripe \#2: Bad Macros

The sharp-eyed among you will have wondered about the

CheckHresult call above. What the heck is that thing doing? It's turning HRESULTs into exceptions.  (More on why that's a bad idea later.)

Unfortunately, it's a macro. Here's some code that I once wrote using this macro, from the aforementioned persistence code. I was adding the ability to save the state in a new format. Does there appear to be anything wrong with it? (

LoadOldFileFormat doesn't return an error, it throws exceptions. LoadNewFileFormat returns an HRESULT.)

if (fOldFileFormat)  
  LoadOldFileFormat(pStorage);  
else if (NULL \!= pNewStream)  
  CheckHresult(LoadNewFileFormat(pNewStream), (IPersistStream\*)this, IID\_IPersistStream);  
else  

  // This file is not in any of our formats.  
  Throw(E\_FAIL);

Imagine my surprise to see that the compiler produces a **syntax error** when attempting to compile this code.

I didn't write that macro, so I had no idea what it did -- I was

[cargo-cult programming](http://blogs.msdn.com/ericlippert/archive/2004/03/01/82168.aspx)\! All the other code that returned error codes called this macro, so I figured I should do so as well. Bad developer\! No biscuit\!

I ran that through the C preprocessor, and take a look at what popped out: (I've added whitespace and removed some cruft for clarity.)

if (fOldFileFormat)  
  LoadOldFileFormat(pStorage);  
else if (0 \!= pStream)  
{  
  HRESULT \_hr;   
  if ((\_hr=LoadNewFileFormat(pXMLStream)) \< 0)   
    Throw(\_hr);   
} ;  
else  
  Throw(0x80004005L);

The trailing semi effectively terminates the if and the compiler then balks on the else.

Aaaaaargh\! It's drivin' me nuts\!

If you're going to have macros, don't write them so that they break C++'s lexical semantics in weird, hard-to-track-down ways. Obviously this wasn't *intentional*, which just makes my point even stronger -- **it is very hard to write correct macros, so don't even try.** Don't write macros that look like functions, write **actual inline functions** and let the compiler take care of optimizing them. Inline functions are guaranteed to not have weird lexical side effects that are not apparent from the text.

In this particular case, we have a macro which represents a **statement block** but can be used like a **statement**. The correct thing to do given the definition of this macro is to never terminate it with a semicolon, which looks weird -- no one will be able to remember to do the correct thing\!

Next time, more on **how mixing C++ exception handling and COM programming drives me nuts.**

