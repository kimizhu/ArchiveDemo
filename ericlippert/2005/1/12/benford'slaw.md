# Benford's Law

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/12/2005 12:18:00 PM

-----

While I was poking through my old numeric analysis textbooks to refresh my memory for this series on floating point arithmetic, I came across one of my favourite weird facts about math. A nonzero base-ten integer starts with some digit other than zero. You might naively expect that given a bunch of "random" numbers, you'd see every digit from 1 to 9 about equally often. You'd see as many 2's as 9's. You'd see each digit as the leading digit about 11% of the time.  For example, consider a random integer between 100000 and 999999. One ninth begin with 1, one ninth begin with 2, etc. But in real-life datasets, that's not the case at all. If you just start grabbing thousands or millions of "random" numbers from newspapers and magazines and books, you soon see that about 30% of the numbers begin with 1, and it falls off rapidly from there. About 18% begin with 2, all the way down to less than 5% for 9. This oddity was discovered by Newcomb in 1881, and then rediscovered by Frank Benford, a physicist, in 1937. As often is the case, the fact became associated with the second discoverer and is now known as Benford's Law. Benford's Law has lots of practical applications. For instance, people who just make up numbers wholesale on their tax returns tend to pick "average seeming" numbers, and to humans, "average seeming" means "starts with a five". People think, I want something between $1000 and $10000, let's say, $5624. The IRS routinely scans tax returns to find unusually high percentages of leading 5's and examines those more carefully. Benford's result was carefully studied by many statisticians and other mathematicians, and we now have a multi-base form of the law. Given a bunch of numbers in base B, we'd expect to see leading digit n approximately ln (1 + 1/n) / ln B of the time. But what could possibly explain Benford's Law? Multiplication. Most numbers we see every day are not random quantities in of themselves. They're usually computed qualities with some aspect of multiplication to them. Consider, for example, any property which grows on a percentage basis. Like, say, the Dow Jones Industrial Average. It typically grows a few percent a year. Suppose, just to pick a rate, that on average the DJIA grows at 7% a year. At that rate, it doubles about every ten years. Suppose that the DJIA is 10000. After ten years of having 1 as the leading digit, it finally gets to 20000. Ten years go by again, but in that ten years, it doubles to 40000, not 30000. Therefore, those ten years were spent about half starting with 2, and about half starting with 3. Ten more years go by, and it doubles again to 80000. Now ten years have 4, 5, 6 and 7 as the leading digits in only ten years. Eventually we get up to 100000, and spend another ten years starting with 1.  Pick a random date and you'd expect that the DJIA on that day would be twice as likely to start with 1 as 2, and four times as likely to start with 1 as 5. We can easily write a program that demonstrates Benford's Law. As we multiply more and more numbers together, they tend to clump based on Benford's Law: var counters = \[  
\[0,0,0,0,0,0,0,0,0,0\],  
\[0,0,0,0,0,0,0,0,0,0\],  
\[0,0,0,0,0,0,0,0,0,0\],  
\[0,0,0,0,0,0,0,0,0,0\] \]; for (var multiplications = 0 ; multiplications \<= 3; ++multiplications)  
{  
  for (var trial = 0 ; trial \< 10000 ; ++trial)  
  {  
    var num = 1;  
    for (var mult = 0 ; mult \<= multiplications; ++mult)  
      num = num \* (Math.floor(Math.random() \* 1000) + 1);  
    var lead = num.toString().substr(0,1);  
    counters\[multiplications\]\[lead\] ++;  
  }  
}  
print(counters.join("\\n")); A typical run produces data from which we can draw this table:                          Leading Digit  
Mults       1    2    3    4    5    6    7    8    9  
  0       1102 1069 1085 1125 1167 1107 1083 1124 1138  
  1       2416 1752 1443 1162 1019  756  643  453  356  
  2       3046 1854 1265  979  778  632  551  468  427  
  3       3090 1814 1197  924  779  661  582  491  462  
predicted 3010 1761 1249  969  792  669  580  511  458 As you can see, with no multiplications, the distribution is a flat 11% for each. But by the time we get up to two or three multiplications, we're almost exactly at the distribution predicted by Benford's Law. What does this have to do with floating point math? Well, we could conceivably design chips that did decimal or hexadecimal floating-point arithmetic. Would such chips yield more accurate results? Well, recall that last time, we used the fact that you can stuff a leading 1 onto a bit field to define a number. Binary is the only system in which every number except 0 begins with a leading 1\! You can make a statistical argument which shows that for bases other than binary, in which you cannot always assume a leading digit, have on average a larger representation error. The argument is somewhat subtle, so I'm not going to actually go through the details of it, but suffice to say that we can show that for typical uses, binary is the least error-producing system we can come up with given that we'll almost always be working with data which follow Benford's Law.

