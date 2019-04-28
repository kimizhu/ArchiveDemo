# First Cousins Once Removed

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/4/2010 7:04:00 AM

-----

Happy New Year all, and welcome to 2010, or, as my friend Professor Orbifold prefers it, MMX. I hope your festive holiday season was as festive and enjoyable as mine.

The extended Lippert family continues to grow; this year at the annual Boxing Day party we needed two overflow tables for dinner instead of the usual one. The older cousins are now all married, some of them are bringing new babies, and some of the younger cousins are starting to bring boyfriends and girlfriends. I pointed out that the youngest person at the table was my brand-new “first cousin once removed”, which touched off a long discussion of how exactly one computes degree of cousin-hood and removed-ness. As a public service, here’s how it works.

**Recursive explanation for computer programmers and mathematicians:**

**Base case**: if X and Y are zeroth cousins no times removed then X and Y are siblings.

**Recursive case 1**: if n \> 0 and X and Y are nth cousins no times removed then X.Parent and Y.Parent are (n-1)th cousins no times removed.

**Recursive case 2**: if m \> 0 and X and Y are nth cousins m times removed then WOLOG assume that X is of a later generation than Y. In this case, X.Parent and Y are nth cousins (m-1) times removed.

**Explanation for normal people:**

Take two different people who have a common ancestor but who are not related by direct succession (that is, neither is the mother, father, grandmother, and so on, of the other):

Degree of cousin-hood is the **minimum** of the numbers of generations back that you have to go to find the nearest common ancestor, minus one.

Removed-ness is the **absolute difference** between the numbers of generations back you have to go to find the nearest common ancestor.

For example, consider this fragment of a family tree:

 

        Mary  
       /     \\  
   Laura     Bob  
     |        |  
   John     Helen  
     |        |  
  Xerxes   Melinda

Take Helen and Xerxes for example. Their nearest common ancestor is Mary. Helen has to go back two generations to get to Mary. Xerxes has to go back three. The minimum of two and three, minus one, is one. The absolute difference of two and three is one. So Helen and Xerxes are first cousins, once removed. So are John and Melinda.

“Zeroth cousins” are not called cousins, but have special names for the relationship. “Zeroth cousins no times removed” are of course brothers and sisters. “Zeroth cousins once removed” are uncles, aunts, nieces and nephews. “Zeroth cousins twice removed” are great-uncles, great-aunts, great-nieces and great-nephews. (Or sometimes grand-uncles, and so on.)

Summing up this family tree:

Mary is Laura and Bob’s mother, John and Helen’s grandmother, and Xerxes and Melinda’s great-grandmother.

Laura is Mary’s daughter, Bob’s sister, John’s mother, Helen’s aunt,  Xerxes’ grandmother, and Melinda’s great-aunt.

Bob is Mary’s son, Laura’s brother, John’s uncle, Helen’s father, Xerxes’ great-uncle, and Melinda’s grandfather.

John is Mary’s grandson, Laura’s son, Bob’s nephew, Helen’s first cousin (no times removed), Xerxes’ father, and Melinda’s first cousin once removed.

Helen is Mary’s granddaughter, Laura’s niece, Bob’s daughter, John’s first cousin (no times removed), Xerxes’ first cousin once removed, and Melinda’s mother.

Xerxes is Mary’s great-grandson, Laura’s grandson, Bob’s great-nephew, John’s son, Helen’s first cousin once removed, and Melinda’s second cousin (no times removed).

Melinda is Mary’s great-granddaughter, Laura’s great-niece, Bob’s granddaughter, John’s first cousin once removed, Helen’s daughter, and Xerxes’ second cousin (no times removed).

Of course, this is all perfectly straightforward. If, say, your mother dies and your father marries her sister and has more children, then working out who is whose cousin can get rather trickier. And heaven only knows what happens [if you are your own grandfather](https://www.youtube.com/watch?v=eYlJH81dSiw&feature=fvw).

Incidentally, it is *illegal* in the state of Washington for a man to marry his widow’s sister. Anyone care to guess why?

