# VBScript Quiz Answers, Parts Eleven and Twelve

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/5/2005 2:25:00 PM

-----

11\) X andY are both Booleans.  Which of the following always assignsTrue?  Why? (a) Z = Not X Or Y = X Imp Y  
(b) Z = X Imp Y = Not X Or Y  
(c) Z = X Eqv Y = Not X XOr Y  
(d) Z = Not X XOr Y = X Eqv Y  (a) assigns False ifX andY are bothFalse.  The others assignTrue no matter whatX andY are. However, these are not as straightforward as you might think. Imp is the "logical implication" operator.XImpY means "IfX isTrue thenY isTrue". It is alwaysTrue unlessX isTrue andY isFalse. Eqv is the "logical equivalence" operator.X Eqv YisTrue ifX andY are bothTrue or bothFalse. Why do we need Eqv -- isn't that the definition of "equals"?  The difference is that if given numbers instead of Booleans, these do the same logic on corresponding pairs of bits. If two logical expressions are always equal no matter what the values of their variables are, then they are logically identical. Such a relationship is called an "identity".  These are clearly identities: (a) Z = ((Not X) Or Y) = (X Imp Y)  
(b) Z = (X Imp Y) = ((Not X) Or Y)  
(c) Z = (X Eqv Y) = (Not (X XOr Y))  
(d) Z = (Not (X XOr Y)) = (X Eqv Y)    No matter what Boolean values X and Y are, Z will always be True.  The trick here though is the operator precedence. Just like 1 + 2 \* 3 is 7, not 9, these guys all have precedence.  To show the precedence with brackets, the expressions I actually asked about are equivalent to (a) Z = (((Not X) Or (Y = X)) Imp Y)  
(b)Z = (X Imp ((Y = (Not X)) Or Y))  
(c)Z = (X Eqv ((Y = (Not X)) XOr Y))  
(d)Z = (((Not X) XOr (Y = X)) Eqv Y) It is much less clear now that the last three are identities. The moral of the story is to be very careful with operator precedence when using logical operators -- equals binds tighter than Or\! 12) Consider this program: s = "Eric read Æsop's Fables"  
arr = Split(s, "e", -1, 1) arr is an array with how many elements? (a) 2  
(b) 3  
(c) 4  
(d) 5 It's (c).  The parts are "", "ric r", "ad Æsop's Fabl", "s" The 1 means "go case insensitive". I agonized over whether or not Split should split on matches to ligature characters. It gave me grief because this *does* work: print Instr(1, "xxxÆxxx", "e", 1) That finds the "e" inside the ligature, so why shouldn't Split split on the ligature? Basically, we just decided that it was *too weird* to split a string in the middle of a ligature. I was never very happy with this decision, and if I had to do it again, I probably would decide the other way.  But it's too late now\! Wow, that took longer than I expected.  Coming up next, a short break from scripting.

