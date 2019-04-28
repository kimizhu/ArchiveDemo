# Is there such a thing as too much precision?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/7/2010 7:02:00 AM

-----

Well, enough chit-chat, back to programming language design.

Suppose you’re building electronic piano software. As we’ve discussed before, the “equal temperament” tuning for a piano goes like this: the 49th note from the left on a standard 88 key piano is A, and its frequency is 440 Hz. Each octave above or below that doubles or halves the frequency. Why? Because humans perceive the ratio between two frequencies as the relevant factor, not the (subtractive) difference. There are twelve semitones in the chromatic scale, so to make each one sound like it has the same relationship with its previous note, each one multiplies the frequency of the previous one by the twelfth root of two.

In short, the frequency of the nth key is 440 x 2 <sup>(n-49) / 12</sup>

Now, on a real piano, its a bit more complicated than that. The high notes have some interesting problems associated with them. For the long strings in the middle of the piano, the ratio of the diameter of the string to its length is really small; zero, for practical purposes. But the short strings at the high end are short enough that the ratio is no longer practically zero. The thickness of the string causes its harmonic vibrations to be flatter than they ought to be; this is called “inharmonicity”. In addition, humans tend to perceive high notes as being flatter than they really are; our ability to determine relative pitches is somewhat skewed at the high end. For both these reasons, piano tuners often “stretch” the octaves at the high end, making the fundamental frequencies of the high notes slightly sharper than the equal temperament would suggest. Obviously electronic pianos do not have inharmonicity, and we’ll ignore the psychoacoustic problems for our purposes. Because I actually want to talk about floating point arithmetic.

Suppose you say “our expression above is equivalent to 440 x R<sup>(n-49)</sup> where R is the twelfth root of two. You compute the 12th root of 2 using calc.exe, and write some code:

 

static double Frequency(int key)  
{  
  const double TwelfthRootOfTwo = 1.0594630943592952645618252949463;  
  const double A4Frequency = 440.0;  
  return A4Frequency \* Math.Pow(TwelfthRootOfTwo, key-49);  
}

Now, this code is fine, it works, and there’s no need to mess with it. It might be nice to validate the parameter to make sure that its between 1 and 88, but, whatever. However, does it strike you as odd how much precision is in the constant? There are about 20 more decimal digits of precision in there than can be represented in a double\! A double is only accurate to around 14 or 15 digits of precision.

One thing that we can deduce from this is that calc.exe does not do its calculations in doubles, which I suppose is interesting. It must use some much higher-precision internal format.

Another thing we can note is that for this application, even fifteen digits is way more precision than we need. The highest value we’re going to get out of this is less than 4200 Hz; a rounding error after the fifteenth digit is going to be far, far too small for a human to hear the difference. If you divide an octave up into 1200 notes instead of 12, each note is called a “cent” (because 100 of them fit into a semitone). Most humans cannot hear a difference of less than a handful of cents, and this is plenty of precision to be accurate to within a few cents.

But if the compiler is simply throwing away all that precision, then that portion of the program is essentially meaningless. Shouldn’t the compiler be warning you that it is  throwing away your precision?

Well, no, not really. And here’s why.

Suppose you say

 

const double d = 108.595;

We wish to represent the exact mathematical value 108595 / 1000 in a double. But doubles use binary arithmetic; we need a power of two on the denominator, not a power of ten. The closest we can get using the 52 precision bits of a double is 1101100.1001100001010001111010111000010100011110101110 or, if you prefer fractions, that’s 3820846886986711 / 35184372088832. That fraction has the exact value in decimal of 108.594999999999998863131622783839702606201171875, which you must admit is awfully close to 108.595, though a touch lower than it.

So here’s the thing. You want to represent the exact value of 108.595, but we don’t let you; we make you accrue some error. And we don’t warn about that; we assume that you know that some error will be accrued. (If you wanted an exact result then you should have used decimal, not double.) But what if the exact value you actually wanted to represent was in fact 108.594999999999998863131622783839702606201171875 ?  That number *can* be represented in binary *exactly*\! We certainly should not give a warning that you have too much precision if you say

 

const double d = 108.594999999999998863131622783839702606201171875;

because you have *exactly the right amount of precision in order to have zero representation error.* It seems wacky to say “please round that off to fifteen decimal digits so that we can give you this exact value right back again.” Why should we punish you for being *more* accurate?

The best we could do is give a warning when there was more precision than could be represented, \*and\* the representation error was non-zero, \*and\* losing the extra precision makes the representation error *smaller*, \*and\*… now we’ve written a whole lot of complicated math code, the result of which makes the user scratch their head and say “how the heck am I supposed to make the compiler happy with this constant?” Warnings that are nothing but a source of pain are bad warnings, and we try to avoid them.

