# Desafinado, Part Five: Getting Down Without Hitting The Bottom

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/18/2005 2:49:00 PM

-----

Back in the 1960's a guy named Shepard published a paper which described a way to create a descending scale of twelve notes such that every consecutive pair was perceived as being two notes, the second one lower than the first. That's not hard -- every descending scale has that property\! The kicker is that in a Shepard scale, **people also perceive the last note of the scale as higher than the first.** Clearly that's totally impossible. If every note is lower than the one before then the twelfth cannot be higher than the first. And yet Shepard's Illusion is pretty strong. Another guy named Risset figured out a way to make the scale "continuous", so it sounds like one long note constantly getting lower but never hitting bottom. It's pretty tricky to pull off the illusion, and I haven't done a perfect job, but this at least illustrates it somewhat. The key to pulling off the illusion is to have the tone actually made up of many overtones and subtones of a primary tone. Take A=440Hz, for example. As I mentioned back in part one, for some reason humans seem to strongly associate perfect octaves. 220Hz and 880Hz sound like "the same note", only lower or higher respectively. It is hard to realize that this is not one tone, but actually many. What we'll do is play the three A's from the three different octaves simultaneously. If we played them all at the same volume, it's hard for the human ear to pick out which one is the "most important", but if we play the bottom one very softly, the middle one very loud, and the top one about medium, then it's easy to "lock on" to the mid-range component of the sound. Imagine someone playing this on the piano -- hitting three A's, the middle one loud and the bottom and top ones quietly. Now move all three notes UP eleven semitones, so that the A♭ at the bottom is just below what was previously the middle A. Play the bottom note really loud but slightly softer than the previous loud A, the middle note slightly louder than the previous high A, and the new high note very soft indeed. The human ear does not perceive that as every note moving eleven semitones up, but rather that the loudest tone has moved down. Now take all three notes down a semitone. Every time you go down a semitone, make the bottom note a little quieter and the middle and top notes a little louder. Keep going until you get back to where you started, at which point go up eleven semitones again. That's the essence of the illusion. The human ear attunes itself to the loudest part of the tone, and therefore doesn't notice that the lows are dropping out of the bottom and being replaced by very faint notes at the top. As the top notes descend they get louder and louder, and eventually the ear switches from hearing the middle note as the primary to the bottom note as the primary, and then back to the middle later. Doing this as a continuously sliding tone requires some easy calculus. Consider just one component of the sound. We want to have a sine wave that is getting slower and slower over time. Say it starts at 440Hz and when we're done, it's going at 220Hz. How are we going to model this? Well, think "cycles per second". Take a wheel spinning on a fixed axle and put a mark on the edge. Ignore the side-to-side component of the mark as the wheel spins and just look at the up-and-down motion. That motion describes a sine wave. Spin the wheel at 440 cycles per second, and we'll get a 440 Hz sine wave out of the vertical component. How are we going to slow down from 440 Hz to 220 Hz, over, say, 16 seconds? We could use a linear model -- lose 14 Hz a second -- but that's not very sensible when we're thinking of sound. Remember, humans hear sounds based on ratios, not based on absolute numbers of cycles. Really what we want is for this to decay geometrically. We want the sound to have a "half life" in the frequency domain. It is convenient to measure the speed not in revolutions per second but in radians per second. There are 2π radians per revolution, so a wheel that is spinning at 440 Hz is spinning at 880π radians per second. Let's say that we're going to decay from our original frequency f=440 Hz down to f/2 in T seconds. What is the angular velocity at time t if it is an exponential decay? ω(t) = (2πf) 2<sup>-t/T</sup> radians per second Check that -- yep, that gives you 880π radians per second for t = 0, and 440π radians per second for t=T. Super. Now we need to determine the height of the mark at any given time. The angular position is easily determined from the angular velocity, and the height is the sine of the angle. Let's work out the angle: Θ(t) = ∫ω(t) dt = (-2fTπ/ln 2) 2<sup>-t/T</sup> + C for some constant C which we'll just set to zero arbitrarily -- we do not care about the phase, just the frequency. Take the sine of the angle, and we're all set, we've got our decaying wave. That takes care of the frequency decay. What about the change in volume of each component? We'll do a rough approximation of a bell curve. We want the very highest highs to come in very quietly. We want the very lowest lows to go out very quietly. And we want most of the sound energy in the middle, so that that's what you hear. Rather than muck around with computing a real bell curve, we'll just do a simple linear approximation of one. I find the illusion to be strongest when the primary note is pretty low. Let's try 110 Hz, two octaves below concert A. We'll make some simple modifications to yesterday's program. We'll decay over 16 seconds, and we'll do three decays. And we'll use eight full octaves.  
namespace Wave {  
  using System;  
  using System.IO;  
  class MainClass {  
    public static void Main(String\[\] args) {  
      FileStream stream = new FileStream("test.wav", FileMode.Create);   
      BinaryWriter writer = new BinaryWriter(stream);   
      int RIFF = 0x46464952;   
      int WAVE = 0x45564157;   
      int formatChunkSize = 16;   
      int headerSize = 8;   
      int format = 0x20746D66;   
      short formatType = 1;   
      short tracks = 1;   
      int samplesPerSecond = 44100;   
      short bitsPerSample = 16;   
      short frameSize = (short)(tracks \* ((bitsPerSample + 7)/8));   
      int bytesPerSecond = samplesPerSecond \* frameSize;   
      int waveSize = 4;   
      int data = 0x61746164;   
      int T = 16;   
      int reps = 3;   
      int samplesperrep = samplesPerSecond \* T;   
      int dataChunkSize = samplesperrep \* reps \* frameSize;   
      int fileSize = waveSize + headerSize + formatChunkSize + headerSize + dataChunkSize;   
      writer.Write(RIFF);    
      writer.Write(fileSize);   
      writer.Write(WAVE);   
      writer.Write(format);   
      writer.Write(formatChunkSize);   
      writer.Write(formatType);   
      writer.Write(tracks);   
      writer.Write(samplesPerSecond);   
      writer.Write(bytesPerSecond);   
      writer.Write(frameSize);   
      writer.Write(bitsPerSample);   
      writer.Write(data);   
      writer.Write(dataChunkSize);   
      double fundamental = 110.0;   
      double ampl = 10000;   
      for (int j = 0 ; j \< reps ; ++j) {   
        for (int i = 0; i \< samplesperrep; i++) {   
          double t = (double)i / (double)samplesPerSecond;   
          double amp0 = (ampl/8) \* t / T;   
          double amp1 = (ampl/8) + (ampl/8) \* t/T;   
          double amp2 = (ampl/4) + (ampl/4) \* t/T;   
          double amp3 = (ampl/2) + (ampl/2) \* t/T;   
          double amp4 = (ampl/1) - (ampl/2) \* t/T;   
          double amp5 = (ampl/2) - (ampl/4) \* t/T;   
          double amp6 = (ampl/4) - (ampl/8) \* t/T;   
          double amp7 = (ampl/8) - (ampl/8) \* t/T;   
          double theta = -fundamental \* 2 \* Math.PI \* T \* Math.Pow(2, -t/T) / Math.Log(2);   
          short s = (short)(   
            amp0*Math.Sin(theta*16)+amp1*Math.Sin(theta*8)+   
            amp2*Math.Sin(theta*4)+amp3*Math.Sin(theta*2)+   
            amp4*Math.Sin(theta*1)+amp5*Math.Sin(theta/2)+   
            amp6*Math.Sin(theta/4)+amp7\*Math.Sin(theta/8)   
          );   
          writer.Write(s);   
        }  
      }  
      writer.Close();  
      stream.Close();  
    }  
  }  
}  
The brain at some point stops thinking that the lower midrange frequency is the interesting one, and switches to the higher. If you listen attentively you can notice when your brain makes the switch, and the illusion is then somewhat dispelled. Pretty weird, eh? Coming up soon: back to wackiness in scripting\!

