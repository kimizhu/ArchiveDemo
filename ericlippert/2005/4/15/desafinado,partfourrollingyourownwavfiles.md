# Desafinado, Part Four: Rolling Your Own WAV Files

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 4/15/2005 2:29:00 PM

-----

We’ve established why every just about piano in the world -- in fact, every concert-pitched musical instrument in the world -- is slightly out of tune. No one actually plays perfect fifths; every fifth interval is slightly flat. Why don't we hear the difference? Is the difference even perceptible? It is *very* hard to hear unless you compare and contrast. So let's do that. Here's a little C\# program I just whipped up. This program creates a WAV file that first plays two seconds of E a perfect fifth above concert A=220Hz, and then two seconds of E a slightly flattened fifth above A. Can you hear the difference? I can't hear the difference between the two E's at all. However, you can REALLY hear the difference in the next section. The file then plays two seconds of E and a "perfect B" above it together, and then two seconds of E and a "concert B" above it. Now it is obvious -- with such clean, perfect waves you can really strongly hear it when it goes out of tune. You get a sort of ringing "wah wah wah" effect as the waves go in and out of sync with each other. The number of wahs, or, as piano tuners call them, **beats** per second tells you how close to a perfect fifth the notes are -- the slower the beats, the more in tune.  Experienced piano tuners can easily hear when the number of beats per second is just right for the piano to be exactly out of tune enough to be evenly tempered. This code could use some explanation. The basic WAV file format follows the Interchange File Format specification. An IFF file consists of a series of "chunks" where chunks can contain other chunks. Each chunk starts with an eight byte header: four bytes describing the chunk, followed by four bytes giving the size of the chunk (not counting the eight byte header). The header is followed by the given number of bytes of data in a chunk-specific format. A WAV file consists of one main chunk called RIFF that contains three things: the string "WAVE", a "format" chunk that describes the sample rate, etc, and a "data" chunk that contains the sampled waveform. We won't mess around with any advanced WAV file features like cue points or playlists or compression. We'll just dump out some data and play it with the WAV file player of your choice. We'll use CD quality audio -- 44100 samples per second, each one with 16 bits per sample. (Unlike a CD, we'll do this in mono, not stereo.)

namespace Wave  
{  
   using System;  
   using System.IO;  
   class MainClass {  
      public static void Main() {  
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
         int samples = 88200 \* 4;  
         int dataChunkSize = samples \* frameSize;  
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
         double aNatural = 220.0;  
         double ampl = 10000;  
         double perfect = 1.5;  
         double concert = 1.498307077;  
         double freq = aNatural \* perfect;  
         for (int i = 0; i \< samples / 4; i++) {  
            double t = (double)i / (double)samplesPerSecond;  
            short s = (short)(ampl \* (Math.Sin(t \* freq \* 2.0 \* Math.PI)));  
            writer.Write(s);  
         }  
         freq = aNatural \* concert;  
         for (int i = 0; i \< samples / 4; i++) {  
            double t = (double)i / (double)samplesPerSecond;  
            short s = (short)(ampl \* (Math.Sin(t \* freq \* 2.0 \* Math.PI)));  
            writer.Write(s);  
         }  
         for (int i = 0; i \< samples / 4; i++) {  
            double t = (double)i / (double)samplesPerSecond;  
            short s = (short)(ampl \* (Math.Sin(t \* freq \* 2.0 \* Math.PI) + Math.Sin(t \* freq \* perfect \* 2.0 \* Math.PI)));  
            writer.Write(s);  
         }  
         for (int i = 0; i \< samples / 4; i++) {  
            double t = (double)i / (double)samplesPerSecond;  
            short s = (short)(ampl \* (Math.Sin(t \* freq \* 2.0 \* Math.PI) + Math.Sin(t \* freq \* concert \* 2.0 \* Math.PI)));  
            writer.Write(s);  
         }  
         writer.Close();  
         stream.Close();  
      }  
   }  
}

Compile this guy up, run it, and listen to test.wav. Pretty cool eh? Next time, we'll wrap up with one of the most interesting psychological effects you can get in music -- a tone that goes down, but never hits bottom.

