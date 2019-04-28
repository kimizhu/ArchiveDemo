# Desafinado, Part Two: A Perfect Pythagorean Tuning

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/11/2005 2:22:00 PM

-----

Last time we talked about the Pythagorean's discovery that sounds with vibrations in the ratios 1:2 and 2:3 sound consonant to the human ear. Let's explore the consequences of that a bit. Suppose we're building a stringed instrument from scratch and we want to tune it so that it sounds good to human ears.  Let's tune the first string so that it vibrates at 440 vibrations per second.  The unit of vibrations per second is the Hertz, so this is a 440Hz string.  Call that string "A2". There's no particularly good reason to pick 440Hz versus any other frequency, but we've got to start somewhere. Also, note that I'm assuming here that we have some magical way to exactly tune a string to a particular number of vibrations. The practical details of piano tuning in the absense of electronic measuring equipment are interesting, but before we get into that, I want to have a theoretical basis for what the ideal tuning ought to be. Anyway, we've labeled this string "A2", and it's 440Hz.  We'll add two more strings, an octave above and below that: A1=220Hz and A3=880Hz. If we want to do nice harmonies and melodies, we have an intuition from Pythagorus that fifths are a good idea. What note below A2 makes a perfect fifth with A2?  That is, what to 440Hz makes a 2:3 ratio?  Easy -- 293.333Hz.  Call that string D1, and make a string D2 an octave above it.  So far we've got Note Frequency  
 A1   220.000  
 D1   293.333  
 A2   440.000  
 D2   586.666  
 A3   880.000 A now has a fifth below, D has a fifth above. Let's make a fifth below D. What to D2 makes a 2:3 ratio?  Call that G1, and add an octave G2 above it: Note Frequency  
 A1   220.000  
 D1   293.333  
 G1   391.111  
 A2   440.000  
 D2   586.666  
 G2   782.222  
 A3   880.000 Now do the same thing to G2 to make C1 and C2, and the same thing to C2 to make F1 and F2, and we end up with a stringed instrument that looks like this in the first octave: (the second octave is all these doubled, so I'm going to stop calling out which octave we're in when I name the note.  It'll be clear from context.) Note Frequency  
 A    220.000  
 C    260.741  
 D    293.333  
 F    347.654  
 G    391.111 We'd then have an eleven-stringed instrument on which you could play traditional Asian music.  This is called the **pentatonic scale** because it has five notes before it repeats. Something that's interesting about this scale is the **ratios between successive notes' frequencies:** A : C = 27 : 32  
C : D =  8 : 9  
D : F = 27 : 32  
F : G =  8 : 9  
G : A =  8 : 9 Clearly there is something interesting going on with this 8:9 ratio.  Therefore, we'll give it a name. Just as the interval between two notes that are 2:3 is called a "fifth", the interval between two notes that are 8:9 is called a "tone". There are two notable things here. First, the gaps from A to C and D to F are awfully big, quite a bit bigger than a tone.  Second, though D:A is a fifth, there is no string such that A:x forms a fifth.  We forgot to add a fifth above A. In keeping with our intuition that the 8:9 ratio is important, let's insert tones above A and D.  We'll add notes in an 8:9 ratio, call them B and E, and then work out the ratios of each string to its neighbour again: Note  Frequency Ratio to next  
 A     220.000      8 : 9  
 B     247.500    243 : 256  
 C     260.741      8 : 9  
 D     293.333      8 : 9  
 E     330.000    243 : 256  
 F     347.654      8 : 9  
 G     391.111      8 : 9 Hey, check it out -- this has the nice additional property that E is a fifth above A\! Coolness. Two birds with one stone and all that. This is called the **diatonic scale**, and is the standard scale for western music.  These are the white keys on a piano.  Interestingly, the ancient Greeks developed this standard scale, but different groups wrote music in modes that emphasized different starting notes. If you play the scale CDEFGABC, we'd call that a "major" scale, but the Greeks would have called it the "Ionian Mode".  The Aeolian Mode started on A, and we'd call that the "natural minor" scale.  The Mixolydians started on G. (The Beatles wrote several songs in Mixolydian Mode, which is why songs like "Love Me Do" and "A Hard Day's Night" have a kind of odd sound to their chord progressions.)  There is a different Greek tribe associated with each possible starting note, but they all used this scale. Now we see where the terms "octave" and "fifth" come from. There are **eight** diatonic notes between (inclusive) any two notes an octave apart, and for every note **except B and F**, there are **five** notes between two notes a fifth apart. B has no fifth above, F has no fifth below. The diatonic scale has many nice properties -- it has a large number of pairs of notes that sound good together. **The greatest number of consonant pairs of any seven note scale, in fact.**  (The proof is left as an exercise to the reader.) A consequence of this fact is that if we want **more** consonant pairs, we're going to have to add **more notes**. Let's do that. After all, so far everything is working out great\! Next time: fixing that problem with B and F not having one of their fifths is going to screw *everything* up. (Foreshadowing -- your sign of a quality blog.)

