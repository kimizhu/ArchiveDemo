# A Simple Puzzle

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/24/2012 6:45:00 AM

-----

My original version of the histogram-generating code that I whipped up for the previous episode of FAIC contained a subtle bug. Can you spot it **without going back and reading the corrected code?**

private static int\[\] CreateHistogram(IEnumerable\<double\> data, int buckets, double min, double max)  
{  
  int\[\] results = new int\[buckets\];  
  double multiplier = buckets / (max - min);  
  foreach (double datum in data)  
  {  
    int index = (int) ((datum - min) \* multiplier);  
    if (0 \<= index && index \< buckets)  
      results\[index\] += 1;  
  }  
  return results;  
}

Note that of course if this were production code, instead of demo code I whipped up in five minutes, it would be a lot more robust in its error detection; the bug that I am looking for is a bona fide error in the logic of the method, rather than things like "the method does not verify that min is smaller than max", and so on.

A hint: the first time I ran this code and displayed the results, the generated histogram looked fine. Then I made a small change to the arguments and the resulting histogram image was obviously wrong. Can you spot the defect?

