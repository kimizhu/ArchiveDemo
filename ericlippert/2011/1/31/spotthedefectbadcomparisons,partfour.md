# Spot the defect: Bad comparisons, part four

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/31/2011 6:44:00 AM

-----

One more easy one. I want to "sort" a list into a random, shuffled order. I can do that by simply randomizing whether any two elements are greater than, less than, or equal to each other:

 

myList.Sort((x, y) =\> (new Random()).Next(-1, 2));

That generates a random -1, 0 or 1 for every comparison, right? So it will sort the list into random order, right?

.

.

.

.

.

.

.

There are multiple defects here. First off, clearly this violates all our rules for comparison functions. It does not produce a total ordering, and in fact it can tell you that two particular elements are equal and then later tell you that they have become unequal. The sort algorithm is allowed to go into infinite loops or crash horribly when given such an ill-behaved comparison function. And in fact, some implementations of sorting algorithms attempt to detect this error and will throw an exception if they determine that the comparison is inconsistent over time.

Second, every time this thing is called it creates a new Random class seeded to the current time, and therefore it produces the same result over and over again; hardly random.

Shuffling is not sorting; it is the opposite of sorting, so don't use a sort algorithm to shuffle. There are lots of efficient shuffle algorithms that are easy to implement.

