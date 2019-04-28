# Compatibility vs. Performance

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/24/2003 1:17:00 PM

-----

Earlier I mentioned that two of the design goals for JScript .NET were **high performance** and **compatibility with JScript Classic**. Unfortunately these are somewhat contradictory goals\! JScript Classic has many dynamic features which make generation of efficient code difficult. Many of these features are rarely used in real-world programs. Others are programming idioms which make programs hard to follow, difficult to debug and slow.

 

 

JScript .NET therefore has two modes: **compatibility mode** and **fast mode**. In compatibility mode there should be almost no JScript program which is not a legal JScript .NET program. Fast mode restricts the use of certain seldom-used features and thereby produces faster programs.

 

 

The JSC.EXE command-line compiler and ASP.NET both use fast mode by default. To turn fast mode off in JSC use the /fast- switch. 

 

 

Fast mode puts the following restrictions on JScript .NET programs:

 

 

\* **All variables must be declared with the** var** keyword.** As I discussed earlier, in JScript Classic it is sometimes legal to use a variable without declaring it. In those situations, the JScript Classic engine automatically creates a new global variable but when in fast mode, JScript .NET does not. This is a good thing -- not only is the code faster but the compiler can now catch spelling errors in variable names.

\* **Functions may not be redefined**. In JScript Classic it is legal to have two or more identical function definitions which do different things. Only the **last** definition is actually used. This is not legal in JScript .NET in fast mode. This is also goodness, as it eliminates a source of confusion and bugs.

\* **Built-in objects are entirely read-only.** In JScript Classic it is legal to add, modify and (if you are perverse), delete some properties on the Math object, the String prototype and the other built-in objects. 

\* **Attempting to write to read-only properties now produces errors**. In JScript Classic writing to a read-only property fails silently, in keeping with the design principle I discussed earlier: muddle on through.

\* **Functions no longer have an** arguments** property.** The primary use of the arguments property is to create functions which take a variable number of arguments. JScript .NET has a specific syntax for creating such a function. This makes the arguments object unnecessary. To create a JScript .NET function which takes any number of arguments the syntax is:

function MyFunction(... args : Object\[\] )

{

  // now use args.length, args\[0\], etc.

}

 

 

Generally speaking, **unclear code is slow code**. If the compiler is unable to generate good code it is usually because the restrictions on the objects described in the code are so loose as to make optimization impossible. These few restrictions not only let JScript .NET generate faster code, they also enforce good programming style without overly damaging the "scripty" nature of the language. And if you must run code which has undeclared variables, redefined functions, modified built-in objects or reflection on the function arguments, then there is always compatibility mode to fall back upon.

 

 

JScript .NET also provides warnings when programming idioms could potentially produce slow code. For example, recall my earlier article on string concatenation.  Using the += operator on strings now produces a warning which suggests using a StringBuilder instead. JScript .NET also produces warnings when code is likely to be incorrect. For example, using a variable before initializing it produces a warning.  So does branching out of a finally block now produce warnings, and so on.

