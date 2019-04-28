# Continuing to an outer loop

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/11/2010 6:54:00 AM

-----

When you have a nested loop, sometimes you want to “continue” the outer loop, not the inner loop. For example, here we have a sequence of criteria and a sequence of items, and we wish to determine if there is any item which matches every criterion:

 

match = null;  
foreach(var item in items)  
{  
  foreach(var criterion in criteria)  
  {  
    if (\!criterion.IsMetBy(item))  
    {  
      // No point in checking anything further; this is not  
      // a match. We want to “continue” the outer loop. How?  
    }  
  }  
  match = item;  
  break;  
}

There are many ways to achieve this. Here are a few, in order of increasing awesomeness:

**Technique \#1 (ugly):** When you have a loop, the “continue” statement is essentially a “goto” which branches to the bottom of the loop, just before it does the loop condition test and pops back up to the top of the loop. (Apparently when you spell “goto” as “continue”, the “all gotos are all evil all the time” crowd doesn’t bother you as much.) You can of course simply make this explicit:

 

match = null;  
foreach(var item in items)  
{  
  foreach(var criterion in criteria)  
  {  
    if (\!criterion.IsMetBy(item))  
    {  
      goto OUTERCONTINUE;  
    }  
  }  
  match = item;  
  break;  
OUTERCONTINUE:  
  ;  
}

**Technique \#2 (better):** When I see a nested loop, I almost always consider refactoring the interior loop into a method of its own.

 

match = null;  
foreach(var item in items)  
{  
  if (MeetsAllCriteria(item, critiera))  
  {  
    match = item;  
    break;  
  }  
}

where the body of MeetsAllCriteria is obviously

 

foreach(var criterion in criteria)  
  if (\!criterion.IsMetBy(item))  
    return false;  
return true;

**Technique \#3 (awesome):** The “mechanism” code here is emphasizing how the code works and hiding the *meaning* of the code. The *purpose* of the code is to answer the question “what is the first item, if any, that meets all the criteria?” If that’s the meaning of the code, then write code that reads more like that meaning:

 

var matches = from item in items  
              where criteria.All(  
                criterion=\>criterion.IsMetBy(item))  
              select item;  
match = matches.FirstOrDefault();

That is, search the items for items that meet every criterion, and take the first of them, if there is one. The code now reads like what it is supposed to be implementing, and of course, if there are no loops in the first place then there’s no need to “continue” at all\!

The biggest thing I love about LINQ is that it enables a whole new way to think about problem solving in C\#. It’s gotten to the point now that whenever I write a loop, I stop and think about two things before I actually type “for”:

\* Have I described anywhere what this code is doing **semantically**, or does the code emphasize more what **mechanisms** I am using at the expense of obscuring the semantics?

\* Is there any way to characterize the solution to my problem as the result of a **query against a data source**, rather than as the result of **a set of steps to be followed procedurally**?

