# Fun With Floating Point Arithmetic, Part Three

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 1/17/2005 1:54:00 PM

-----

I've been getting lots of mail, questions and pointers to interesting articles on some of the trials and tribulations of using floating point arithmetic correctly. Please do keep it coming\! Though I am certainly no expert in this area, I'm happy to take a crack at any questions you might have.

To sum up the story so far, "normal" floats are all numbers representable by this pattern: +/- 1.(52 binary digits after the decimal point) x 2<sup>exp</sup> , where exp is any integer from -1022 through +1023. We also have special floats to represent zero, other tiny floats with less than 52 digits of precision, infinities and NaNs, but we'll ignore those for now. Clearly a float can represent every 53 bit integer with full fidelity. After you get to 54 bit integers though, only every other integer is going to be representable with full fidelity. With 55 bit integers, only every fourth, and so on. A reader wrote in to ask some questions about how floating point numbers are displayed in decimal. He noticed a weirdness in VBScript, but it's actually easier to show the scenario in JScript. (There are additional factors at play in VBScript which I may get to in another article later.) Consider the following: var x = 0x8000000000000800; That would be a 64 bit unsigned integer. It's obviously too large to fit into a 32 bit signed integer, so JScript generates a float and assigns it to the variable slot. However, since this number requires exactly 53 bits, **it can be represented with full fidelity as a float**. It is not rounded. In decimal notation, this value should be 922337203685477 7856. But if we print out the value of x, we get 9223372036854777000\! Why is it rounded off when this particular float has full precision? Maybe it doesn't have full precision. Maybe I've been lying to you this whole time. Maybe in fact the float is stored in decimal internally, with a 16 slot decimal digit buffer\! Fortunately, we can test this hypothesis out. print(x % 0x800); // 0  
print(x % 10);  // 6 Whew\! The mod operator shows that JScript believes that this number is evenly divisible by 2048, and that the last digit when represented in base 10 is in fact 6. But that just makes it even more confusing\! If JScript knows that the last decimal digit is a six, why does converting the number to a string end in a zero? Because **we do not want to ever make it look like a float has more precision than it actually does**. By lopping off the last few decimal digits and replacing them with zeros, we emphasize that floats are accurate only to about fifteen or sixteen significant decimal digits. Imagine the confusion that would result if the situation were reversed: var x = 9223372036854777000;  
print(x); // prints 9223372036854777856 Where did the extra precision come from? To the naïve user who does not realize that numbers are stored in binary internally, this looks really bizarre. They put in something with 16 significant digits and something with 19 comes out\! We do this rounding because in the real world, people expect floating point numbers to act like decimal numbers, not binary numbers.  A correct and efficient float-to-string algorithm which shows numbers with a prescribed level of decimal precision is surprisingly difficult to write, particularly if you add the requirement that the string-to-float algorithm have nice "round trip" properties. There are lots of places where things can go slightly wrong. You've probably noticed already, for instance, that the algorithm which JScript uses does NOT have the property that the decimal integer which comes out is the *closest* decimal integer to the actual value. Given that we're going to round to a fixed number of decimal significant digits, we would expect that 9223372036854777856 would be rounded to 922337203685477 8000, not 9223372036854777000. In fact, the specification categorically states that the last digit need not be correctly rounded, because doing so is a pain. Section 9.8.1 of ECMA specification 262, Revision 3 reads as follows: (emphasis added) The operator ToString converts a number m to string format as follows:

1.  If m is NaN, return the string "NaN".
2.  If m is +0 or

\-0, return the string "0".

If m is less than zero, return the string concatenation of the string "-" and ToString(

\-m).

If m is infinity, return the string "Infinity".

Otherwise, let n, k, and s be integers such that k

³ 1, 10<sup>k-1</sup> £ s \< 10<sup>k</sup>, the number value for s ´ 10<sup>n-k</sup> is m, and k is as small as possible. Note that k is the number of digits in the decimal representation of s, that s is not divisible by 10, and that **the least significant digit of s is not necessarily uniquely determined by these criteria.**

If k

£ n £ 21, return the string consisting of the k digits of the decimal representation of s (in order, with no leading zeroes), followed by n-k occurrences of the character ‘0’.

If 0 \< n

£ 21, return the string consisting of the most significant n digits of the decimal representation of s, followed by a decimal point, followed by the remaining k-n digits of the decimal representation of s.

If

\-6 \< n £ 0, return the string consisting of the character ‘0’, followed by a decimal point, followed by -n occurrences of the character ‘0’, followed by the k digits of the decimal representation of s.

Otherwise, if k = 1, return the string consisting of the single digit of s, followed by lowercase character ‘e’, followed by a plus sign ‘+’ or minus sign ‘

\-’ according to whether n-1 is positive or negative, followed by the decimal representation of the integer abs(n-1) (with no leading zeros).

Return the string consisting of the most significant digit of the decimal representation of s, followed by a decimal point, followed by the remaining k

\-1 digits of the decimal representation of s, followed by the lowercase character ‘e’, followed by a plus sign ‘+’ or minus sign ‘-’ according to whether n-1 is positive or negative, followed by the decimal representation of the integer abs(n-1) (with no leading zeros). NOTE The following observations may be useful as guidelines for implementations, but are not part of the normative requirements of this standard.

  - If x is any number value other than

\-0, then ToNumber(ToString(x)) is exactly the same number value as x. ****

The least significant digit of s is not always uniquely determined by the requirements listed in step 5.

For implementations that provide more accurate conversions than required by the rules above, it is recommended that the following alternative version of step 5 be used as a guideline:

5\. Otherwise, let n, k, and s be integers such that k

³ 1, 10<sup>k-1</sup> £ s \< 10<sup>k</sup>, the number value for s ´ 10<sup>n-k</sup> is m, and k is as small as possible. If there are multiple possibilities for s, choose the value of s for which s ´ 10<sup>n-k</sup> is **closest in value to m**. **If there are two such possible values of s, choose the one that is even.** Note that k is the number of digits in the decimal representation of s and that s is not divisible by 10. If you are *really* interested in this subject, read this excellent paper: <http://www.ampl.com/REFS/rounding.pdf>. This paper was highly influential in the design and implementation of the JScript float-to-string conversion algorithm. VBScript has some even weirder rules for conversion of floats to strings -- VBScript's FormatNumber method will convert that float to the string 9,223,372,036,854,780,000.00, which has one fewer digits of precision. The reasoning behind that will have to wait for another post\!

