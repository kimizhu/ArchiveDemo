# More on full qualification, this time in C\#

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 7/15/2005 1:01:00 PM

-----

We were speaking earlier about how to resolve ambiguous enumerated type names in VBScript. Here’s a related C\# question I got recently: namespace Alpha.Bravo.Charlie {  
  public class Romeo {}  
}  
namespace Zulu.Alpha.Bravo {  
  public class Tango {}  
}  
namespace Zulu.Alpha.Bravo.Charlie {  
  public class Sierra {  
    Alpha.Bravo.Charlie.Romeo xray;  
    // Above line fails, even though it is fully qualified.  
    Alpha.Bravo.Tango yankee;        
    // Above line works even though Alpha.Bravo has no such type.  
  }  
} What the heck is up with that? VBScript was designed to get this right when fully qualified, so why doesn’t C\# get it right? The reason is because the namespace statement doesn’t do what you think it does.  The above program is syntactic sugar for this program: namespace Alpha {  
  namespace Bravo {  
    namespace Charlie {  
      public class Romeo {}  
    }  
  }  
}  
namespace Zulu {  
  namespace Alpha {  
    namespace Bravo {  
      public class Tango {}  
      namespace Charlie {  
        public class Sierra {  
          Alpha.Bravo.Charlie.Romeo xray;   
          Alpha.Bravo.Tango yankee;  
        }  
      }  
    }  
  }  
} And now it is clear why the first fails and the second succeeds.  C\# binds Alpha by searching up the scope chain until it finds a match, and it finds one before it gets to the global Alpha. This is a rather unfortunate situation. To guarantee that you [reallio trulio](http://www.eecs.harvard.edu/~keith/poems/Custard.html)fully qualify a type in this situation, you can do this in C\# v2.0: global::Alpha.Bravo.Charlie.Romeo xray;

