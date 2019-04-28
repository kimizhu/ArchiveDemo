<div id="page">

# Google Desktop On The Cheap Part One: Nope, That Doesn't Work

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/20/2004 12:12:00 PM

-----

<div id="content">

So, how many files are we talking about here? 

  - I've got over thirty thousand source code files containing over **seven million** unique words. There are pretty much two kinds of words -- words that occur in one, two, a dozen files, and words that occur in almost every file. ("Microsoft", "next", etc.)
  - An **index** consists of a **searchable list of words**. Each word is mapped to its **postings list** -- the list of all the documents in which the word is found. (A postings list could also contain information about where in the document the word is found, but in general, these documents are so short that it doesn't really matter. If we need to know precisely where the word is, it's fast enough to just search that document again from scratch. We want to make finding the document in the first place cheaper.)
  - Building the index can take **some** time -- it can take hours, certainly. Days, no.

Like I mentioned last time, it's that last step that's a doozy. Building the index efficiently is going to be the hard problem here. Here's an idea that doesn't work -- but why it doesn't work is important. As you know from my previous article on counting unique words in a document, I like making clever use of hash tables. Hash tables are (in theory…) O(1) for each insert, so building a hash table mapping n words to their posting lists should be O(n). Searching a hash table is also O(1), so we're all set, right? Of course, the combination of having a working set that might exceed the virtual memory space, plus keeping all that stuff in memory in script is going to tax the garbage collector, which was not designed for such tasks. Hmm. We could keep the hash table buckets on disk, and just read in the bucket as we needed it. Imagine if we had, say, 1024 files each containing roughly 7000 words and their posting lists. The files could be named x000, x001, … x400. All the words in, say, x001, are those words which when hashed to a ten-bit hash, give 001. We could then easily sort each index file alphabetically to make searching within it faster.  The search algorithm pseudocode looks something like this: hash the target word  
load the sorted index file for that hash into memory  
binary-search the sorted index file for the target word  
if found, display its posting list Hashing is fast, and let's assume that the index files are small enough that loading them into an array in memory is also fast. Binary searching the 7000 = n / 1024 items in the list takes about 13 comparisons, and we're done. This is pretty darn quick -- one hash, one file load, a dozen string comparisons.  How do we get here though? The index building algorithm goes like this pseudocode: for each file  
      extract the word list for this file  
      for each word in the word list  
            hash the word  
            add the word and the file name to the appropriate index file   
      next  
next  
for each index file  
      sort index file alphabetically  
next (There are more optimizations we could perform, like how to efficiently compress the posting list, but we'll come back to that later.) What's the order of this thing? Let's assume for the moment that the cost of opening the source file and extracting a list of words from it is "cheap". (We'll also return to this handwave in a later entry\!) We have in total n words in all source files. Hashing and adding each word to the end of its index file is O(1) per word, so that's O(n) altogether for the first loop. Sorting the hash files using a comparison sort comes down to O(n log n) if you work out the math. (We pick up some advantage from having to only sort 1024 small chunks of the table at a time rather than the whole table, but there's no asymptotic advantage.) Clearly that dominates the linear first loop. It seems from our analysis so far that, unsurprisingly, the sorting is the expensive part, not the hashing. We could eliminate that.  What if we went to a 20 bit hash, a million index files, and a linear search?  Then the cost of building the index is still O(n), and the sorting cost goes away, and the search gets cheaper.  Of course, then we're slamming the file system with a million tiny files, but that's another story. Let's say that there are b index buckets. **Unfortunately, neither of these techniques work very well, and the analysis above is completely bogus**. In the real world, the asymptotic analysis of processor time is irrelevant because it discounts an important fact: **disk seeks are way more expensive than processor time**. Disk access involves physically moving a robot arm over a spinning hunk of iron-coated aluminum, as opposed to juggling a few electrons at the speed of light. Let's look at this from the point of view of disk accesses. What does building the index do in this algorithm? Well, odds are good that every new word that comes in will go to a different hash bucket. That's the whole point of a good hash algorithm, after all. That means that every time we add the word to the appropriate index file, we open the file, seek the disk head to the end of the file, write bits to disk, close the file, and repeat -- seek to a new location, open the file, write bits… The hardware is pretty smart about keeping in-memory caches of what's going on, but eventually we're going to hit the same problem we're trying to avoid -- the cache will fill up, and we'll spend all our time running around the disk putting bits on platters. There are O(n) seeks\!  Seven million words, some number of files -- on average, there will be O(b) other files opened and closed before any given index file is opened the next time\! That's going to slam any cache, and probably fragment the disk while we're at it. The sorting, on the other hand, opens a file, reads it all in, sorts it, writes it all out, repeat. The disk access is highly efficient -- there are O(b) seeks, **period** -- and the processor cost of sorting a 7000 word file is not huge compared to each seek. External hash tables aren't the answer. And, for pretty much exactly the same reason, balanced multiway trees (aka "b-trees") don't work either. **The O(n) disk seeks in the first part thoroughly dominate the O(b) seeks in the sort. The processor time is largely irrelevant. ** Therefore, any indexing algorithm that we come up with should have the property that it allows **large** **sections of the index** to be written to a **small** **number of files** at a time. Next time, we'll suss out an algorithm that does fewer disk seeks during the index building.

</div>

</div>

