# What Are The Semantics Of Multiple Implicitly Typed Declarations? Part Two

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 6/27/2006 1:08:00 PM

-----

Many thanks for all your input in my informal poll yesterday. The results were similar to other "straw polls" we've done over the last couple of months. In this particular poll the results were:

var a=A, b=B; where the expressions are of different types should:

  - have the same semantics as var a=A; var b=B;: 12
  - replace the var with some type for both: 3
  - give an error: 6

There were 18 comments; a few people voted twice, which is fine with me.

The way the feature is specified is that the var is to be replaced with the best type compatible with all the expressions, to maintain the invariant that parallel declarations like this always give the same type to each variable. Many people that we've polled believe that this is the "intuitively obvious" choice, including much of the language design team. A larger group of language users believes that "infer each variable type separately" is the "intuitively obvious" choice.

So what to do? We have a relatively unimportant edge-case feature where customers strongly disagree as to what the code "obviously" means, and the difference can lead to subtle bugs. That's clearly badness. Given this feedback, amply confirmed by you all, we are probably going to simply remove multiple implicitly typed declarations from the C\# 3.0 language.

Thanks for your feedback\!

