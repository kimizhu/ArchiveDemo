# Psychic Debugging, Part One

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 9/5/2007 1:30:00 PM

-----

Here is a compiler bug report I got the other day. The user is trying to write a unit test for a method which takes a Foo and returns a collection of Bars. The test is supposed to confirm that GetBars throws an exception if the argument is null. The test was failing with “got no exception”. The user was wondering if somehow the compiler had optimized away the call.

 

    static void Test() {  
        bool gotException = false;  
        try {  
            IEnumerable\<Bar\> bars = Foo.GetBars(null);  
        }  
        catch (ArgumentNullException ex) {  
            Console.WriteLine("SUCCESS: Got expected exception");  
            gotException = true;  
        }  
        catch (Exception ex) {  
            Console.WriteLine("FAILURE: Got unexpected exception");  
            gotException = true;  
        }  
        finally {  
            if (\!gotException)  
                Console.WriteLine("FAILURE: Got no exception");  
        }  
    }

Several people (including myself and [Cyrus](http://blogs.msdn.com/cyrusn/)) psychically debugged this one independently. The user did not include the source code of GetBars, which is what necessitated the use of our psychic powers. Because I am a nice guy, I will give you the beginning:

 

    static public IEnumerable\<Bar\> GetBars(Foo foo) {  
        if (foo == null)  
            throw new ArgumentNullException("foo");

**Our psychic powers correctly told us that the behaviour of the test program is correct and expected.** What do your psychic powers tell you about the rest of the implementation of GetBars? What is going on here?

