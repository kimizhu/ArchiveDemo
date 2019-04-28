# Desafinado, Part Three: Too Many Fifths

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/13/2005 3:27:00 PM

-----

Last time we established the diatonic scale which has the nice property that there are five tone intervals and six fifths:

<table>
<thead>
<tr class="header">
<th>Note</th>
<th>Frequency</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>A</td>
<td>220.000</td>
</tr>
<tr class="even">
<td>B</td>
<td>247.500</td>
</tr>
<tr class="odd">
<td>C</td>
<td>260.741</td>
</tr>
<tr class="even">
<td>D</td>
<td>293.333</td>
</tr>
<tr class="odd">
<td>E</td>
<td>330.000</td>
</tr>
<tr class="even">
<td>F</td>
<td>347.654</td>
</tr>
<tr class="odd">
<td>G</td>
<td>391.111</td>
</tr>
</tbody>
</table>

and then double for the next octave up and so on. But it's a little weird in that B doesn't have a fifth above it, just E a fifth below it. Similarly, F has no fifth below it. If there were any justice in this world, B and F ought to be fifth's of each other. But they're not quite right\! A fifth above B is smack between F and G. Similarly, a fifth below F is between A and B. We stuck E and B between two other notes. So let’s do it again. The fifth below F is a little bit below B, so we'll call it "B flat", or B♭. A fifth above B is a little bit above F, so we'll call it "F sharp", or F♯ In our first octave we have

<table>
<thead>
<tr class="header">
<th>Note</th>
<th>Frequency</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>A</td>
<td>220.000</td>
</tr>
<tr class="even">
<td>B♭</td>
<td>231.769</td>
</tr>
<tr class="odd">
<td>B</td>
<td>247.500</td>
</tr>
<tr class="even">
<td>C</td>
<td>260.741</td>
</tr>
<tr class="odd">
<td>D</td>
<td>293.333</td>
</tr>
<tr class="even">
<td>E</td>
<td>330.000</td>
</tr>
<tr class="odd">
<td>F</td>
<td>347.654</td>
</tr>
<tr class="even">
<td>F♯</td>
<td>371.250</td>
</tr>
<tr class="odd">
<td>G</td>
<td>391.111</td>
</tr>
</tbody>
</table>

Gah. This doesn't solve anything. **Now we've got two more notes missing a fifth**. Well, let's add them. A fifth below B♭ is E♭, a fifth above F♯ is C♯. And **again**, we've got the same problem\! A fifth below E♭ is A♭, a fifth above C♯ is G♯, and our first octave and a bit now looks like this:

<table>
<thead>
<tr class="header">
<th>Note</th>
<th>Frequency</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>A♭</td>
<td>206.017</td>
</tr>
<tr class="even">
<td>A</td>
<td>220.000</td>
</tr>
<tr class="odd">
<td>B♭</td>
<td>231.769</td>
</tr>
<tr class="even">
<td>B</td>
<td>247.500</td>
</tr>
<tr class="odd">
<td>C</td>
<td>260.741</td>
</tr>
<tr class="even">
<td>C♯</td>
<td>278.438</td>
</tr>
<tr class="odd">
<td>D</td>
<td>293.333</td>
</tr>
<tr class="even">
<td>E♭</td>
<td>309.025</td>
</tr>
<tr class="odd">
<td>E</td>
<td>330.000</td>
</tr>
<tr class="even">
<td>F</td>
<td>347.654</td>
</tr>
<tr class="odd">
<td>F♯</td>
<td>371.250</td>
</tr>
<tr class="even">
<td>G</td>
<td>391.111</td>
</tr>
<tr class="odd">
<td>A♭</td>
<td>412.033</td>
</tr>
<tr class="even">
<td>G♯</td>
<td>417.657</td>
</tr>
<tr class="odd">
<td>A</td>
<td>440.000</td>
</tr>
</tbody>
</table>

Hold on a minute here. This is getting ridiculous. A♭ and G♯ are less than 2% apart, and seem to have gotten out of order -- how is it that A♭ is lower than G♯? This is a mess. In two octaves our stringed instrument now has over 25 strings and we don't seem to be slowing down at all as far as adding new ones goes\! **** We could keep on doing this literally forever. Why? Because as the Pythagoreans discovered, **there are no whole number ratios that can be squared, cubed, or put to any other power such that you eventually end up with two**. We are **never** going to multiply **any** new note by any combination of 2/1, 1/2, 3/2 or 2/3 and end up with a note that we've already got in any other octave. The system is not closed. Look at how close A♭ and G♯ are. Wouldn't it be tempting to just "split the difference", set them equal to each other, and be done with it? Hmm. Let's throw out that A♭ in there and look at the ratios of successive notes -- not as whole number ratios, but as percentages.

<table>
<thead>
<tr class="header">
<th>Note</th>
<th>Increase</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>A♭</td>
<td>7.0%</td>
</tr>
<tr class="even">
<td>A</td>
<td>5.3%</td>
</tr>
<tr class="odd">
<td>B♭</td>
<td>6.8%</td>
</tr>
<tr class="even">
<td>B</td>
<td>5.3%</td>
</tr>
<tr class="odd">
<td>C</td>
<td>6.8%</td>
</tr>
<tr class="even">
<td>D</td>
<td>5.1%</td>
</tr>
<tr class="odd">
<td>E♭</td>
<td>7.0%</td>
</tr>
<tr class="even">
<td>E</td>
<td>5.3%</td>
</tr>
<tr class="odd">
<td>F</td>
<td>6.8%</td>
</tr>
<tr class="even">
<td>F♯</td>
<td>5.3%</td>
</tr>
<tr class="odd">
<td>G</td>
<td>6.8%</td>
</tr>
<tr class="even">
<td>G♯</td>
<td>5.3%</td>
</tr>
</tbody>
</table>

Each note on this scale is between 5.1% and 7.0% higher than the previous note. **What if we made every interval the same percentage?** What percentage would it be? We need to double in frequency over an octave, and have twelve steps to do it in. The twelfth root of two is about 1.05946. Suppose we redo this twelve-note scale so that every note is 5.946% higher than the previous. What do we get?

<table>
<thead>
<tr class="header">
<th>Note</th>
<th>Frequency</th>
<th>Difference</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>A</td>
<td>220.000</td>
<td>0.00%</td>
</tr>
<tr class="even">
<td>A♯/B♭</td>
<td>233.082</td>
<td>0.57%</td>
</tr>
<tr class="odd">
<td>B</td>
<td>246.942</td>
<td>-0.23%</td>
</tr>
<tr class="even">
<td>C</td>
<td>261.625</td>
<td>0.34%</td>
</tr>
<tr class="odd">
<td>C♯/D♭</td>
<td>277.183</td>
<td>-0.45%</td>
</tr>
<tr class="even">
<td>D</td>
<td>293.664</td>
<td>-0.11%</td>
</tr>
<tr class="odd">
<td>D♯/E♭</td>
<td>311.127</td>
<td>0.68%</td>
</tr>
<tr class="even">
<td>E</td>
<td>329.627</td>
<td>-0.11%</td>
</tr>
<tr class="odd">
<td>F</td>
<td>349.228</td>
<td>0.45%</td>
</tr>
<tr class="even">
<td>F♯/G♭</td>
<td>369.995</td>
<td>-0.34%</td>
</tr>
<tr class="odd">
<td>G</td>
<td>391.995</td>
<td>0.23%</td>
</tr>
<tr class="even">
<td>G♯/A♭</td>
<td>415.305</td>
<td>-0.56%</td>
</tr>
<tr class="odd">
<td>A</td>
<td>440.000</td>
<td>0.00%</td>
</tr>
</tbody>
</table>

This is pretty darn close to the scale we deduced from Pythagorean principles. And we have successfully split the difference -- we say that A♭ and G♯ are the same frequency. Similarly, we add D♭ = C♯ and so on, so that **every note has a fifth below and a fifth above.** This twelve-note scale is called the **equally-tempered chromatic scale**, and it is the scale that pianos and other musical instruments are actually tuned to. This scale is **a compromise between flexibility and tonal perfection**. The price you pay for having a closed system is that instead of the frequency of the higher note being 1.5 times the frequency of the lower note, it's actually 1.4983 times higher -- **every fifth interval is slightly flat**. **** And in fact that's how piano tuners do it "by ear". They tune one string to a tuning fork, then tune a fifth above it to a **perfect** 3:2 fifth, and then flatten the upper string just very slightly (or, equivalently, slightly sharpen the lower string if they are tuning a fifth below). Then they use that as the reference to tune the next note above it to a slightly flattened fifth, and so on until all twelve notes in one octave are tuned. Every other note can then be tuned to those reference notes. The process of getting the middle octave of the piano tuned is called "setting the temperament", which explains my little pun in part one. Bach's famous set of preludes and fugues, one in all twelve major and minor keys, is called "The Well-Tempered Clavier". Though it sounds good when played in an equal temperament, Bach actually designed the pieces to be played on a "Well Tempered" instrument. In that temperament certain intervals are kept pure while others are significantly off, and must be avoided by the composer. Next time, we get computers back into the picture. We'll write some programs that illustrate this fact, and thereby show how piano tuners can tune pianos by ear. Then in my final installment we'll write a program that illustrates a psychoacoustic oddity called Shepard's Illusion.

