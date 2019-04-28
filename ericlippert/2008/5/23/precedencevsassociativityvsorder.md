# Precedence vs Associativity vs Order

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 5/23/2008 10:19:00 AM

-----

[Raymond has written about this](http://blogs.msdn.com/oldnewthing/archive/2007/08/14/4374222.aspx), I[have written about Raymond writing about it](http://blogs.msdn.com/ericlippert/archive/2007/08/14/c-and-the-pit-of-despair.aspx), but I still frequently get questions from people who are unclear on the difference between precedence, associativity and evaluation order.

I suspect that this confusion arises from the difference between how most people are trained to evaluate arithmetical expressions versus how compilers generate code to evaluate arithmetical expressions. As a child it was drilled into me that the way to evaluate an arithmetical expression was to recursively apply the PEDMAS rule. That is, evaluate anything in parentheses first. Then evaluate any exponentiations. Then divisions, multiplications, additions and subtractions, in that order. So if you had 4 x 5 x (20 - 12 / 3) you start by evaluating what's in parentheses: 20 - 12 / 3. In there, there are no parens or exponents, so start with the division. Replace 12 / 3 with 4 to get 20 - 4. Then evaluate the subtraction to get 16. Now we have the value for the parens, and we are down to 4 x 5 x 16. Evaluate one of the multiplications -- but wait, we do not know what order to evaluate the multiplications in. But we can do it in any old order, so lets say 5 x 16 is 80, so we have 4 x 80, which is 320, done.

You'll notice that the algorithm that I was taught emphasizes that you do the work on whatever is in the deepest set of parentheses first, no matter what. Many people believe that arithmetical expressions in programming languages work the same way. **They do not.**

Rather, in programming languages, parentheses indicate how the results of evaluations are combined, but not necessarily the order in which the calculations are carried out. In languages with no side effects, the order of evaluation is irrelevant. But in languages where side effects might occur, order becomes relevant.

The evaluation of an arithmetical expression is controlled by three sets of rules: precedence rules, associativity rules, and order rules.

**Precedence** rules describe how an underparenthesized expression should be parenthesized *when the expression mixes different kinds of operators*. For example, multiplication is of higher precedence than addition, so 2 + 3 x 4 is equivalent to 2 + (3 x 4), not (2 + 3) x 4.

**Associativity** rules describe how an underparenthesized expression should be parenthesized when the expression has a bunch of the same kind of operator. For example, addition is associative from left to right, so a + b + c is equivalent to (a + b) + c, not a + (b + c). In ordinary arithmetic, these two expressions always give the same result; in computer arithmetic, they do not necessarily. (As an exercise can you find values for a, b, c such that (a + b) + c is unequal to a + (b + c) in C\#?)

Now the confusing one.

**Order of evaluation** rules describe the order in which each operand in an expression is evaluated. The parentheses just describe how the results are grouped together; "do the parentheses first" is not a rule of C\#. Rather, the rule in C\# is "evaluate each subexpression strictly left to right".

The expression F() + G() \* H() is equivalent to F() + (G() \* H()), but C\# does NOT evaluate G() \* H() before F(). Rather, this is equivalent to:

 

temp1 = F();  
temp2 = G();  
temp3 = H();  
temp4 = temp2 \* temp3;  
result = temp1 + temp4;

Another way to look at it is that the rule in C\# is not "do the parentheses first", but rather to parenthesize everything then recursively apply the rule "evaluate the left side, then evaluate the right side, then perform the operation".

**This is not a rule of C++.** In C++, F(), G() and H() can be evaluated in whatever order the compiler chooses, so long as it combines the results in the right way. A legal C++ compiler might do left to right, right to left, parentheses first, whatever the compiler writer felt like.

The way this topic usually comes up is when someone has an expression chock full of side effects -- assignments, increments, decrements, pointer stores and so on, which they are attempting to convert from C++ to C\#, and report the "bug" in the C\# compiler to me that C\# does not follow the "rules" of C++. Which is ironic, because since there are not actually any *rules* in C++ about order of evaluation between two sequence points. Thus the *bug* actually is that their code was never portable C++, and only worked because the code author happened to know (or guess) what ordering the compiler writer chose.

