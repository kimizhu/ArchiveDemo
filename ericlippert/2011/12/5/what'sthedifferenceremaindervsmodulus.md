# What's the difference? Remainder vs Modulus

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 12/5/2011 7:05:00 AM

-----

Today, another episode of my ongoing series "[What's the difference?](http://blogs.msdn.com/b/ericlippert/archive/tags/what_2700_s+the+difference_3f00_/)" Today, what's the difference between a remainder and a modulus, and which, if either, does the % operator represent in C\#?

-----

A powerful idea that you see come up in mathematics and computer programming [over and over again](http://blogs.msdn.com/b/ericlippert/archive/2010/02/08/making-the-code-read-like-the-spec.aspx) is the idea of an **equivalence relation**.

First off, let's define (again) what a "relation" is; a relation is a function that takes two things, say, integers, and returns a Boolean that says whether the two integers are "related" or not. For example, "is greater than" is a relation. It takes two integers, say 4 and 2, and returns a Boolean that indicates whether 4 "is greater than" 2 or not; in this case, yes, it is.

An "equivalence relation" is a relation which is reflexive, symmetric, and transitive. That is, if ∾ is an equivalence relation, then:

Reflexive: X∾X is true for any X  
Symmetric: X∾Y = Y∾X for any X and Y  
Transitive: if X∾Y and Y∾Z are both true then X∾Z is also true.

Clearly "is greater than" is not an equivalence relation; though it is transitive, it is neither symmetric nor reflexive. "Have the same sign" on the other hand, is an equivalence relation; all negative numbers are equivalent to each other, all positive numbers are equivalent to each other, and zero is equivalent to itself.

An equivalence relation partitions the set of integers into (possibly infinitely many) *equivalence classes*. For example, let's consider the equivalence relation A∾B if and only if A-B is evenly divisible by 4. So 0∾4 is true and -86∾2 is true, and so on. We can make four "equivalence classes" where every integer is in exactly one class, and every integer in each class is related to every other integer in that class. The classes are {0, 4, -4, 8, -8, 12, -12, ... }, {1, -3, 5, -7, 9, -11, ... }, {2, -2, 6, -6, 10, -10} and {3, -1, 7, -5, 11, -9, ... }.

These classes are classes of "equivalent" integers, where "equivalence" means "is equally good in terms of determining whether the relation holds". Equivalence classes are often identified by a "canonical" member; in the case of the equivalence classes of "integers that are congruent modulo four", the canonical members are usually taken to be 0, 1, 2 and 3.

One can of course generalize this; given any positive integer n you can create the equivalence relation "A∾B if and only if A-B is evenly divisible by n". This defines n equivalence classes, which are usually identified by the canonical elements 0, 1, ..., n-1. The relation is usually written (somewhat clumsily, in my opinion) as A ≡ B mod n and is read as "A is congruent to B modulo n".

It is very tempting to think of the A % M operator in C\# as meaning "partition the integers into M equivalence classes and give me the canonical element associated with the class that contains A" operator. That is, if you say 123 % 4, the answer you get is 3 because 123 ≡ 3 mod 4, and 3 is the "canonical" element associated with the equivalence class that contains 123.

However, that is not at all what the % operator actually does in C\#. The % operator is not the *canonical modulus operator*, it is the *remainder operator*. The A % B operator actually answer the question "If I divided A by B using integer arithmetic, what would the remainder be?"

In order to work out what the remainder is, we need to first clearly define our terms. The divisor, dividend, remainder and quotient must obey the following identity:

dividend = quotient \* divisor + remainder

Now, that is not enough to uniquely determine an answer to the question of what, say, 123 / 4 is. 123 = 30 \* 4 + 3 is the desired solution, but 123 = 6000 \* 4 - 23877 is also a possible solution. We need to put restrictions on the quotient and remainder.

Naively we might say that the lesser quotient is the better quotient. But of course that does not help either, because 123 = 1 \* 4 + 119 is also perfectly good, and 1 is less than 30, but 1 is not a better quotient. We need some better rules; the rules we've come up with to determine the quotient and remainder (assuming that the divisor and dividend are valid, that is, we're not dividing by zero and so on) are:

\* If there is any quotient that makes the remainder zero then that's the quotient and the remainder is zero, and we're done.  
\* Otherwise, do the division in all non-negative integers. Take the **largest** quotient that keeps the remainder **positive**.  
\* If the divisor and dividend have opposite signs then make the quotient negative.  
\* The remainder can now be worked out so as to obey the identity.

The net result here is that integer division rounds towards zero. That should make sense; we want 123/4 to be 30 with a remainder of 3, not 31 with a remainder of -1. Similarly, we want -123/4 to be -30, not -31\! It would be bizarre to say that 4 goes into 123 30 times but goes into -123 -31 times\! One expects that changing the sign of a term changes the *sign* of the result; it does not change the *magnitude* of the result.

If -123/4 is -30, then what is the remainder? It must obey the identity, so the remainder is -3. That is not the canonical item associated with the equivalence class that contains -123; that canonical item is 1.

It's important to remember this fact when doing things like balancing a hash table:

int bucketIndex = item.GetHashCode() % bucketCount; 

or determining if a number is odd:

var oddNumbers = someSequence.Where(x =\> x % 2 == 1);

The first is wrong because the array index can be negative if the hash code is negative; the second does not classify -123 as an odd number. **The % operator does not give the canonical modulus, it gives the remainder.**

